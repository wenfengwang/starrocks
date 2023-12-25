---
displayed_sidebar: Chinese
---

# AUTO INCREMENT 列を使用してグローバル辞書を構築し、正確な重複計算と結合を高速化する

## アプリケーションシナリオ

- **シナリオ1**：大量の注文データ（小売注文、宅配注文など）の正確な重複計算が必要です。ただし、重複カウントの列はSTRING型であり、直接計算するとパフォーマンスが十分ではありません。たとえば、注文テーブル`orders`の`order_uuid`列はSTRING型で、通常は32〜36バイトで、`UUID()`などの関数によって生成されます。STRING列`order_uuid`を使用して正確な重複カウントを行う`SELECT count(DISTINCT order_uuid) FROM orders WHERE create_date >= CURDATE();`というクエリのパフォーマンスは、要件を満たすことができないかもしれません。
  INTEGER列を使用して正確な重複カウントを行う場合、パフォーマンスが大幅に向上します。
- **シナリオ2**：[ビットマップ関数を使用して注文の正確な重複計算をさらに高速化する](https://docs.starrocks.io/zh-cn/latest/using_starrocks/Using_bitmap)必要があります。ただし、`bitmap_count()`関数はINTEGER型の入力値を必要としますが、ビジネスシナリオで重複カウントの列がSTRING型の場合、`bitmap_hash()`関数を使用する必要があります。ただし、これにより、返される重複カウントは近似的で少ない値になる可能性があります。また、連続的に割り当てられたINTEGER値に比べて、`bitmap_hash()`によって生成されるINTEGER値はより分散しており、クエリのパフォーマンスの低下やデータのストレージ量の増加につながる可能性があります。
- **シナリオ3**：注文から支払いまでの時間が比較的短い注文の数をクエリする必要がありますが、注文時間と支払い時間は2つのテーブルに格納されており、異なるビジネスチームによって管理されています。したがって、注文番号を基に2つのテーブルを結合し、注文の正確な重複カウントを行う必要があります。以下は例です：

    ```SQL
    SELECT count(distinct order_uuid)
    FROM orders_t1 as t1 JOIN orders_t2 as t2
        ON t1.order_uuid = t2.order_uuid
    WHERE t2.payment_time - t1.create_time <= 3600
        AND create_date >= CURDATE();
    ```

   ただし、注文番号の`order_uuid`列はSTRING型であり、STRING列を基に結合すると、パフォーマンスもINTEGER列を基に結合する場合ほど良くありません。

## 最適化のアイデア

上記のアプリケーションシナリオに対して、最適化のアイデアは、注文データをターゲットテーブルにインポートし、STRING値とINTEGER値の間のマッピング関係を構築し、後続のクエリ分析をINTEGER列を基に行うことです。このアイデアは、次のフェーズで実行できます：

1. フェーズ1：グローバル辞書を作成し、STRING値とINTEGER値のマッピング関係を構築します。辞書のキー列はSTRING型であり、値列はINTEGER型であり、自動インクリメント列です。データをインポートするたびに、システムは自動的に各STRING値に対してテーブル内で一意のIDを生成し、STRING値とINTEGER値のマッピング関係が確立されます。
2. フェーズ2：注文データとグローバル辞書のマッピング関係をターゲットテーブルにインポートします。
3. フェーズ3：後続のクエリ分析では、ターゲットテーブルのINTEGER列を基に正確な重複計算や結合を行うことで、パフォーマンスを大幅に向上させることができます。
4. フェーズ4：さらなるパフォーマンスの最適化のために、INTEGER列にビットマップ関数を使用することもできます。

## 解決策

上記のフェーズ2は2つの方法で実現できるため、このドキュメントでは2つの解決策を提供します。主な違いは次のとおりです：

- **解決策1**
  
  1つの操作でデータをインポートします：注文データテーブルを辞書テーブルと結合し、注文データと辞書テーブルのマッピング関係をINSERT INTOステートメントで同時にターゲットテーブルにインポートします。<br />
  ここで、結合句の注文データテーブルは外部テーブルまたは中間テーブル（元のデータを最初にこの中間テーブルにインポートすることができます）のいずれかです。
- **解決策2**

  データを2つの操作でインポートします：まず、注文データをターゲットテーブルにインポートします。次に、UPDATEステートメント（技術的にはターゲットテーブルと辞書テーブルの結合）を使用して、辞書テーブルのマッピング関係をターゲットテーブルのINTEGER列に更新します。<br />
  ここでの更新操作では、ターゲットテーブルは主キーモデルテーブルである必要があります。

### 解決策1：External Catalog 外部テーブル + INSERT INTOデータを使用

#### ユースケース

注文データは、External Catalog 外部テーブル、ファイル外部テーブル、またはFILESテーブル関数（外部テーブルとして理解できる）のいずれかから取得する必要があります。

この解決策では、Hive Catalog 外部テーブル `source_table` を例に説明します。ここで、`order_uuid` 列は注文番号を表し、STRING型です。

> **注意**
>
> 外部テーブルを持っていない場合は、内部テーブルを使用することもできます。元のデータを中間テーブルにインポートして、そのテーブルを外部テーブルの役割を果たすようにします。

| id   | order_uuid | バッチ |
| ---- | ---------- | ----- |
| 1    | A1 (日本語)         | 1     |
| 2    | A2の         | 1     |
| 3    | A3の         | 1     |
| 11   | A1 (日本語)         | 1     |
| 11   | A2の         | 1     |
| 12   | A1 (日本語)         | 1     |
| 1    | A2の         | 2     |
| 2    | A2の         | 2     |
| 3    | A2の         | 2     |
| 11   | A2の         | 2     |
| 12   | A102 (日本語)       | 2     |
| 12   | A101の       | 2     |
| 13   | A102 (日本語)       | 2     |

> **注意**
>
> - ここでの `id` は、注文ビジネスの簡略化されたキー、他のフィールドなどです。
> - 注文データは通常、バッチごとにHDFSやAWS S3などのオブジェクトストレージにインポートされます。このドキュメントでは、`batch` フィールドはデータのインポートバッチを区別するために使用されます。

#### 具体的な手順

**フローチャート**

注文データテーブルを辞書テーブルと結合し、注文データと辞書テーブルのマッピング関係をINSERT INTOステートメントで同時にターゲットテーブルにインポートします。

![ExternalCatalog](../assets/ExternalCatalog.png)

**準備**: [Hive catalog](../data_source/catalog/hive_catalog.md) を作成し、StarRocksがHiveテーブル `source_table` にアクセスできるようにします。

**フェーズ1: グローバル辞書を作成し、Hiveテーブルの注文番号列の値をインポートして、STRING値とINTEGER値のマッピング関係を構築します。**

1. グローバル辞書として主キーモデルテーブルを作成します。主キーである `order_uuid` 列はSTRING型であり、値列である `order_id_int` 列はBIGINT型であり、自動インクリメントです。

    ```SQL
    CREATE TABLE dict (
        order_uuid STRING,
        order_id_int BIGINT AUTO_INCREMENT -- システムは各order_uuid値に対してテーブル内で一意のIDを自動的に割り当てます
    )
    PRIMARY KEY (order_uuid)
    DISTRIBUTED BY HASH (order_uuid)
    PROPERTIES("replicated_storage" = "true");
    ```

2. Hiveテーブル `source_table` の最初のバッチの `order_uuid` 値（STRING型）を辞書テーブル `dict` の主キー `order_uuid`（STRING型）に挿入します。このようにすることで、辞書テーブルの自動インクリメント列 `order_id_int`（BIGINT型）が各 `order_uuid` に対してテーブル内で一意のIDを自動的に生成し、STRING値とBIGINT値のマッピング関係が確立されます。
   注意点として、挿入する際には、Left Join句と `WHERE dict.order_uuid IS NULL` の条件を使用して、挿入する `order_uuid` 値が辞書テーブルに既に存在しないことを確認します。重複した `order_uuid` 値を辞書テーブルに挿入すると、その `order_uuid` 値に対するBIGINT値のマッピングが変わる可能性があります。なぜなら、辞書テーブルは主キーモデルのテーブルであり、INSERT INTOで同じ主キーを挿入すると、辞書テーブルに既に存在する行が上書きされ、対応する自動インクリメント列の値も変わるからです（注意：現在、INSERT INTOは部分的な列の更新をサポートしていません）。

      ```SQL
      INSERT INTO dict (order_uuid)
      SELECT DISTINCT src.order_uuid  -- 重複を排除したorder_uuidを取得する必要があります
      FROM hive_catalog.hive_db.source_table src LEFT JOIN dict
          ON src.order_uuid = dict.order_uuid
      WHERE dict.order_uuid IS NULL
          AND src.batch = 1;
      ```

**フェーズ2: ターゲットテーブルを作成し、Hiveテーブルの注文データと辞書テーブルのINTEGER列の値をインポートします。これにより、STRING値とINTEGER値のマッピング関係がターゲットテーブルに構築され、後続のクエリ分析に基づいてINTEGER列を使用できます。**

1. 明細モデルテーブル `dest_table` を作成します。`source_table` のすべての列に加えて、STRING型の `order_uuid` 列とマッピングするINTEGER型の `order_id_int` 列を定義します。後続のクエリ分析では、`order_id_int` 列を基に処理を行います。

      ```SQL
      CREATE TABLE dest_table (
          id BIGINT,
          order_uuid STRING, -- STRING型の注文番号を記録する列
          order_id_int BIGINT NULL, -- STRING型とBIGINT型のマッピングを相互に行う列。正確な重複計算と結合に使用します
          batch int comment 'used to distinguish different batch loading'
      )
      DUPLICATE KEY (id, order_uuid, order_id_int)
      DISTRIBUTED BY HASH(id);
      ```

2. Hiveテーブル `source_table` の最初のバッチの注文データと辞書テーブル `dict` のINTEGER列の値をターゲットテーブル `dest_table` にインポートします。

      ```SQL
      INSERT INTO dest_table (id, order_uuid, order_id_int, batch)
      SELECT src.id, src.order_uuid, dict.order_id_int, src.batch
      FROM hive_catalog.hive_db.source_table AS src LEFT JOIN dict
          ON src.order_uuid = dict.order_uuid
      WHERE src.batch = 1;
      ```

      > **注意**
      >
      > 理論的には、ここではLeft Joinではなく、通常のJoinでも問題ありません。ただし、いくつかの例外の影響を防ぐために、Left Joinを使用して、`source_table` のデータがすべて `dest_table` にインポートされることを保証することができます。たとえば、`dict` に `order_uuid` をインポートする際にエラーが発生し、それに気づかなかった場合などです。後続の処理では、`dest_table` に `order_id_int` 列がNULLの行が存在するかどうかを確認することで、これを判断することができます。

3. フェーズ1のステップ2とフェーズ2のステップ2を繰り返し、`source_table` の2番目のバッチのデータを `dest_table` にインポートし、`order_uuid`（STRING型）と `order_id_int`（BIGINT型）のマッピング関係を構築します。

   1. `source_table` の2番目のバッチの `order_uuid` 値を `dict` テーブルの主キー `order_uuid` に挿入します。

      ```SQL
      INSERT INTO dict (order_uuid)
      SELECT DISTINCT src.order_uuid
      FROM hive_catalog.hive_db.source_table src LEFT JOIN dict
          ON src.order_uuid = dict.order_uuid
      WHERE dict.order_uuid IS NULL
          AND src.batch = 2;
      ```

   2. `source_table` の2番目のバッチの注文データと `dict` テーブルのINTEGER列の値を `dest_table` にインポートします。

      ```SQL
      INSERT INTO dest_table (id, order_uuid, order_id_int, batch)
      SELECT src.id, src.order_uuid, dict.order_id_int, src.batch
      FROM hive_catalog.hive_db.source_table AS src LEFT JOIN dict
          ON src.order_uuid = dict.order_uuid
      WHERE src.batch = 2;
      ```


4. `dest_table` の `order_uuid` 列と `order_id_int` 列のマッピング関係を確認します。

    ```sql
    MySQL > SELECT * FROM dest_table ORDER BY batch, id;
    +------+------------+--------------+-------+
    | id   | order_uuid | order_id_int | batch |
    +------+------------+--------------+-------+
    |    1 | a1         |            1 |     1 |
    |    2 | a2         |            2 |     1 |
    |    3 | a3         |            3 |     1 |
    |   11 | a1         |            1 |     1 |
    |   11 | a2         |            2 |     1 |
    |   12 | a1         |            1 |     1 |
    |    1 | a2         |            2 |     2 |
    |    2 | a2         |            2 |     2 |
    |    3 | a2         |            2 |     2 |
    |   11 | a2         |            2 |     2 |
    |   12 | a102       |       100002 |     2 |
    |   12 | a101       |       100001 |     2 |
    |   13 | a102       |       100002 |     2 |
    +------+------------+--------------+-------+
    13 rows in set (0.02 sec)
    ```

**フェーズ3：実際のクエリ分析では、INTEGER 型の列で精確な重複排除や Join を行うことができ、STRING 型の列を使用するよりもパフォーマンスが大幅に向上します。**

```SQL
-- BIGINT 型の order_id_int を使用した精確な重複排除
SELECT id, COUNT(DISTINCT order_id_int) FROM dest_table GROUP BY id ORDER BY id;
-- STRING 型の order_uuid を使用した精確な重複排除
SELECT id, COUNT(DISTINCT order_uuid) FROM dest_table GROUP BY id ORDER BY id;
```

[bitmap 関数を使用して精確な重複排除の計算を加速する](#bitmap-関数を使用して精確な重複排除の計算を加速する)こともできます。

### 解決策2：インポート + UPDATE データ（プライマリキーモデルテーブル）

#### ビジネスシナリオ

この解決策は、2つの CSV ファイル `batch1.csv` と `batch2.csv` を例に説明します。ファイルには `id` と `order_uuid` の2列が含まれています。

- batch1.csv

    ```csv
    1, a1
    2, a2
    3, a3
    11, a1
    11, a2
    12, a1
    ```

- batch2.csv

    ```csv
    1, a2
    2, a2
    3, a2
    11, a2
    12, a101
    12, a102
    13, a102
    ```

#### 具体的な手順

**フローチャート**

まず注文データをターゲットテーブルにインポートし、その後で辞書テーブルのマッピング関係をターゲットテーブルの INTEGER 列に更新します（更新操作はターゲットテーブルがプライマリキーモデルである必要があります）。

![loading](../assets/loading.png)

**フェーズ1：グローバル辞書テーブルを作成し、CSV ファイルの注文番号列の値をインポートして、STRING と INTEGER の値のマッピング関係を構築します。**

1. グローバル辞書としてプライマリキーモデルテーブルを作成し、主キーである `order_uuid` 列（STRING 型）と値の `order_id_int` 列（BIGINT 型、自動インクリメント）を定義します。

      ```SQL
      CREATE TABLE dict (
          order_uuid STRING,
          order_id_int BIGINT AUTO_INCREMENT  -- 各 order_uuid に一意の ID を自動割り当て
      )
      PRIMARY KEY (order_uuid)
      DISTRIBUTED BY HASH (order_uuid)
      PROPERTIES("replicated_storage" = "true");
      ```

2. この例では Stream Load を使用して、2つの CSV ファイルの `order_uuid` 列を辞書テーブル `dict` の `order_uuid` 列にバッチでインポートします。ここで注意が必要なのは、部分的な列の更新を使用することです。

      ```Bash
      curl --location-trusted -u root: \
          -H "partial_update: true" \
          -H "format: CSV" -H "column_separator:," -H "columns: id, order_uuid" \
          -T batch1.csv \
          -XPUT http://<fe_host>:<fe_http_port>/api/example_db/dict/_stream_load
          
      curl --location-trusted -u root: \
          -H "partial_update: true" \
          -H "format: CSV" -H "column_separator:," -H "columns: id, order_uuid" \
          -T batch2.csv \
          -XPUT http://<fe_host>:<fe_http_port>/api/example_db/dict/_stream_load
      ```

**フェーズ2：ターゲットテーブルを作成し、CSV ファイルの注文データをインポートします。**

この解決策では、ターゲットテーブルはプライマリキーモデルである必要があります。なぜなら、JOIN を伴う UPDATE コマンドはプライマリキーモデルテーブルでのみサポートされるからです。

1. CSV ファイルのすべての列を含む**プライマリキーモデルテーブル** `dest_table_pk` を作成します。また、STRING 型の `order_uuid` 列とマッピングする INTEGER 型の `order_id_int` 列を定義する必要があります。後で `order_id_int` 列を使用してクエリ分析を行います。

   また、プライマリキーモデルテーブルにすべてのインポートされたデータを保持するために、各データの一意の識別子として機能する自動インクリメントの `implicit_auto_id` 列を主キーとして定義する必要があります（実際の注文ビジネステーブルに他のプライマリキーが既に存在する場合は、それを使用し、`implicit_auto_id` を作成する必要はありません）。

      ```SQL
      CREATE TABLE dest_table_pk (
          implicit_auto_id BIGINT AUTO_INCREMENT comment 'used to keep all loaded rows',
          id BIGINT,
          order_uuid STRING, -- STRING 型の注文番号を記録する列
          order_id_int BIGINT NULL, -- BIGINT 型の注文番号を記録する列で、前の列と相互にマッピングされ、後で精確な重複排除と Join に使用されます
          batch int comment 'used to distinguish different batch loading'
      )
      PRIMARY KEY (implicit_auto_id)  -- プライマリキーモデルテーブルである必要があります
      DISTRIBUTED BY HASH (implicit_auto_id)
      PROPERTIES("replicated_storage" = "true");
      ```

2. 2つの CSV ファイルに含まれる注文データを `dest_table_pk` テーブルにバッチでインポートします。

      ```Bash
      curl --location-trusted -u root: \
          -H "format: CSV" -H "column_separator:," -H "columns: id, order_uuid, batch=1" \
          -T batch1.csv \
          -XPUT http://<fe_host>:<fe_http_port>/api/example_db/dest_table_pk/_stream_load
          
      curl --location-trusted -u root: \
          -H "format: CSV" -H "column_separator:," -H "columns: id, order_uuid, batch=2" \
          -T batch2.csv \
          -XPUT http://<fe_host>:<fe_http_port>/api/example_db/dest_table_pk/_stream_load
      ```

**フェーズ2：辞書テーブルの INTEGER 列の値を、ターゲットテーブルの `order_id_int` 列に更新します。これにより、ターゲットテーブル内で STRING と INTEGER の値のマッピング関係を構築し、後で `order_id_int` 列を使用してクエリ分析を行います。**

1. グローバル辞書のマッピング関係に基づいて、辞書テーブルの INTEGER 値を `dest_table_pk` テーブルの `order_id_int` 列に部分的に更新します。

      ```SQL
      UPDATE dest_table_pk
      SET order_id_int = dict.order_id_int
      FROM dict
      WHERE dest_table_pk.order_uuid = dict.order_uuid AND batch = 1;
      
      UPDATE dest_table_pk
      SET order_id_int = dict.order_id_int
      FROM dict
      WHERE dest_table_pk.order_uuid = dict.order_uuid AND batch = 2;
      ```

2. `dest_table_pk` の `order_uuid` 列と `order_id_int` 列のマッピング関係を確認します。

    ```sql
    MySQL > SELECT * FROM dest_table_pk ORDER BY batch, id;
    +------------------+------+------------+--------------+-------+
    | implicit_auto_id | id   | order_uuid | order_id_int | batch |
    +------------------+------+------------+--------------+-------+
    |                0 |    1 | a1         |            1 |     1 |
    |                1 |    2 | a2         |            2 |     1 |
    |                2 |    3 | a3         |            3 |     1 |
    |                4 |   11 | a2         |            2 |     1 |
    |                3 |   11 | a1         |            1 |     1 |
    |                5 |   12 | a1         |            1 |     1 |
    |                6 |    1 | a2         |            2 |     2 |
    |                7 |    2 | a2         |            2 |     2 |
    |                8 |    3 | a2         |            2 |     2 |
    |                9 |   11 | a2         |            2 |     2 |
    |               10 |   12 | a102       |            4 |     2 |
    |               11 |   12 | a101       |            5 |     2 |
    |               12 |   13 | a102       |            4 |     2 |
    +------------------+------+------------+--------------+-------+
    13 rows in set (0.01 sec)
    ```

**フェーズ3：**実際のクエリ分析では、INTEGER 型の `order_id_int` 列で精確な重複排除や Join を行うことができ、STRING 型の `order_uuid` 列を使用するよりもパフォーマンスが大幅に向上します。

```SQL
-- BIGINT 型の order_id_int を使用した精確な重複排除
SELECT id, COUNT(DISTINCT order_id_int) FROM dest_table_pk GROUP BY id ORDER BY id;
-- STRING 型の order_uuid を使用した精確な重複排除
SELECT id, COUNT(DISTINCT order_uuid) FROM dest_table_pk GROUP BY id ORDER BY id;
```

[bitmap 関数を使用して精確な重複排除の計算を加速する](#bitmap-関数を使用して精確な重複排除の計算を加速する)こともできます。

### bitmap 関数を使用して精確な重複排除の計算を加速する

計算をさらに加速するために、グローバル辞書を構築した後、辞書テーブルの INTEGER 列の値を bitmap 列に直接挿入することができます。その後、bitmap 列に対して bitmap 関数を使用して精確な重複排除を行います。

1. 最初に**アグリゲートモデルテーブル**を作成し、2つの列を含みます。アグリゲート列は BITMAP 型の `order_id_bitmap` で、集約関数 `bitmap_union()` を指定します。もう一つの列は BIGINT 型の `id` です。このテーブルは元の STRING 列を**含まない**ことに注意してください（そうでなければ、各 bitmap には1つの値しか含まれず、加速効果は得られません）。

      ```SQL
      CREATE TABLE dest_table_bitmap (
          id BIGINT,
          order_id_bitmap BITMAP BITMAP_UNION
      )
      AGGREGATE KEY (id)
      DISTRIBUTED BY HASH(id) BUCKETS 2;
      ```

2. 使用する解決策に応じて、データの挿入方法を選択します。
   - **解決策1**を使用する場合は、`hive_catalog.hive_db.source_table` テーブルの `id` 列からアグリゲートモデルテーブル `dest_table_bitmap` の `id` 列にデータを挿入し、辞書テーブル `dict` の INTEGER 列 `order_id_int` のデータを `to_bitmap` 関数を使用して処理した後、アグリゲートモデルテーブルの `order_id_bitmap` 列に挿入する必要があります。

        ```SQL
        INSERT INTO dest_table_bitmap (id, order_id_bitmap)
        SELECT src.id, to_bitmap(dict.order_id_int)
        FROM hive_catalog.hive_db.source_table AS src LEFT JOIN dict
            ON src.order_uuid = dict.order_uuid
        WHERE src.batch = 1; -- バッチごとにデータを模擬的にインポートするため、2番目のバッチのデータをインポートする際には 2 を指定します
        ```

   - **解決策2**を使用する場合は、`dest_table_pk` テーブルの `id` 列からアグリゲートモデルテーブル `dest_table_bitmap` の `id` 列にデータを挿入し、辞書テーブル `dict` の INTEGER 列 `order_id_int` のデータを `to_bitmap` 関数を使用して処理した後、アグリゲートモデルテーブルの `order_id_bitmap` 列に挿入する必要があります。

        ```SQL
        INSERT INTO dest_table_bitmap (id, order_id_bitmap)
        SELECT dest_table_pk.id, to_bitmap(dict.order_id_int)
        FROM dest_table_pk LEFT JOIN dict
            ON dest_table_pk.order_uuid = dict.order_uuid
        WHERE dest_table_pk.batch = 1; -- これはデータをバッチでインポートするためのもので、2番目のバッチをインポートする際には 2 を指定します
        ```

3. 次に、bitmap 列を使用して `BITMAP_UNION_COUNT()` 関数で正確な重複排除カウントを行います。

      ```SQL
      SELECT id, BITMAP_UNION_COUNT(order_id_bitmap) FROM dest_table_bitmap
      GROUP BY id ORDER BY id;
      ```