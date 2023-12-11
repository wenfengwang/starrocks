---
displayed_sidebar: "Japanese"
---

# 同期素材化ビュー

このトピックでは、**同期素材化ビュー(Rollup)** の作成、使用、および管理方法について説明します。

同期素材化ビューでは、基本テーブルのすべての変更が対応する同期素材化ビューに同時に更新されます。同期素材化ビューのリフレッシュは自動的にトリガーされます。同期素材化ビューはメンテナンスおよび更新が格段に安価であり、リアルタイムの単一テーブル集約クエリの透過的な高速化に適しています。

StarRocksの同期素材化ビューは、[デフォルトカタログ](../data_source/catalog/default_catalog.md) の単一の基本テーブルのみで作成できます。これらは、非同期素材化ビューのような物理テーブルではなく、クエリの高速化のための特別なインデックスです。

v2.4以降、StarRocksは複数のテーブルで作成をサポートし、より多くの集約演算子をサポートする非同期素材化ビューを提供しています。**非同期素材化ビュー** の使用方法については、[非同期素材化ビュー](../using_starrocks/Materialized_view.md) を参照してください。

次の表は、StarRocks v2.5、v2.4の非同期素材化ビュー（ASYNC MVs）と同期素材化ビュー（SYNC MV）の機能の観点からの比較です。

