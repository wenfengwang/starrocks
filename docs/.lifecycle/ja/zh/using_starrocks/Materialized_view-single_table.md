---
displayed_sidebar: Chinese
---

# 同期物化ビュー

本文では、StarRocks で**同期物化ビュー（Rollup）**を作成、使用、管理する方法について説明します。

同期物化ビューでは、基表に対するすべてのデータ変更が物化ビューに自動的に同期更新されます。物化ビューを手動でリフレッシュする必要はなく、自動的に同期リフレッシュが行われます。同期物化ビューは管理コストと更新コストが低く、リアルタイムシナリオでの単一テーブル集約クエリの透過的な加速に適しています。

StarRocks の同期物化ビューは、[Default Catalog](../data_source/catalog/default_catalog.md) の単一基表に基づいてのみ作成でき、特殊なクエリ加速インデックスです。

バージョン2.4以降、StarRocks は**非同期物化ビュー**をサポートし、複数の基表に基づいて作成でき、より豊富な集約関数をサポートしています。詳細は [非同期物化ビュー](../using_starrocks/Materialized_view.md) を参照してください。

:::note
現在、StarRocks のストレージと計算の分離クラスターは同期物化ビューをサポートしていません。
:::

以下の表は、StarRocks 2.5および2.4の非同期物化ビューと同期物化ビュー（Rollup）をサポートする特徴を比較しています：

|                              | **単一テーブル集約** | **複数テーブル結合** | **クエリ書き換え** | **リフレッシュ戦略** | **基表** |
| ---------------------------- | ------------------- | -------------------- | ----------------- | -------------------- | -------- |
| **非同期物化ビュー** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | 複数テーブル構築をサポート。基表は以下から来ることができます：<ul><li>Default Catalog</li><li>External Catalog（v2.5）</li><li>既存の非同期物化ビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **同期物化ビュー（Rollup）** | 一部の集約関数のみ | いいえ | はい | インポート時に同期リフレッシュ | Default Catalog の単一テーブルに基づいてのみ構築をサポート |

## 関連概念

- **基表（Base Table）**

  物化ビューのドライブテーブル。

  StarRocks の同期物化ビューにおいて、基表は [Default catalog](../data_source/catalog/default_catalog.md) の単一内部テーブルのみ可能です。StarRocks は、明細モデル（Duplicate Key type）、集約モデル（Aggregate Key type）、更新モデル（Unique Key type）で同期物化ビューを作成することをサポートしています。

- **リフレッシュ（Refresh）**

  StarRocks の同期物化ビューのデータは、基表にデータがインポートされると自動的に更新され、手動でリフレッシュを呼び出す必要はありません。

- **クエリ書き換え（Query Rewrite）**

  クエリ書き換えとは、物化ビューが構築された基表に対するクエリを実行する際、システムが自動的に物化ビューの事前計算結果を再利用してクエリを処理できるかどうかを判断することです。再利用可能な場合、システムは関連する物化ビューから事前計算結果を直接読み取り、システムリソースと時間の消費を避けるために重複計算を行いません。

  StarRocks の同期物化ビューは、一部の集約演算子のクエリ書き換えをサポートしています。詳細は [集約関数のマッチング関係](#集約関数のマッチング関係) を参照してください。

## 準備

同期物化ビューを作成する前に、データウェアハウスがクエリを加速するために同期物化ビューを必要としているかどうかを確認する必要があります。例えば、データウェアハウスのクエリが特定のサブクエリステートメントを繰り返し使用しているかどうかを確認できます。

以下の例は `sales_records` テーブルに基づいており、各取引の取引ID `record_id`、販売員 `seller_id`、販売店 `store_id`、販売日 `sale_date`、販売額 `sale_amt` を含んでいます。以下のデータでテーブルを作成し、インポートします：

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

この例のビジネスシナリオでは、異なる店舗の売上を頻繁に分析する必要があり、そのためには sum() 関数を多用し、多くのシステムリソースを消費します。このクエリを実行してクエリの実行時間を記録し、EXPLAIN コマンドを使用してこのクエリの Query Profile を確認できます。

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

この時点でのクエリ時間は0.02秒であり、Query Profile の `rollup` 項目が `sales_records`（基表）と表示されていることから、このクエリが物化ビューを使用して加速されていないことがわかります。

## 同期物化ビューの作成

特定のクエリステートメントに対して物化ビューを作成するには、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md) ステートメントを使用できます。

