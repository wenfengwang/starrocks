---
displayed_sidebar: "Japanese"
---

# 同期素材化ビュー

このトピックでは、**同期素材化ビュー（ロールアップ）**の作成、使用、および管理方法について説明します。

同期素材化ビューでは、基本テーブルのすべての変更が対応する同期素材化ビューに同時に更新されます。同期素材化ビューのリフレッシュは自動的にトリガされます。同期素材化ビューはメンテナンスおよびアップデートが非常に安価であり、リアルタイムの単一テーブル集計クエリの透過的な高速化に適しています。

StarRocksの同期素材化ビューは、[デフォルトカタログ](../data_source/catalog/default_catalog.md)の単一の基本テーブルにのみ作成できます。これらは、物理的なテーブルではなく、クエリの加速用の特別なインデックスです。

v2.4以降、StarRocksでは複数のテーブルで作成が可能であり、より多くの集計演算子をサポートする非同期素材化ビューが提供されています。**非同期素材化ビュー**の使用については、[非同期素材化ビュー](../using_starrocks/Materialized_view.md)を参照してください。

以下の表は、StarRocks v2.5、v2.4の非同期素材化ビュー（ASYNC MVs）と同期素材化ビュー（SYNC MV）を、サポートされている機能の観点から比較したものです。