|                       | **単一テーブルの集約** | **複数テーブルの結合** | **クエリの書き換え** | **リフレッシュ戦略** | **基本テーブル** |
| --------------------- | ----------------------- | ----------------------- | -------------------- | -------------------- | ---------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | デフォルトカタログからの複数のテーブル:<ul><li>デフォルトカタログ</li><li>外部カタログ（v2.5）</li><li>既存の素材化ビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **SYNC MV (Rollup)**  | [集約関数の制限された選択肢](#correspondence-of-aggregate-functions) | いいえ | はい | データのロード中に同期リフレッシュ | デフォルトカタログの単一のテーブル |

## 基本的な概念

- **基本テーブル**

  基本テーブルは素材化ビューの駆動テーブルです。

  StarRocksの同期素材化ビューでは、基本テーブルはデフォルトカタログの単一のネイティブテーブルである必要があります。StarRocksは、重複キーのテーブル、集約テーブル、ユニークキーのテーブルで同期素材化ビューを作成することをサポートしています。

- **リフレッシュ**

  同期素材化ビューは、基本テーブルのデータが変更されるたびに自動的に更新されます。リフレッシュを手動でトリガーする必要はありません。

- **クエリの書き換え**

  クエリの書き換えとは、素材化ビューで構築された基本テーブルのクエリを実行する際に、システムが自動的に事前計算された結果をクエリに再利用できるかどうかを判断することを意味します。再利用できる場合、システムは関連する素材化ビューからデータを直接読み込んで、時間とリソースを消費する計算や結合を回避します。

  同期素材化ビューは、一部の集約演算子に基づいてクエリの書き換えをサポートしています。詳細については、[集約関数の対応関係](#correspondence-of-aggregate-functions) を参照してください。

## 事前準備

同期素材化ビューを作成する前に、データウェアハウスが同期素材化ビューを使用してクエリを高速化できるかどうかを確認してください。たとえば、クエリが特定のサブクエリステートメントを再利用するかどうかを確認します。

次の例は、取引ID `record_id`、販売員ID `seller_id`、店舗ID `store_id`、日付 `sale_date`、販売金額 `sale_amt` を含む`sales_records`テーブルを基にしています。この表を作成し、データを挿入するために次の手順に従います:

```SQL
CREATE TABLE sales_records(
    record_id INT,
    seller_id INT,
    store_id INT,
    sale_date DATE,
    sale_amt BIGINT
) DISTRIBUTED BY HASH(record_id);

INSERT INTO sales_records
VALUES
    (001,01,1,"2022-03-13",8573),
    (002,02,2,"2022-03-14",6948),
    (003,01,1,"2022-03-14",4319),
    (004,03,3,"2022-03-15",8734),
    (005,03,3,"2022-03-16",4212),
    (006,02,2,"2022-03-17",9515);
```

この例のビジネスシナリオでは、異なる店舗の販売金額を頻繁に分析する必要があります。その結果、各クエリで`sum()`関数が使用され、大量の計算リソースを消費します。クエリを実行してその時間を記録し、EXPLAINコマンドを使用してそのクエリプロファイルを表示できます。

```Plain
MySQL > SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
+----------+-----------------+
| store_id | sum(`sale_amt`) |
+----------+-----------------+
|        2 |           16463 |
|        3 |           12946 |
|        1 |           12892 |
+----------+-----------------+
3 rows in set (0.02 sec)

MySQL > EXPLAIN SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
+-----------------------------------------------------------------------------+
| Explain String                                                              |
+-----------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                             |
|  OUTPUT EXPRS:3: store_id | 6: sum                                          |
|   PARTITION: UNPARTITIONED                                                  |
|                                                                             |
|   RESULT SINK                                                               |
|                                                                             |
|   4:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 1                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: HASH_PARTITIONED: 3: store_id                                  |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 04                                                         |
|     UNPARTITIONED                                                           |
|                                                                             |
|   3:AGGREGATE (merge finalize)                                              |
|   |  output: sum(6: sum)                                                    |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   2:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 2                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: RANDOM                                                         |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 02                                                         |
|     HASH_PARTITIONED: 3: store_id                                           |
|                                                                             |
|   1:AGGREGATE (update serialize)                                            |
|   |  STREAMING                                                              |
|   |  output: sum(5: sale_amt)                                               |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   0:OlapScanNode                                                            |
|      TABLE: sales_records                                                   |
|      PREAGGREGATION: ON                                                     |
|      partitions=1/1                                                         |
|      rollup: sales_records                                                  |
|      tabletRatio=10/10                                                      |
|      tabletList=12049,12053,12057,12061,12065,12069,12073,12077,12081,12085 |
|      cardinality=1                                                          |
|      avgRowSize=2.0                                                         |
|      numNodes=0                                                             |
+-----------------------------------------------------------------------------+
45 rows in set (0.00 sec)
```

クエリには約0.02秒かかり、クエリプロファイルの`rollup`フィールドの値が`sales_records`であるため、同期素材化ビューがクエリの高速化に使用されていないことがわかります。

## 同期素材化ビューの作成

特定のクエリステートメントに基づいて同期素材化ビューを作成するには、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md) を使用します。

`sales_records`テーブルと上記のクエリステートメントを基にして、次の例では同期素材化ビュー `store_amt` を作成しています。これにより、各店舗の販売金額の合計を分析できます。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **注意**
>
> - 同期素材化ビューで集約関数を使用する場合、SELECTリストでGROUP BY句を使用し、少なくとも1つのGROUP BY列を指定する必要があります。
> - 同期素材化ビューでは、1つの列に対して複数の集約関数を使用するクエリステートメント（`sum(a+b)`など）はサポートされていません。
> - 同期素材化ビューでは、1つの列に複数の集約関数を使用するクエリステートメント（`select sum(a), min(a) from table`など）はサポートされていません。
> - 同期素材化ビューの作成時にJOINはサポートされていません。
> - 基本テーブルの特定の列を`ALTER TABLE DROP COLUMN`を使用して削除する場合、基本テーブルのすべての同期素材化ビューに削除された列が含まれていないことを確認する必要があります。そうでない場合、削除操作を実行できません。同期素材化ビューで使用されている列を削除するには、まずその列を含むすべての同期素材化ビューを削除し、その後に列を削除する必要があります。
> - 1つのテーブルに対して多くの同期素材化ビューを作成すると、データのロード効率に影響を与えます。基本テーブルにデータをロードしている間、同期素材化ビューと基本テーブルのデータが同期して更新されます。基本テーブルに`n`個の同期素材化ビューが含まれている場合、基本テーブルへのデータのロード効率は`n`個のテーブルへのデータのロード効率とほぼ同じになります。
- 現在、StarRocksは複数の同期マテリアライズドビューの同時作成をサポートしていません。新しい同期マテリアライズドビューは、前のビューが完了した場合にのみ作成できます。
- 共有データのStarRocksクラスターは同期マテリアライズドビューをサポートしていません。

## 同期マテリアライズドビューのビルド状況を確認する

同期マテリアライズドビューの作成は非同期操作です。CREATE MATERIALIZED VIEWの実行が成功すると、マテリアライズドビューのタスクが正常に送信されたことを示します。データベースにおける同期マテリアライズドビューのビルド状況は、[SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)を使用して確認できます。

```Plain
MySQL > SHOW ALTER MATERIALIZED VIEW\G
*************************** 1. row ***************************
          JobId: 12090
      TableName: sales_records
     CreateTime: 2022-08-25 19:41:10
   FinishedTime: 2022-08-25 19:41:39
  BaseIndexName: sales_records
RollupIndexName: store_amt
       RollupId: 12091
  TransactionId: 10
          State: FINISHED
            Msg: 
       Progress: NULL
        Timeout: 86400
1 row in set (0.00 sec)
```

`RollupIndexName`のセクションは、同期マテリアライズドビューの名前を示し、`State`のセクションはビルドが完了しているかどうかを示します。

## 直接同期マテリアライズドビューをクエリする

同期マテリアライズドビューは、基本テーブルのインデックスであり、物理的なテーブルではないため、`[_SYNC_MV_]`ヒントを使用してのみクエリできます:

```SQL
-- ヒント内の角かっこ[]を省略しないでください。
MySQL > SELECT * FROM store_amt [_SYNC_MV_];
+----------+----------+
| store_id | sale_amt |
+----------+----------+
|        2 |     6948 |
|        3 |     8734 |
|        1 |     4319 |
|        2 |     9515 |
|        3 |     4212 |
|        1 |     8573 |
+----------+----------+
```

> **注意**
>
> 現時点では、StarRocksは、同期マテリアライズドビュー内の列の名前を自動的に生成します。たとえば、それらにエイリアスを指定していてもです。

## 同期マテリアライズドビューを使用したクエリの書き直しと加速

作成した同期マテリアライズドビューには、クエリ文に従って事前に計算された完全な結果セットが含まれています。後続のクエリはそのデータを使用します。前回準備した際と同じクエリを実行してクエリ時間をテストできます。

```Plain
MySQL > SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
+----------+-----------------+
| store_id | sum(`sale_amt`) |
+----------+-----------------+
|        2 |           16463 |
|        3 |           12946 |
|        1 |           12892 |
+----------+-----------------+
3 rows in set (0.01 sec)
```

このクエリ時間が0.01秒に短縮されていることがわかります。

## クエリが同期マテリアライズドビューにヒットしたかどうかを確認する

クエリが同期マテリアライズドビューにヒットしたかどうかを確認するために、再度EXPLAINコマンドを実行します。

```Plain
MySQL > EXPLAIN SELECT store_id, SUM(sale_amt) FROM sales_records GROUP BY store_id;
+-----------------------------------------------------------------------------+
| Explain String                                                              |
+-----------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                             |
|  OUTPUT EXPRS:3: store_id | 6: sum                                          |
|   PARTITION: UNPARTITIONED                                                  |
|                                                                             |
|   RESULT SINK                                                               |
|                                                                             |
|   4:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 1                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: HASH_PARTITIONED: 3: store_id                                  |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 04                                                         |
|     UNPARTITIONED                                                           |
|                                                                             |
|   3:AGGREGATE (merge finalize)                                              |
|   |  output: sum(6: sum)                                                    |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   2:EXCHANGE                                                                |
|                                                                             |
| PLAN FRAGMENT 2                                                             |
|  OUTPUT EXPRS:                                                              |
|   PARTITION: RANDOM                                                         |
|                                                                             |
|   STREAM DATA SINK                                                          |
|     EXCHANGE ID: 02                                                         |
|     HASH_PARTITIONED: 3: store_id                                           |
|                                                                             |
|   1:AGGREGATE (update serialize)                                            |
|   |  STREAMING                                                              |
|   |  output: sum(5: sale_amt)                                               |
|   |  group by: 3: store_id                                                  |
|   |                                                                         |
|   0:OlapScanNode                                                            |
|      TABLE: sales_records                                                   |
|      PREAGGREGATION: ON                                                     |
|      partitions=1/1                                                         |
|      rollup: store_amt                                                      |
|      tabletRatio=10/10                                                      |
|      tabletList=12092,12096,12100,12104,12108,12112,12116,12120,12124,12128 |
|      cardinality=6                                                          |
|      avgRowSize=2.0                                                         |
|      numNodes=0                                                             |
+-----------------------------------------------------------------------------+
45 rows in set (0.00 sec)
```

クエリプロファイルの`rollup`セクションの値が`store_amt`になっており、これは作成した同期マテリアライズドビューを示しています。つまり、このクエリは同期マテリアライズドビューにヒットしたことを意味します。

## 同期マテリアライズドビューを表示する

テーブルおよびその従属する同期マテリアライズドビューのスキーマを確認するには、DESC \<tbl_name\> ALLを実行できます。

```Plain
MySQL > DESC sales_records ALL;
+---------------+---------------+-----------+--------+------+-------+---------+-------+
| IndexName     | IndexKeysType | Field     | Type   | Null | Key   | Default | Extra |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
| sales_records | DUP_KEYS      | record_id | INT    | Yes  | true  | NULL    |       |
|               |               | seller_id | INT    | Yes  | true  | NULL    |       |
|               |               | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_date | DATE   | Yes  | false | NULL    | NONE  |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | NONE  |
|               |               |           |        |      |       |         |       |
| store_amt     | AGG_KEYS      | store_id  | INT    | Yes  | true  | NULL    |       |
|               |               | sale_amt  | BIGINT | Yes  | false | NULL    | SUM   |
+---------------+---------------+-----------+--------+------+-------+---------+-------+
8 rows in set (0.00 sec)
```

## 同期マテリアライズドビューを削除する

次の状況の場合、同期マテリアライズドビューを削除する必要があります。

- 誤ったマテリアライズドビューを作成してしまい、ビューが完了する前にそれを削除する必要がある場合。
- 多数のマテリアライズドビューを作成した結果、ロードパフォーマンスが著しく低下し、一部のマテリアライズドビューが重複している場合。
- 関連するクエリの頻度が低く、比較的高いクエリ遅延を容認できる場合。

### 完了していない同期マテリアライズドビューを削除する

作成中の同期マテリアライズドビューをキャンセルして削除することで、同期マテリアライズドビューを削除できます。まず、[同期マテリアライズドビューのビルド状況を確認](#check-the-building-status-of-a-synchronous-materialized-view)してマテリアライズドビューの作成タスクのジョブID `JobID`を取得する必要があります。ジョブIDを取得した後、CANCLE ALTERコマンドで作成タスクをキャンセルする必要があります。

```Plain
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 既存の同期マテリアライズドビューを削除する

[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)コマンドを使用して、既存の同期マテリアライズドビューを削除できます。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## ベストプラクティス

### 正確なカウントのユニーク数

次の例は、広告ビジネス分析テーブル`advertiser_view_record`に基づいています。このテーブルは広告の閲覧日`click_time`、広告の名前`advertiser`、広告のチャンネル`channel`、および閲覧したユーザーのID`user_id`を記録しています。

```SQL
CREATE TABLE advertiser_view_record(
    click_time DATE,
    advertiser VARCHAR(10),
    channel VARCHAR(10),
    user_id INT
) distributed BY hash(click_time);
```

分析の焦点は、広告のUVにあります。

```SQL
SELECT advertiser, channel, count(distinct user_id)
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

正確なカウントのユニーク数を高速化するには、このテーブルに基づいて同期マテリアライズドビューを作成し、bitmap_union関数を使用してデータを事前集約できます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, bitmap_union(to_bitmap(user_id))
FROM advertiser_view_record
```
広告主、チャネルごとにGROUP BYします。
```

マテリアライズドビューが作成された後、サブクエリ内の`count(distinct user_id)`は自動的に`bitmap_union_count (to_bitmap(user_id))`に書き換えられ、同期マテリアライズドビューを利用できるようになります。

### 近似カウントの取得

上記の`advertiser_view_record`テーブルを例に挙げます。近似カウントを加速するために、このテーブルに基づいた同期マテリアライズドビューを作成し、[hll_union()](../sql-reference/sql-functions/aggregate-functions/hll_union.md) 関数を使用してデータを事前集計することができます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, hll_union(hll_hash(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### 追加のソートキーを設定

ベーステーブル`tableA`には、`k1`、`k2`、`k3`の列が含まれており、`k1`と`k2`だけがソートキーです。サブクエリに`where k3=x`を含むクエリを加速する必要がある場合、`k3`を最初の列として同期マテリアライズドビューを作成することができます。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA
```

## 集約関数の対応関係

同期マテリアライズドビューでクエリが実行された場合、元のクエリ文は自動的に書き換えられて同期マテリアライズドビューに格納された中間結果をクエリするために使用されます。以下の表は、元のクエリ内の集約関数と同期マテリアライズドビューを構築するために使用される集約関数との対応関係を示しています。ビジネスシナリオに応じて、適切な集約関数を選択して同期マテリアライズドビューを構築することができます。

| **元のクエリ内の集約関数**               | **マテリアライズドビュー内の集約関数** |
| ----------------------------------------- | ------------------------------------- |
| sum                                       | sum                                   |
| min                                       | min                                   |
| max                                      | max                                   |
| count                                     | count                                 |
| bitmap_union, bitmap_union_count, count(distinct) | bitmap_union                        |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                 |
```