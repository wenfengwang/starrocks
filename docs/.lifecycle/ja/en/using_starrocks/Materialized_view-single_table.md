---
displayed_sidebar: English
---

# 同期マテリアライズドビュー

このトピックでは、**同期マテリアライズドビュー（Rollup）**の作成、使用、および管理方法について説明します。

同期マテリアライズドビューの場合、ベーステーブルのすべての変更は、対応する同期マテリアライズドビューに同時に更新されます。同期マテリアライズドビューのリフレッシュは自動的にトリガーされます。同期マテリアライズドビューは、保守と更新のコストが大幅に低いため、リアルタイムの単一テーブル集計クエリの透過的な高速化に適しています。

StarRocksの同期マテリアライズドビューは、[デフォルトカタログ](../data_source/catalog/default_catalog.md)の単一ベーステーブルでのみ作成できます。これらは基本的に、非同期マテリアライズドビューのような物理テーブルではなく、クエリアクセラレーションのための特別なインデックスです。

v2.4以降、StarRocksは非同期マテリアライズドビューを提供し、複数のテーブルとより多くの集計演算子での作成をサポートします。**非同期マテリアライズドビュー**の使用方法については、[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。

:::note
現在、同期マテリアライズドビューは共有データクラスターではサポートされていません。
:::

次の表は、StarRocks v2.5、v2.4の非同期マテリアライズドビュー（ASYNC MVs）と同期マテリアライズドビュー（SYNC MV）を、それらがサポートする機能の観点から比較したものです：

|                       | **単一テーブル集計** | **複数テーブル結合** | **クエリ書き換え** | **リフレッシュ戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | 複数テーブル：<ul><li>デフォルトカタログ</li><li>外部カタログ (v2.5)</li><li>既存のマテリアライズドビュー (v2.5)</li><li>既存のビュー (v3.1)</li></ul> |
| **SYNC MV (Rollup)**  | [集計関数](#correspondence-of-aggregate-functions)の選択肢が限られている | いいえ | はい | データロード中の同期リフレッシュ | デフォルトカタログ内の単一テーブル |

## 基本概念

- **ベーステーブル**

  ベーステーブルは、マテリアライズドビューの駆動テーブルです。

  StarRocksの同期マテリアライズドビューにおいて、ベーステーブルは[デフォルトカタログ](../data_source/catalog/default_catalog.md)の単一ネイティブテーブルでなければなりません。StarRocksは、Duplicate Keyテーブル、Aggregateテーブル、およびUnique Keyテーブルに対する同期マテリアライズドビューの作成をサポートしています。

- **リフレッシュ**

  同期マテリアライズドビューは、ベーステーブルのデータが変更されるたびに自身を更新します。リフレッシュを手動でトリガーする必要はありません。

- **クエリ書き換え**

  クエリ書き換えとは、マテリアライズドビューが構築されたベーステーブル上でのクエリ実行時に、システムが自動的に事前計算された結果をクエリに再利用できるかどうかを判断することを意味します。再利用可能であれば、システムは関連するマテリアライズドビューから直接データをロードし、時間とリソースを消費する計算や結合を避けます。

  同期マテリアライズドビューは、いくつかの集計演算子に基づくクエリ書き換えをサポートしています。詳細は[集計関数の対応](#correspondence-of-aggregate-functions)を参照してください。

## 準備

同期マテリアライズドビューを作成する前に、データウェアハウスが同期マテリアライズドビューを通じたクエリアクセラレーションに適しているかどうかを確認してください。たとえば、クエリが特定のサブクエリステートメントを再利用しているかどうかを確認します。

以下の例は、トランザクションID `record_id`、販売員ID `seller_id`、店舗ID `store_id`、日付 `sale_date`、および各トランザクションの売上金額 `sale_amt` を含むテーブル `sales_records` に基づいています。以下の手順に従ってテーブルを作成し、データを挿入してください：

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

このビジネスシナリオでは、さまざまな店舗の売上金額に関する頻繁な分析が求められます。結果として、`sum()`関数は各クエリで使用され、大量の計算リソースを消費します。クエリを実行してその時間を記録し、EXPLAINコマンドを使用してクエリプロファイルを表示することができます。

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
|                                                                             |

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

クエリには約0.02秒かかりますが、クエリプロファイルの`rollup`フィールドの値がベーステーブルである`sales_records`であるため、クエリを高速化するための同期マテリアライズドビューは使用されていません。

## 同期マテリアライズドビューの作成

[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を使用して、特定のクエリ文に基づいて同期マテリアライズドビューを作成できます。

`sales_records`テーブルと上記のクエリステートメントに基づいて、以下の例では`store_amt`という同期マテリアライズドビューを作成して、各店舗の売上金額の合計を分析します。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **注意**
>
> - 同期マテリアライズドビューで集約関数を使用する場合、GROUP BY句を使用し、SELECTリストに少なくとも1つのGROUP BY列を指定する必要があります。
> - 同期マテリアライズドビューは、複数の列に対する1つの集約関数の使用をサポートしていません。`sum(a+b)`の形式のクエリステートメントはサポートされていません。
> - 同期マテリアライズドビューは、1つの列に対する複数の集約関数の使用をサポートしていません。`select sum(a), min(a) from table`の形式のクエリステートメントはサポートされていません。
> - 同期マテリアライズドビューの作成時にJOINはサポートされていません。
> - ALTER TABLE DROP COLUMNを使用してベーステーブルの特定の列を削除する場合、すべての同期マテリアライズドビューがその列を含んでいないことを確認する必要があります。そうでないと、列の削除操作は実行できません。同期マテリアライズドビューで使用されている列を削除するには、まずその列を含むすべての同期マテリアライズドビューを削除し、その後で列を削除する必要があります。
> - テーブルに対して多くの同期マテリアライズドビューを作成すると、データのロード効率に影響を与えます。ベーステーブルにデータがロードされると、同期マテリアライズドビューとベーステーブルのデータが同時に更新されます。ベーステーブルに`n`個の同期マテリアライズドビューが含まれている場合、ベーステーブルへのデータのロード効率は、`n`個のテーブルにデータをロードするのとほぼ同じです。
> - 現在、StarRocksは複数の同期マテリアライズドビューを同時に作成することをサポートしていません。新しい同期マテリアライズドビューは、前のものが完了した後にのみ作成できます。
> - 共有データStarRocksクラスターは、同期マテリアライズドビューをサポートしていません。

## 同期マテリアライズドビューの構築状況を確認する

同期マテリアライズドビューの作成は非同期操作です。CREATE MATERIALIZED VIEWを成功裏に実行すると、マテリアライズドビューの作成タスクが正常に提出されたことを示します。データベース内の同期マテリアライズドビューの構築状況は、[SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)を使用して確認できます。

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

`RollupIndexName`セクションは同期マテリアライズドビューの名前を示し、`State`セクションは構築が完了したかどうかを示します。

## 同期マテリアライズドビューを直接クエリする

同期マテリアライズドビューは本質的にベーステーブルのインデックスであり、物理テーブルではないため、ヒント`[_SYNC_MV_]`を使用してのみ同期マテリアライズドビューをクエリできます。

```SQL
-- ヒントの中のブラケット[]を省略しないでください。
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
> 現在、StarRocksは、同期マテリアライズドビューのカラムにエイリアスを指定していても、カラム名を自動的に生成します。

## 同期マテリアライズドビューを使用したクエリの書き換えと高速化

作成した同期マテリアライズドビューには、クエリステートメントに従って事前に計算された結果の完全なセットが含まれています。後続のクエリはその中のデータを使用します。準備で行ったのと同じクエリを実行してクエリ時間をテストすることができます。

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

クエリ時間が0.01秒に短縮されていることが観察できます。

## クエリが同期マテリアライズドビューにヒットしているかを確認する

再びEXPLAINコマンドを実行して、クエリが同期マテリアライズドビューにヒットしているかどうかを確認します。

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

クエリプロファイルの`rollup`セクションの値が`store_amt`になっており、これは同期マテリアライズドビューであることを意味します。つまり、このクエリは同期マテリアライズドビューを利用しています。

## 同期マテリアライズドビューの表示

`DESC <tbl_name> ALL`を実行して、テーブルとその下位の同期マテリアライズドビューのスキーマをチェックできます。

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

## 同期マテリアライズドビューの削除

次の状況では、同期マテリアライズドビューを削除する必要があります。

- 誤ってマテリアライズドビューを作成したため、構築が完了する前に削除する必要があります。
- 作成したマテリアライズドビューが多すぎてロードパフォーマンスが大幅に低下し、一部のマテリアライズドビューが重複している場合。
- 関連するクエリの頻度が低く、比較的高いクエリ遅延を許容できる場合。

### 未完成の同期マテリアライズドビューを削除する

作成中の同期マテリアライズドビューは、進行中の作成タスクをキャンセルすることで削除できます。まず、[同期マテリアライズドビューの構築状況を確認](#同期マテリアライズドビューの構築状況を確認)して、マテリアライズドビュー作成タスクのジョブID `JobID`を取得する必要があります。ジョブIDを取得したら、CANCEL ALTERコマンドを使用して作成タスクをキャンセルします。

```Plain
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 既存の同期マテリアライズドビューを削除する

既存の同期マテリアライズドビューは、[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)コマンドで削除できます。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## ベストプラクティス

### 正確なカウントディスティンクト

次の例は、広告が表示された日付`click_time`、広告主`advertiser`、広告のチャネル`channel`、および広告を閲覧したユーザーのID `user_id`を記録する広告ビジネス分析テーブル`advertiser_view_record`に基づいています。

```SQL
CREATE TABLE advertiser_view_record(
    click_time DATE,
    advertiser VARCHAR(10),
    channel VARCHAR(10),
    user_id INT
) DISTRIBUTED BY HASH(click_time);
```

分析は主に広告のUV（ユニークビジター数）に焦点を当てています。

```SQL
SELECT advertiser, channel, COUNT(DISTINCT user_id)
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

正確なカウントディスティンクトを高速化するために、このテーブルに基づいて同期マテリアライズドビューを作成し、bitmap_union関数を使用してデータを事前集計することができます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, BITMAP_UNION(TO_BITMAP(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

同期マテリアライズドビューが作成された後、後続のクエリでのサブクエリ`COUNT(DISTINCT user_id)`は自動的に`BITMAP_UNION_COUNT(TO_BITMAP(user_id))`に書き換えられ、同期マテリアライズドビューを利用できるようになります。

### おおよそのカウントディスティンクト

上記の`advertiser_view_record`テーブルを再度例に挙げます。おおよそのカウントディスティンクトを高速化するために、このテーブルに基づいて同期マテリアライズドビューを作成し、[HLL_UNION()](../sql-reference/sql-functions/aggregate-functions/hll_union.md)関数を使用してデータを事前集計することができます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, HLL_UNION(HLL_HASH(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### 追加のソートキーを設定

ベーステーブル`tableA`には`k1`、`k2`、`k3`のカラムがあり、`k1`と`k2`のみがソートキーです。サブクエリ`WHERE k3 = x`を含むクエリを高速化する必要がある場合、`k3`を最初のカラムとする同期マテリアライズドビューを作成できます。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA;
```

## 集計関数の対応

同期マテリアライズドビューを使用してクエリが実行されると、元のクエリステートメントは自動的に書き換えられ、同期マテリアライズドビューに格納されている中間結果をクエリするために使用されます。以下の表は、元のクエリの集計関数と、同期マテリアライズドビューを構築するために使用される集計関数の対応を示しています。ビジネスシナリオに応じて、対応する集計関数を選択して同期マテリアライズドビューを構築できます。

| **元のクエリの集計関数**           | **マテリアライズドビューの集計関数** |
| ------------------------------------------------------ | ----------------------------------------------- |
| sum                                                    | sum                                             |
| min                                                    | min                                             |
| max                                                    | max                                             |
| count                                                  | count                                           |
| bitmap_union, bitmap_union_count, count(distinct)      | bitmap_union                                    |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                                       |