以下の例では、上記のクエリステートメントに基づいて、`sales_records` テーブルに対して「販売店ごとにグループ化し、各販売店の全取引額を合計する」同期物化ビューを作成します。

```SQL
CREATE MATERIALIZED VIEW store_amt AS
SELECT store_id, SUM(sale_amt)
FROM sales_records
GROUP BY store_id;
```

> **注意**
>
> - 同期物化ビューで集約関数を使用する場合、クエリステートメントは GROUP BY 句を使用しなければならず、SELECT LIST には少なくとも1つのグループ化列が含まれている必要があります。
> - 同期物化ビューは、複数の列に対して単一の集約関数を使用することをサポートしておらず、`sum(a+b)` のようなクエリステートメントはサポートされていません。
> - 同期物化ビューは、同じ列に対して複数の集約関数を使用することをサポートしておらず、`select sum(a), min(a) from table` のようなクエリステートメントはサポートされていません。
> - 同期物化ビューの作成ステートメントは JOIN をサポートしていません。
> - ALTER TABLE DROP COLUMN を使用して基表の特定の列を削除する場合、その基表のすべての同期物化ビューに削除される列が含まれていないことを確認する必要があります。削除するには、その列を含むすべての同期物化ビューを削除してから、その列を削除する必要があります。
> - 1つのテーブルに多くの同期物化ビューを作成すると、インポートの効率が低下します。データをインポートする際、同期物化ビューと基表のデータは同時に更新され、基表に `n` 個の同期物化ビューがある場合、基表へのデータインポートの効率は `n` 個のテーブルにデータをインポートするのとほぼ同じになり、データインポートの速度が遅くなります。
> - 複数の同期物化ビューを同時に作成することはサポートされていません。現在の作成タスクが完了した後に次の作成タスクを実行できます。
> - StarRocks のストレージと計算の分離クラスターは、現在同期物化ビューをサポートしていません。

## 同期物化ビューの構築状態の確認


同期物化ビューの作成は非同期操作です。CREATE MATERIALIZED VIEW コマンドが成功すると、同期物化ビューの作成タスクが正常に送信されたことを意味します。現在のデータベースで同期物化ビューの構築状態を確認するには、[SHOW ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md) コマンドを使用できます。

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

ここで、`RollupIndexName` は同期物化ビューの名前で、`State` 項目が `FINISHED` であることは、同期物化ビューの構築が完了したことを意味します。

## 同期物化ビューを直接クエリする

同期物化ビューは本質的にはベーステーブルのインデックスであり、物理テーブルではないため、Hint `[_SYNC_MV_]` を使用して同期物化ビューをクエリする必要があります：

```SQL
-- Hint の角括弧[]を省略しないでください。
SELECT * FROM <mv_name> [_SYNC_MV_];
```

> **注意**
>
> 現在、StarRocks は同期物化ビューのカラムに自動的に名前を生成します。同期物化ビューのカラムに指定した Alias は効果がありません。

## 同期物化ビューを使用してクエリを高速化する

新しく作成された同期物化ビューは、上記のクエリ結果を事前計算して保存し、後続のクエリはこの結果を直接呼び出してクエリを高速化します。作成に成功した後、同じクエリを再度実行してクエリ時間をテストできます。

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

この時点で、クエリ時間は 0.01 秒に短縮されていることがわかります。

## クエリが同期物化ビューをヒットしているかを確認する

EXPLAIN コマンドを再度使用して、クエリが同期物化ビューをヒットしているかを確認できます。

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

この時点で、Query Profile の `rollup` 項目が `store_amt`（同期物化ビュー）と表示されていることから、クエリが同期物化ビューをヒットしていることがわかります。

## 同期物化ビューのテーブル構造を確認する

DESC \<tbl_name\> ALL コマンドを使用して、特定のテーブルのテーブル構造とそれに属するすべての同期物化ビューを確認できます。

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

## 同期物化ビューを削除する

以下の三つの状況で、同期物化ビューを削除する必要があります:

- 同期物化ビューが誤って作成されたため、作成中の同期物化ビューを削除する必要があります。
- 大量の同期物化ビューが作成され、データのインポート速度が遅くなり、一部の同期物化ビューが重複しています。
- 関連するクエリの頻度が低く、ビジネスシナリオが高いクエリ遅延を許容できる場合。

### 作成中の同期物化ビューを削除する

進行中の同期物化ビューの作成タスクをキャンセルすることで、作成中の同期物化ビューを削除できます。まず、[同期物化ビューの構築状態を確認する](#同期物化ビューの構築状態を確認する) セクションを参照して、その同期物化ビューのタスク ID `JobID` を取得する必要があります。タスク ID を取得した後、CANCEL ALTER コマンドを使用してその作成タスクをキャンセルします。

```SQL
CANCEL ALTER TABLE ROLLUP FROM sales_records (12090);
```

### 作成済みの同期物化ビューを削除する

[DROP MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/DROP_MATERIALIZED_VIEW.md) コマンドを使用して、作成済みの同期物化ビューを削除できます。

```SQL
DROP MATERIALIZED VIEW store_amt;
```

## ベストプラクティス

### 精確な重複排除

以下の例は、クリック日 `click_time`、広告主 `advertiser`、クリックチャネル `channel`、およびクリックユーザー ID `user_id` を記録した広告ビジネス関連の詳細テーブル `advertiser_view_record` に基づいています。

```SQL
CREATE TABLE advertiser_view_record(
    click_time DATE,
    advertiser VARCHAR(10),
    channel VARCHAR(10),
    user_id INT
) distributed BY hash(click_time);
```

このシナリオでは、以下のステートメントを頻繁に使用して、広告のクリック UV をクエリする必要があります。

```SQL
SELECT advertiser, channel, count(distinct user_id)
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

精確な重複排除クエリを高速化するためには、上記の詳細テーブルに基づいて同期物化ビューを作成し、bitmap_union() 関数を使用して事前にデータを集約できます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv AS
SELECT advertiser, channel, bitmap_union(to_bitmap(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

同期物化ビューの作成が完了すると、後続のクエリステートメントのサブクエリ `count(distinct user_id)` は自動的に `bitmap_union_count(to_bitmap(user_id))` に書き換えられ、クエリが物化ビューをヒットするようになります。

### 近似重複排除

前述の `advertiser_view_record` テーブルを例にとると、広告のクリック UV をクエリする際に近似重複排除クエリを高速化したい場合は、詳細テーブルに基づいて同期物化ビューを作成し、[hll_union()](../sql-reference/sql-functions/aggregate-functions/hll_union.md) 関数を使用して事前にデータを集約できます。

```SQL
CREATE MATERIALIZED VIEW advertiser_uv2 AS
SELECT advertiser, channel, hll_union(hll_hash(user_id))
FROM advertiser_view_record
GROUP BY advertiser, channel;
```

### プレフィックスインデックスの追加

基本テーブル `tableA` が `k1`、`k2`、`k3` の列を含んでいると仮定し、`k1` と `k2` のみがソートキーである場合、クエリステートメントにサブクエリ `where k3=x` を含めてプレフィックスインデックスを使用してクエリを高速化する必要があるビジネスシナリオでは、`k3` を最初の列とする同期物化ビューを作成できます。

```SQL
CREATE MATERIALIZED VIEW k3_as_key AS
SELECT k3, k2, k1
FROM tableA
```

## 集約関数のマッチング関係


同期マテリアライズドビューをクエリする際、元のクエリ文は自動的に書き換えられ、同期マテリアライズドビューに保存されている中間結果をクエリするために使用されます。以下の表は、元のクエリの集約関数と同期マテリアライズドビューを構築するために使用される集約関数の対応関係を示しています。ビジネスシナリオに応じて、対応する集約関数を選択して同期マテリアライズドビューを構築できます。

| **原始查询聚合函数**                                   | **物化视图构建聚合函数** |
| ------------------------------------------------------ | ------------------------ |
| sum                                                    | sum                      |
| min                                                    | min                      |
| max                                                    | max                      |
| count                                                  | count                    |
| bitmap_union, bitmap_union_count, count(distinct)      | bitmap_union             |
| hll_raw_agg, hll_union_agg, ndv, approx_count_distinct | hll_union                |
