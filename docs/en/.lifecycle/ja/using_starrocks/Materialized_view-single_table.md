---
displayed_sidebar: "Japanese"
---

# 同期マテリアライズドビュー

このトピックでは、**同期マテリアライズドビュー（ロールアップ）**の作成、使用、および管理方法について説明します。

同期マテリアライズドビューでは、ベーステーブルの変更は同期マテリアライズドビューにも同時に反映されます。同期マテリアライズドビューのリフレッシュは自動的にトリガされます。同期マテリアライズドビューはメンテナンスおよび更新が非常に低コストであり、リアルタイムの単一テーブル集計クエリの透過的な高速化に適しています。

StarRocksの同期マテリアライズドビューは、[デフォルトカタログ](../data_source/catalog/default_catalog.md)の単一のベーステーブル上のみ作成できます。これらは、非同期マテリアライズドビューのような物理テーブルではなく、クエリの高速化のための特別なインデックスです。

v2.4以降、StarRocksでは複数のテーブルおよびより多くの集計演算子をサポートする非同期マテリアライズドビューが提供されています。**非同期マテリアライズドビュー**の使用方法については、[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。

以下の表は、StarRocks v2.5、v2.4の非同期マテリアライズドビュー（ASYNC MV）と同期マテリアライズドビュー（SYNC MV）の機能の観点からの比較です。

|                       | **単一テーブルの集計** | **複数テーブルの結合** | **クエリの書き換え** | **リフレッシュ戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **ASYNC MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | デフォルトカタログからの複数のテーブル:<ul><li>デフォルトカタログ</li><li>外部カタログ（v2.5）</li><li>既存のマテリアライズドビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **SYNC MV (ロールアップ)**  | [集計関数の制限付き](#集計関数の対応) | いいえ | はい | データローディング中の同期リフレッシュ | デフォルトカタログの単一テーブル |

## 基本的な概念

- **ベーステーブル**

  ベーステーブルは、マテリアライズドビューの基になるテーブルです。

  StarRocksの同期マテリアライズドビューでは、ベーステーブルは[デフォルトカタログ](../data_source/catalog/default_catalog.md)の単一のネイティブテーブルである必要があります。StarRocksは、重複キーテーブル、集計テーブル、およびユニークキーテーブルに対して同期マテリアライズドビューを作成することができます。

- **リフレッシュ**

  同期マテリアライズドビューは、ベーステーブルのデータが変更されるたびに自動的に更新されます。リフレッシュを手動でトリガする必要はありません。

- **クエリの書き換え**

  クエリの書き換えとは、マテリアライズドビューが構築されたベーステーブルでクエリを実行する際に、システムが自動的にマテリアライズドビューの事前計算結果をクエリに再利用できるかどうかを判断することを意味します。再利用できる場合、システムは時間とリソースを消費する計算や結合を回避するために、関連するマテリアライズドビューからデータを直接ロードします。

  同期マテリアライズドビューは、一部の集計演算子に基づいてクエリの書き換えをサポートしています。詳細については、[集計関数の対応](#集計関数の対応)を参照してください。

## 準備

同期マテリアライズドビューを作成する前に、データウェアハウスが同期マテリアライズドビューを介したクエリの高速化に適しているかどうかを確認してください。たとえば、クエリが特定のサブクエリ文を再利用しているかどうかを確認します。

以下の例は、トランザクションID `record_id`、セールスパーソンID `seller_id`、ストアID `store_id`、日付 `sale_date`、および売上金額 `sale_amt` を含む `sales_records` テーブルを基にしています。次の手順に従ってテーブルを作成し、データを挿入します。

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

この例のビジネスシナリオでは、さまざまな店舗の売上金額の頻繁な分析が求められます。その結果、各クエリで `sum()` 関数が使用され、大量の計算リソースが消費されます。クエリの時間を記録し、EXPLAINコマンドを使用してクエリのプロファイルを表示できます。

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

クエリには、約0.02秒かかり、同期マテリアライズドビューは使用されていないことがわかります。クエリプロファイルの `rollup` フィールドの値が `sales_records` であるためです。これはベーステーブルです。

## 同期マテリアライズドビューの作成

特定のクエリ文に基づいて同期マテリアライズドビューを作成するには、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を使用します。

テーブル `sales_records` と上記のクエリ文に基づいて、次の例では同期マテリアライズドビュー `store_amt` を作成して各店舗の売上金額の合計を分析します。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **注意**
>
> - 同期マテリアライズドビューで集計関数を使用する場合、SELECTリストでGROUP BY句を使用し、少なくとも1つのGROUP BYカラムを指定する必要があります。
> - 同期マテリアライズドビューでは、複数の列に対して1つの集計関数を使用することはサポートされていません。`sum(a+b)` の形式のクエリ文はサポートされていません。
> - 同期マテリアライズドビューでは、1つの列に対して複数の集計関数を使用することはサポートされていません。`select sum(a), min(a) from table` の形式のクエリ文はサポートされていません。
> - JOINおよびWHERE句は、同期マテリアライズドビューの作成時にサポートされていません。
> - ベーステーブルの特定の列をALTER TABLE DROP COLUMNを使用して削除する場合、削除される列を含むすべての同期マテリアライズドビューが存在しないことを確認する必要があります。それ以外の場合、削除操作を実行できません。同期マテリアライズドビューに使用される列を削除するには、まず列を含むすべての同期マテリアライズドビューを削除し、その後列を削除する必要があります。
> - テーブルに対して多数の同期マテリアライズドビューを作成すると、データのロード効率に影響を与えます。ベーステーブルにデータがロードされている間、同期マテリアライズドビューとベーステーブルのデータは同期して更新されます。ベーステーブルに`n`個の同期マテリアライズドビューが含まれている場合、ベーステーブルへのデータのロード効率は、`n`個のテーブルへのデータのロード効率とほぼ同じです。
> - 現在、StarRocksは複数の同期マテリアライズドビューを同時に作成することをサポートしていません。前の同期マテリアライズドビューが完了した後に、新しい同期マテリアライズドビューを作成することができます。

## 同期マテリアライズドビューのビルド状態の確認

同期マテリアライズドビューの作成は非同期の操作です。CREATE MATERIALIZED VIEWを正常に実行すると、マテリアライズドビューの作成タスクが正常に送信されたことを示します。[SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)を使用して、データベース内の同期マテリアライズドビューのビルド状態を表示できます。

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

`RollupIndexName` セクションには、同期マテリアライズドビューの名前が表示され、`State` セクションにはビルドが完了しているかどうかが表示されます。

## 同期マテリアライズドビューを直接クエリする

同期マテリアライズドビューは、基本的には物理テーブルではなくベーステーブルのインデックスですので、ヒント `[_SYNC_MV_]` を使用してのみクエリできます。

```SQL
-- ヒントの[]内の括弧を省略しないでください。
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
> 現在、StarRocksは、列に別名を指定した場合でも、同期マテリアライズドビューの列の名前を自動的に生成します。

## 同期マテリアライズドビューを使用したクエリの書き換えと高速化

作成した同期マテリアライズドビューには、クエリ文に基づいて事前計算された結果の完全なセットが含まれています。その後のクエリでは、そのデータを使用します。準備で行ったのと同じクエリを実行して、クエリの時間をテストできます。

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

クエリの時間が0.01秒に短縮されたことがわかります。

## クエリが同期マテリアライズドビューにヒットするかどうかを確認する

再度EXPLAINコマンドを実行して、クエリが同期マテリアライズドビューにヒットするかどうかを確認します。

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

クエリプロファイルの `rollup` セクションの値が `store_amt` になっていることがわかります。これは作成した同期マテリアライズドビューです。つまり、このクエリが同期マテリアライズドビューにヒットしていることを意味します。

## 同期マテリアライズドビューの表示

DESC \<tbl_name\> ALL を実行して、テーブルとその下位の同期マテリアライズドビューのスキーマを確認できます。

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

以下の場合には、同期マテリアライズドビューを削除する必要があります。

- 誤ったマテリアライズドビューを作成し、ビルドが完了する前に削除する必要がある場合。
- 多数のマテリアライズドビューを作成し、ロードパフォーマンスが大幅に低下し、一部のマテリアライズドビューが重複している場合。
- 関連するクエリの頻度が低く、比較的高いクエリ待ち時間を許容できる場合。

### 未完了の同期マテリアライズドビューの削除

作成中の同期マテリアライズドビューをキャンセルして削除するには、進行中の作成タスクをキャンセルすることで、同期マテリアライズドビューを削除できます。まず、マテリアライズドビューの作成タスクのジョブID `JobID` を[同期マテリアライズドビューのビルド状態の確認](#同期マテリアライズドビューのビルド状態の確認)を使用して取得します。ジョブIDを取得したら、CANCEL ALTERコマンドを使用して作成タスクをキャンセルします。

```Plain
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 既存の同期マテリアライズドビューの削除

[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md)コマンドを使用して、既存の同期マテリアライズドビューを削除できます。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## ベストプラクティス

### 正確な重複カウント

以下の例は、広告ビジネス分析テーブル `advertiser_view_record` を基にしています。このテーブルには、広告が表示された日付 `click_time`、広告の名前 `advertiser`、広告のチャネル `channel`、および広告を表示したユーザーのID `user_id` が記録されています。

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

正確な重複カウントを高速化するためには、このテーブルに基づいて同期マテリアライズドビューを作成し、bitmap_union関数を使用してデータを事前集約します。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, bitmap_union(to_bitmap(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

同期マテリアライズドビューが作成された後、サブクエリ `count(distinct user_id)` は自動的に `bitmap_union_count (to_bitmap(user_id))` に書き換えられ、同期マテリアライズドビューにヒットするようになります。

### 近似的な重複カウント

再度、上記の `advertiser_view_record` テーブルを使用して近似的な重複カウントを高速化するために、このテーブルに基づいて同期マテリアライズドビューを作成し、[hll_union()](../sql-reference/sql-functions/aggregate-functions/hll_union.md) 関数を使用してデータを事前集約します。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, hll_union(hll_hash(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### 追加のソートキーの設定

ベーステーブル `tableA` には、`k1`、`k2`、および `k3` の列が含まれており、`k1` と `k2` のみがソートキーです。`k3=x` のサブクエリを含むクエリを高速化する必要がある場合、`k3` を最初の列として持つ同期マテリアライズドビューを作成できます。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA
```

## 集計関数の対応

同期マテリアライズドビューでクエリが実行されると、元のクエリ文は自動的に書き換えられ、同期マテリアライズドビューに格納されている中間結果をクエリするために使用されます。次の表は、元のクエリで使用される集計関数と同期マテリアライズドビューの構築に使用される集計関数の対応関係を示しています。ビジネスシナリオに応じて、対応する集計関数を選択して同期マテリアライズドビューを構築することができます。

| **元のクエリでの集計関数**           | **マテリアライズドビューでの集計関数** |
| -------------------------------------- | -------------------------------- |
| sum                                    | sum                              |
| min                                    | min                              |
| max                                    | max                              |
| count                                  | count                            |
| bitmap_union、bitmap_union_count、count(distinct) | bitmap_union                     |
| hll_raw_agg、hll_union_agg、ndv、approx_count_distinct | hll_union                        |