|                       | **単一テーブル集計** | **複数テーブル結合** | **クエリの置き換え** | **リフレッシュ戦略** | **基本テーブル** |
| --------------------- | --------------------- | --------------------- | ----------------- | --------------------- | -------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | デフォルトカタログからの複数のテーブル:<ul><li>デフォルトカタログ</li><li>外部カタログ（v2.5）</li><li>既存の素材化ビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **SYNC MV（ロールアップ）**  | [集計関数の選択肢が限られています](#aggregate-functionsにおける対応) | いいえ | はい | データの読み込み中に同期的にリフレッシュ | デフォルトカタログ内の単一のテーブル |

## 基本概念

- **基本テーブル**

  基本テーブルは素材化ビューのドライビングテーブルです。

  StarRocksの同期素材化ビューでは、基本テーブルは[デフォルトカタログ](../data_source/catalog/default_catalog.md)からの単一のネイティブテーブルである必要があります。StarRocksは、Duplicate Keyテーブル、集計テーブル、およびUnique Keyテーブルでの同期素材化ビューの作成をサポートしています。

- **リフレッシュ**

  同期素材化ビューは、基本テーブルのデータが変更されるたびに自動的に更新されます。リフレッシュを手動でトリガーする必要はありません。

- **クエリの置き換え**

  クエリの置き換えとは、素材化ビューを使用して構築された基本テーブルのクエリを実行する際、システムが自動的に事前に計算された素材化ビューの結果がクエリで再利用可能かどうかを判断することを意味します。再利用可能であれば、システムはリソースを消費する計算や結合を回避するために、関連する素材化ビューからデータを直接ロードします。

  同期素材化ビューは、一部の集計演算子に基づいたクエリの置き換えをサポートしています。詳細については、[集計関数における対応](#aggregate-functionsにおける対応)を参照してください。

## 準備

同期素材化ビューを作成する前に、データウェアハウスが同期素材化ビューを使用したクエリの高速化に適しているかどうかを確認してください。たとえば、クエリが特定のサブクエリステートメントを再利用しているかどうかを確認します。

以下の例は、トランザクションID `record_id`、販売者ID `seller_id`、店舗ID `store_id`、日付 `sale_date`、販売金額 `sale_amt` を含む `sales_records` テーブルに基づいています。このテーブルを作成し、データを挿入する手順は以下のとおりです：

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

この例のビジネスシナリオでは、異なる店舗の販売金額を頻繁に分析することが求められています。その結果、各クエリで `sum()` 関数が使用され、大量の計算リソースを消費しています。クエリを実行してその時間を記録し、EXPLAINコマンドを使用してそのクエリプロファイルを表示できます。

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

クエリが約0.02秒かかり、クエリプロファイルの `rollup` フィールドの値が基本テーブルである `sales_records` であるため、同期素材化ビューはクエリの高速化に使用されていないことがわかります。

## 同期素材化ビューの作成

特定のクエリステートメントに基づいて同期素材化ビューを作成するには、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を使用します。

`sales_records` テーブルと上記のクエリステートメントに基づいて、以下の例では同期素材化ビュー `store_amt` を作成し、各店舗での販売金額の合計を分析します。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **注意**
>
> - 同期素材化ビューで集計関数を使用する場合、必ずSELECTリストでGROUP BY句を使用し、少なくとも1つのGROUP BY列を指定する必要があります。
> - 同期素材化ビューでは、1つの列に対して複数の集計関数を使用することはできません。`sum(a+b)` といったクエリステートメントはサポートされていません。
> - 同期素材化ビューでは、1つの列に対して複数の集計関数を使用することはできません。`select sum(a), min(a) from table` といったクエリステートメントはサポートされていません。
> - 同期素材化ビューの作成時にJOINはサポートされていません。
> - 基本テーブルの特定の列を削除するために`ALTER TABLE DROP COLUMN`を使用する場合、基本テーブルのすべての同期素材化ビューが削除されていないことを確認する必要があります。削除されていない場合は、列の削除操作を実行できません。同期素材化ビューに含まれる列を削除するには、まず列を含むすべての同期素材化ビューを削除し、その後列を削除する必要があります。
> - テーブルに対して多数の同期素材化ビューを作成すると、データのロード効率に影響を与えます。基本テーブルにデータをロードしている間、同期素材化ビューと基本テーブルのデータが同期して更新されます。基本テーブルが`n`個の同期素材化ビューを含む場合、基本テーブルへのデータのロード効率は`n`個のテーブルへのデータのロードとほぼ同じです。
> - 現時点では、StarRocksでは複数の同期マテリアライズド・ビューを同時に作成することはサポートされていません。新しい同期マテリアライズド・ビューは、前の作業が完了したときにのみ作成できます。
> - 共有データStarRocksクラスターでは、同期マテリアライズド・ビューはサポートされていません。

## 同期マテリアライズド・ビューの構築ステータスを確認

同期マテリアライズド・ビューの作成は非同期の操作です。CREATE MATERIALIZED VIEWを成功裏に実行すると、マテリアライズド・ビューの作業が正常に提出されたことを示します。同期マテリアライズド・ビューの構築ステータスは、[SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)を使用してデータベース内で表示できます。

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

`RollupIndexName`のセクションは同期マテリアライズド・ビューの名前を示し、`State`のセクションは構築が完了しているかどうかを示します。

## 直接的な同期マテリアライズド・ビューのクエリ

同期マテリアライズド・ビューは基本的には物理的なテーブルではなく、ベース・テーブルのインデックスですので、ヒント`[_SYNC_MV_]`を使用して同期マテリアライズド・ビューをクエリすることができます。

```SQL
-- ヒント内の角かっこ[]は省かないでください。
MySQL > SELECT * FROM store_amt [_SYNC_MV_];
+----------+----------+
| store_id | sale_amt  |
+----------+----------+
|        2 |     6948  |
|        3 |     8734  |
|        1 |     4319  |
|        2 |     9515  |
|        3 |     4212  |
|        1 |     8573  |
+----------+----------+
```

> **注意**
>
> 現時点では、StarRocksは同期マテリアライズド・ビューの列名を自動的に生成しますが、それらにエイリアスを指定していても同様です。

## 同期マテリアライズド・ビューを使用したクエリの書き換えと加速

作成した同期マテリアライズド・ビューには、クエリ文に従って事前に計算された完全な結果セットが含まれています。これ以降のクエリは、それ内部のデータを使用します。準備時と同じクエリを実行して、クエリ時間をテストできます。

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

クエリ時間が0.01秒に短縮されていることが確認できます。

## クエリが同期マテリアライズド・ビューにヒットするかを確認

再びEXPLAINコマンドを実行して、クエリが同期マテリアライズド・ビューにヒットするかを確認できます。

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

クエリプロファイルの`rollup`セクションの値が`store_amt`になっており、これは作成した同期マテリアライズド・ビューを指しています。つまり、このクエリは同期マテリアライズド・ビューにヒットしたことを意味します。

## 同期マテリアライズド・ビューを表示

DESC \<tbl_name\> ALLを実行して、テーブルとその従属的な同期マテリアライズド・ビューのスキーマを確認できます。

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

## 同期マテリアライズド・ビューの削除

以下の場合には、同期マテリアライズド・ビューを削除する必要があります：

- 誤ったマテリアライズド・ビューを作成しており、構築が完了する前に削除する必要がある場合。
- 多くのマテリアライズド・ビューを作成しており、その結果ロードパフォーマンスが大きく低下し、いくつかのマテリアライズド・ビューが重複している場合。
- 関連するクエリの頻度が低く、かなり高いクエリ遅延を許容できる場合。

### 完了していない同期マテリアライズド・ビューの削除

作成中の同期マテリアライズド・ビューをキャンセルすることで削除できます。まず、[同期マテリアライズド・ビューの構築ステータスを確認](#check-the-building-status-of-a-synchronous-materialized-view)してマテリアライズド・ビュー作成タスクのJob ID `JobID`を取得する必要があります。Job IDを取得した後は、CANCE ALTERコマンドで作成タスクをキャンセルする必要があります。

```Plain
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 既存の同期マテリアライズド・ビューの削除

[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)コマンドを使用して既存の同期マテリアライズド・ビューを削除できます。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## ベストプラクティス

### 正確な重複数のカウント

以下の例は広告ビジネス分析テーブル`advertiser_view_record`をベースにしており、広告が閲覧された日付`click_time`、広告の名前`advertiser`、広告のチャンネル`channel`、閲覧したユーザーのID`user_id`を記録しています。

```SQL
CREATE TABLE advertiser_view_record(
    click_time DATE,
    advertiser VARCHAR(10),
    channel VARCHAR(10),
    user_id INT
) distributed BY hash(click_time);
```

分析は主に広告のUVに焦点を当てています。

```SQL
SELECT advertiser, channel, count(distinct user_id)
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

正確な重複数のカウントを加速するために、このテーブルに基づいて同期マテリアライズド・ビューを作成し、bitmap_union関数を使用してデータを事前に集約できます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, bitmap_union(to_bitmap(user_id))
FROM advertiser_view_record
```
広告主、チャネルごとにグループ化する;

```

同期マテリアライズドビューが作成された後、サブクエリの`count(distinct user_id)`は自動的に`bitmap_union_count(to_bitmap(user_id))`に書き換えられ、同期マテリアライズドビューにアクセスできるようになります。

### 近似カウントのディスティンクト

前述の`advertiser_view_record`テーブルを再度例にしてください。 近似カウントのディスティンクトを高速化するには、このテーブルに基づいて同期マテリアライズドビューを作成し、[hll_union()](../sql-reference/sql-functions/aggregate-functions/hll_union.md)関数を使用してデータを事前に集計することができます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, hll_union(hll_hash(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### 追加のソートキーを設定する

基本テーブル`tableA`には、`k1`、`k2`、`k3`の列が含まれており、`k1`と`k2`のみがソートキーであるとします。 サブクエリ`where k3=x`を含むクエリを高速化する必要がある場合は、`k3`を最初の列として持つ同期マテリアライズドビューを作成することができます。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA
```

## 集約関数の対応

クエリを同期マテリアライズドビューで実行する場合、元のクエリ文は自動的に書き換えられ、同期マテリアライズドビューに格納された中間結果へのクエリに使用されます。 次の表は、元のクエリ内の集約関数と同期マテリアライズドビューの構築に使用される集約関数との対応を示しています。 ビジネスシナリオに応じて対応する集約関数を選択して同期マテリアライズドビューを構築することができます。

| **元のクエリ内の集約関数**           | **マテリアライズドビュー内の集約関数** |
| ------------------------------------- | ------------------------------------- |
| sum                                   | sum                                   |
| min                                   | min                                   |
| max                                   | max                                   |
| count                                 | count                                 |
| bitmap_union, bitmap_union_count, count(distinct) | bitmap_union                        |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                           |
```