---
displayed_sidebar: "Japanese"
---

# 非同期マテリアライズドビューのトラブルシューティング

このトピックでは、非同期マテリアライズドビューの調査方法と、それらを使用する際に遭遇する問題の解決方法について説明します。

> **注意**
>
> 以下に示す一部の機能は、StarRocks v3.1以降でのみサポートされています。

## 非同期マテリアライズドビューの調査

作業中の非同期マテリアライズドビューの全体像を把握するために、まずその作業状態、リフレッシュ履歴、およびリソース消費状況を確認できます。

### 非同期マテリアライズドビューの作業状態を確認する

[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して、非同期マテリアライズドビューの作業状態を確認できます。返される情報の中で、次のフィールドに注目できます。

- `is_active`：マテリアライズドビューの状態がアクティブかどうか。アクティブなマテリアライズドビューのみがクエリの高速化と書き換えに使用できます。
- `last_refresh_state`：最後のリフレッシュの状態。PENDING、RUNNING、FAILED、SUCCESSなどが含まれます。
- `last_refresh_error_message`：最後のリフレッシュが失敗した理由（マテリアライズドビューの状態がアクティブでない場合）。
- `rows`：マテリアライズドビューのデータ行数。この値は、更新が遅延される可能性があるため、実際の行数と異なる場合があります。

返される他のフィールドの詳細な情報については、[SHOW MATERIALIZED VIEWS - Returns](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md#returns)を参照してください。

例：

```Plain
MySQL > SHOW MATERIALIZED VIEWS LIKE 'mv_pred_2'\G
***************************[ 1. row ]***************************
id                                   | 112517
database_name                        | ssb_1g
name                                 | mv_pred_2
refresh_type                         | ASYNC
is_active                            | true
inactive_reason                      | <null>
partition_type                       | UNPARTITIONED
task_id                              | 457930
task_name                            | mv-112517
last_refresh_start_time              | 2023-08-04 16:46:50
last_refresh_finished_time           | 2023-08-04 16:46:54
last_refresh_duration                | 3.996
last_refresh_state                   | SUCCESS
last_refresh_force_refresh           | false
last_refresh_start_partition         |
last_refresh_end_partition           |
last_refresh_base_refresh_partitions | {}
last_refresh_mv_refresh_partitions   |
last_refresh_error_code              | 0
last_refresh_error_message           |
rows                                 | 0
text                                 | CREATE MATERIALIZED VIEW `mv_pred_2` (`lo_quantity`, `lo_revenue`, `sum`)
DISTRIBUTED BY HASH(`lo_quantity`, `lo_revenue`) BUCKETS 2
REFRESH ASYNC
PROPERTIES (
"replication_num" = "1",
"storage_medium" = "HDD"
)
AS SELECT `lineorder`.`lo_quantity`, `lineorder`.`lo_revenue`, sum(`lineorder`.`lo_tax`) AS `sum`
FROM `ssb_1g`.`lineorder`
WHERE `lineorder`.`lo_linenumber` = 1
GROUP BY 1, 2;

1 row in set
Time: 0.003s
```

### 非同期マテリアライズドビューのリフレッシュ履歴を表示する

非同期マテリアライズドビューのリフレッシュ履歴は、データベース`information_schema`のテーブル`task_runs`をクエリすることで表示できます。返される情報の中で、次のフィールドに注目できます。

- `CREATE_TIME`および`FINISH_TIME`：リフレッシュタスクの開始時刻と終了時刻。
- `STATE`：リフレッシュタスクの状態。PENDING、RUNNING、FAILED、SUCCESSなどが含まれます。
- `ERROR_MESSAGE`：リフレッシュタスクが失敗した理由。

例：

```Plain
MySQL > SELECT * FROM information_schema.task_runs WHERE task_name ='mv-112517' \G
***************************[ 1. row ]***************************
QUERY_ID      | 7434cee5-32a3-11ee-b73a-8e20563011de
TASK_NAME     | mv-112517
CREATE_TIME   | 2023-08-04 16:46:50
FINISH_TIME   | 2023-08-04 16:46:54
STATE         | SUCCESS
DATABASE      | ssb_1g
EXPIRE_TIME   | 2023-08-05 16:46:50
ERROR_CODE    | 0
ERROR_MESSAGE | <null>
PROGRESS      | 100%
EXTRA_MESSAGE | {"forceRefresh":false,"mvPartitionsToRefresh":[],"refBasePartitionsToRefreshMap":{},"basePartitionsToRefreshMap":{}}
PROPERTIES    | {"FORCE":"false"}
***************************[ 2. row ]***************************
QUERY_ID      | 72dd2f16-32a3-11ee-b73a-8e20563011de
TASK_NAME     | mv-112517
CREATE_TIME   | 2023-08-04 16:46:48
FINISH_TIME   | 2023-08-04 16:46:53
STATE         | SUCCESS
DATABASE      | ssb_1g
EXPIRE_TIME   | 2023-08-05 16:46:48
ERROR_CODE    | 0
ERROR_MESSAGE | <null>
PROGRESS      | 100%
EXTRA_MESSAGE | {"forceRefresh":true,"mvPartitionsToRefresh":["mv_pred_2"],"refBasePartitionsToRefreshMap":{},"basePartitionsToRefreshMap":{"lineorder":["lineorder"]}}
PROPERTIES    | {"FORCE":"true"}
```

### 非同期マテリアライズドビューのリソース消費状況を監視する

リフレッシュ中およびリフレッシュ後の非同期マテリアライズドビューによって消費されるリソースを監視および分析できます。

#### リフレッシュ中のリソース消費を監視する

リフレッシュタスク中に、リアルタイムでリソース消費状況を[SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md)を使用して監視できます。

返される情報の中で、次のフィールドに注目できます。

- `ScanBytes`：スキャンされるデータのサイズ。
- `ScanRows`：スキャンされるデータ行の数。
- `MemoryUsage`：使用されるメモリのサイズ。
- `CPUTime`：CPU時間のコスト。
- `ExecTime`：クエリの実行時間。

例：

```Plain
MySQL > SHOW PROC '/current_queries'\G
***************************[ 1. row ]***************************
StartTime     | 2023-08-04 17:01:30
QueryId       | 806eed7d-32a5-11ee-b73a-8e20563011de
ConnectionId  | 0
Database      | ssb_1g
User          | root
ScanBytes     | 70.981 MB
ScanRows      | 6001215 rows
MemoryUsage   | 73.748 MB
DiskSpillSize | 0.000
CPUTime       | 2.515 s
ExecTime      | 2.583 s
```

#### リフレッシュ後のリソース消費を分析する

リフレッシュタスク後、クエリプロファイルを使用してリフレッシュタスクによって消費されるリソースを分析できます。

非同期マテリアライズドビューが自動的にリフレッシュされる間、INSERT OVERWRITEステートメントが実行されます。リフレッシュタスクによって消費される時間とリソースを分析するために、対応するクエリプロファイルを確認できます。

返される情報の中で、次のメトリックに注目できます。

- `Total`：クエリによって消費される合計時間。
- `QueryCpuCost`：クエリの合計CPU時間のコスト。CPU時間のコストは並行プロセスで集計されます。そのため、このメトリックの値はクエリの実行時間よりも大きい場合があります。
- `QueryMemCost`：クエリの合計メモリコスト。
- 結合演算子や集計演算子など、個々の演算子の他のメトリック。

クエリプロファイルを確認する方法や他のメトリックの理解についての詳細な情報については、[クエリプロファイルの分析](../administration/query_profile.md)を参照してください。

### 非同期マテリアライズドビューによってクエリが書き換えられたかどうかを検証する

[EXPLAIN](../sql-reference/sql-statements/Administration/EXPLAIN.md)を使用して、クエリプランの`SCAN`メトリックが対応するマテリアライズドビューの名前を示している場合、クエリがマテリアライズドビューによって書き換えられたことを確認できます。

例1：

```Plain
MySQL > SHOW CREATE TABLE mv_agg\G
***************************[ 1. row ]***************************
Materialized View        | mv_agg
Create Materialized View | CREATE MATERIALIZED VIEW `mv_agg` (`c_custkey`)
DISTRIBUTED BY RANDOM
REFRESH ASYNC
PROPERTIES (
"replication_num" = "1",
"replicated_storage" = "true",
"storage_medium" = "HDD"
)
AS SELECT `customer`.`c_custkey`
FROM `ssb_1g`.`customer`
GROUP BY `customer`.`c_custkey`;

MySQL > EXPLAIN LOGICAL SELECT `customer`.`c_custkey`
                      -> FROM `ssb_1g`.`customer`
                      -> GROUP BY `customer`.`c_custkey`;
+-----------------------------------------------------------------------------------+
| Explain String                                                                    |
+-----------------------------------------------------------------------------------+
| - Output => [1:c_custkey]                                                         |
|     - SCAN [mv_agg] => [1:c_custkey]                                              |
|             Estimates: {row: 30000, cpu: ?, memory: ?, network: ?, cost: 15000.0} |
|             partitionRatio: 1/1, tabletRatio: 12/12                               |
|             1:c_custkey := 10:c_custkey                                           |
+-----------------------------------------------------------------------------------+
```

クエリのプランにおいて`SCAN`メトリックが対応するマテリアライズドビューの名前を示している場合、クエリはマテリアライズドビューによって書き換えられています。

例2：

```Plain
MySQL > SET enable_materialized_view_rewrite = false;
MySQL > EXPLAIN LOGICAL SELECT `customer`.`c_custkey`
                      -> FROM `ssb_1g`.`customer`
                      -> GROUP BY `customer`.`c_custkey`;
+---------------------------------------------------------------------------------------+
| Explain String                                                                        |
+---------------------------------------------------------------------------------------+
| - Output => [1:c_custkey]                                                             |
|     - AGGREGATE(GLOBAL) [1:c_custkey]                                                 |
|             Estimates: {row: 15000, cpu: ?, memory: ?, network: ?, cost: 120000.0}    |
|         - SCAN [mv_bitmap] => [1:c_custkey]                                           |
|                 Estimates: {row: 60000, cpu: ?, memory: ?, network: ?, cost: 30000.0} |
|                 partitionRatio: 1/1, tabletRatio: 12/12                               |
+---------------------------------------------------------------------------------------+
```

## 問題の診断と解決

ここでは、非同期マテリアライズドビューを使用する際に遭遇する可能性のある一部の一般的な問題と、それに対応する解決策を示します。

### 非同期マテリアライズドビューの作成に失敗する

非同期マテリアライズドビューの作成に失敗した場合、つまり、CREATE MATERIALIZED VIEWステートメントを実行できない場合、次の側面を確認できます。

- **同期マテリアライズドビューのSQLステートメントを誤って使用していないかどうかを確認してください。**
  
  StarRocksでは、同期マテリアライズドビューと非同期マテリアライズドビューの2つの異なるマテリアライズドビューが提供されています。

  同期マテリアライズドビューを作成するための基本的なSQLステートメントは次のとおりです。

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  AS <query>
  ```

  ただし、非同期マテリアライズドビューを作成するためのSQLステートメントには、さらに多くのパラメータが含まれます。

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  REFRESH ASYNC -- 非同期マテリアライズドビューのリフレッシュ戦略。
  DISTRIBUTED BY HASH(<column>) -- 非同期マテリアライズドビューのデータ分散戦略。
  AS <query>
  ```

  SQLステートメント以外の、2つのマテリアライズドビューの主な違いは、非同期マテリアライズドビューがStarRocksが提供するすべてのクエリ構文をサポートしているのに対し、同期マテリアライズドビューは限られた集計関数の選択肢のみをサポートしていることです。

- **正しい`Partition By`列を指定しているかどうかを確認してください。**

  非同期マテリアライズドビューを作成する際に、リフレッシュの粒度を細かくするためのパーティショニング戦略を指定できます。

  現在、StarRocksは範囲パーティショニングのみをサポートしており、クエリステートメントで使用されるSELECT式の中の単一の列を参照することしかサポートしていません。パーティショニング戦略の粒度レベルを変更するためには、date_trunc()関数を使用して列を切り捨てることができます。ただし、他の式はサポートされていません。

- **マテリアライズドビューを作成するために必要な権限を持っているかどうかを確認してください。**

  非同期マテリアライズドビューを作成する際には、クエリされるすべてのオブジェクト（テーブル、ビュー、マテリアライズドビュー）のSELECT権限が必要です。クエリでUDFを使用する場合は、関数のUSAGE権限も必要です。

### マテリアライズドビューのリフレッシュに失敗する

マテリアライズドビューのリフレッシュに失敗した場合、つまり、リフレッシュタスクの状態がSUCCESSでない場合、次の側面を確認できます。

- **適切なリフレッシュ戦略を採用しているかどうかを確認してください。**

  デフォルトでは、マテリアライズドビューは作成後すぐにリフレッシュされます。ただし、v2.5およびそれ以前のバージョンでは、MANUALリフレッシュ戦略を採用したマテリアライズドビューは作成後にリフレッシュされません。REFRESH MATERIALIZED VIEWを使用して手動でリフレッシュする必要があります。

- **リフレッシュタスクがメモリ制限を超えていないかどうかを確認してください。**

  通常、非同期マテリアライズドビューには大規模な集計や結合計算が関わるため、メモリリソースを使い果たすことがあります。この問題を解決するためには、次のことができます。

  - マテリアライズドビューにパーティショニング戦略を指定し、1つのパーティションごとにリフレッシュする。
  - リフレッシュタスクに対してSpill to Disk機能を有効にします。StarRocksは、マテリアライズドビューをリフレッシュする際に、中間結果をディスクにスピルする機能をv3.1以降でサポートしています。次のステートメントを実行してSpill to Diskを有効にします。

    ```SQL
    SET enable_spill = true;
    ```

- **リフレッシュタスクがタイムアウト期間を超えていないかどうかを確認してください。**

  大規模なマテリアライズドビューのリフレッシュタスクは、リフレッシュタスクがタイムアウト期間を超えるために失敗することがあります。この問題を解決するためには、次のことができます。

  - マテリアライズドビューにパーティショニング戦略を指定し、1つのパーティションごとにリフレッシュする。
  - より長いタイムアウト期間を設定する。

v3.0以降、マテリアライズドビューを作成する際に次のプロパティ（セッション変数）を定義するか、ALTER MATERIALIZED VIEWを使用して追加することができます。

例：

```SQL
-- マテリアライズドビューを作成する際にプロパティを定義する
CREATE MATERIALIZED VIEW mv1 
REFRESH ASYNC
PROPERTIES ( 'session.enable_spill'='true' )
AS <query>;

-- プロパティを追加する
ALTER MATERIALIZED VIEW mv2 
    SET ('session.enable_spill' = 'true');
```

### マテリアライズドビューの状態がアクティブでない

マテリアライズドビューがクエリの書き換えやリフレッシュを行えず、マテリアライズドビューの状態`is_active`が`false`である場合、これはベーステーブルのスキーマ変更の結果かもしれません。この問題を解決するためには、次のステートメントを実行してマテリアライズドビューの状態を手動でアクティブに設定できます。

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

マテリアライズドビューの状態をアクティブに設定しても効果がない場合は、マテリアライズドビューを削除して再作成する必要があります。

### マテリアライズドビューのリフレッシュタスクが過剰なリソースを使用する

リフレッシュタスクが過剰なシステムリソースを使用している場合、次の側面を確認できます。

- **オーバーサイズのマテリアライズドビューを作成していないかどうかを確認してください。**

  大量の計算を引き起こすテーブルの結合が行われている場合、リフレッシュタスクは多くのリソースを占有します。この問題を解決するためには、マテリアライズドビューのサイズを評価し、再計画する必要があります。

- **不必要に頻繁なリフレッシュ間隔を設定していないかどうかを確認してください。**

  固定間隔のリフレッシュ戦略を採用している場合、問題を解決するためにリフレッシュ頻度を下げることができます。リフレッシュタスクがベーステーブルのデータの変更によってトリガされる場合、データを頻繁にロードすることもこの問題の原因となります。この問題を解決するためには、マテリアライズドビューに適切なリフレッシュ戦略を定義する必要があります。

- **マテリアライズドビューがパーティション化されているかどうかを確認してください。**

  パーティション化されていないマテリアライズドビューはリフレッシュにコストがかかる場合があります。StarRocksは、非同期マテリアライズドビューをリフレッシュする際には常にマテリアライズドビュー全体をリフレッシュします。この問題を解決するためには、マテリアライズドビューにパーティショニング戦略を指定して、1つのパーティションごとにリフレッシュする必要があります。

リソースを過剰に消費するリフレッシュタスクを停止するためには、次のことができます。

- マテリアライズドビューの状態を非アクティブに設定し、そのリフレッシュタスクをすべて停止します。

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- SHOW PROCESSLISTとKILLを使用して実行中のリフレッシュタスクを終了します。

  ```SQL
  -- 実行中のリフレッシュタスクのConnectionIdを取得します。
  SHOW PROCESSLIST;
  -- 実行中のリフレッシュタスクを終了します。
  KILL QUERY <ConnectionId>;
  ```

### マテリアライズドビューがクエリを書き換えられない

マテリアライズドビューが関連するクエリを書き換えられない場合、次の側面を確認できます。

- **マテリアライズドビューとクエリが一致しているかどうかを確認してください。**

  - StarRocksは、マテリアライズドビューとクエリをテキストベースの一致ではなく、構造ベースの一致技術を使用して一致させます。そのため、クエリがマテリアライズドビューによって書き換えられることは保証されません。
  - マテリアライズドビューは、SPJG（Select/Projection/Join/Aggregation）タイプのクエリのみを書き換えることができます。ウィンドウ関数、ネストされた集計、または結合と集計を含むクエリはサポートされていません。
  - マテリアライズドビューは、アウタージョインの複雑な結合述語を含むクエリを書き換えることはできません。たとえば、`A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id`のような場合、結合の`JOIN ON`句で述語を指定することをお勧めします。

  マテリアライズドビューのクエリ書き換えの制限についての詳細な情報については、[マテリアライズドビューによるクエリの書き換え - 制限事項](./query_rewrite_with_materialized_views.md#limitations)を参照してください。

- **マテリアライズドビューの状態がアクティブであるかどうかを確認してください。**

  StarRocksは、クエリを書き換える前にマテリアライズドビューの状態をチェックします。クエリは、マテリアライズドビューの状態がアクティブである場合にのみ書き換えることができます。この問題を解決するためには、次のステートメントを実行してマテリアライズドビューの状態をアクティブに設定できます。

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **マテリアライズドビューがデータの整合性要件を満たしているかどうかを確認してください。**

  StarRocksは、マテリアライズドビューのデータとベーステーブルのデータの整合性をチェックします。デフォルトでは、データが最新の場合にのみクエリを書き換えることができます。この問題を解決するためには、次のことができます。

  - マテリアライズドビューに`PROPERTIES('query_rewrite_consistency'='LOOSE')`を追加して整合性チェックを無効にする。
  - マテリアライズドビューに`PROPERTIES('mv_rewrite_staleness_second'='5')`を追加して、一定程度のデータの不整合を許容する。最後のリフレッシュがこの時間間隔よりも前の場合、ベーステーブルのデータの変更に関係なくクエリを書き換えることができます。

- **マテリアライズドビューのクエリステートメントに出力列が不足していないかどうかを確認してください。**

  範囲およびポイントクエリを書き換えるためには、マテリアライズドビューのクエリステートメントのSELECT式に、フィルタリング述語として使用される列を指定する必要があります。クエリのWHEREおよびORDER BY句で参照される列がマテリアライズドビューのSELECTステートメントに含まれていることを確認する必要があります。

例1：マテリアライズドビュー`mv1`はネストされた集計を使用しているため、クエリの書き換えには使用できません。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

例2：マテリアライズドビュー`mv2`は結合と集計を使用しているため、クエリの書き換えには使用できません。この問題を解決するためには、まず集計を含むマテリアライズドビューを作成し、それを基にした結合を含むネストされたマテリアライズドビューを作成することができます。

```SQL
CREATE MATERIALIZED VIEW mv2 REFRESH ASYNC AS
select *
from (
    select lo_orderkey, lo_custkey, p_partkey, p_name
    from lineorder
    join part on lo_partkey = p_partkey
) lo
join (
    select c_custkey
    from customer
    group by c_custkey
) cust
on lo.lo_custkey = cust.c_custkey;
```

例3：マテリアライズドビュー`mv3`は、`SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx'`のパターンのクエリを書き換えることができません。なぜなら、述語が参照する列がSELECT式に含まれていないためです。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

この問題を解決するためには、次のようにマテリアライズドビューを作成できます。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;
```
