---
displayed_sidebar: Chinese
---

# 非同期マテリアライズドビューのトラブルシューティング

この記事では、非同期マテリアライズドビューのチェック方法と、使用中に遭遇する問題の解決方法について説明します。

> **注意**
>
> 以下に示す機能の一部は、StarRocks v3.1以降のバージョンでのみサポートされています。

## 非同期マテリアライズドビューのチェック

使用中の非同期マテリアライズドビューを完全に理解するために、まずその動作状態、リフレッシュ履歴、リソース消費状況をチェックすることができます。

### 非同期マテリアライズドビューの動作状態のチェック

[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) コマンドを使用して、非同期マテリアライズドビューの動作状態をチェックできます。返されるすべての情報の中で、以下のフィールドに注目してください：

- `is_active`：マテリアライズドビューが Active 状態であるかどうか。Active 状態のマテリアライズドビューのみがクエリの加速と書き換えに使用できます。
- `last_refresh_state`：最後のリフレッシュ状態を含む PENDING（保留中）、RUNNING（実行中）、FAILED（失敗）および SUCCESS（成功）。
- `last_refresh_error_message`：最後のリフレッシュが失敗した理由（マテリアライズドビューの状態が Active でない場合）。
- `rows`：マテリアライズドビュー内のデータ行数。更新に遅延があるため、この値はマテリアライズドビューの実際の行数と異なる場合があることに注意してください。

返される他のフィールドの詳細については、[SHOW MATERIALIZED VIEWS - 戻り値](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md#戻り値)を参照してください。

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

### 非同期マテリアライズドビューのリフレッシュ履歴の表示

`information_schema` データベースの `task_runs` テーブルを照会して、非同期マテリアライズドビューのリフレッシュ履歴を表示できます。返されるすべての情報の中で、以下のフィールドに注目してください：

- `CREATE_TIME` と `FINISH_TIME`：リフレッシュタスクの開始と終了時間。
- `STATE`：リフレッシュタスクの状態を含む PENDING（保留中）、RUNNING（実行中）、FAILED（失敗）および SUCCESS（成功）。
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

### 非同期マテリアライズドビューのリソース消費状況の監視

リフレッシュタスクの実行中または完了後に、非同期マテリアライズドビューが消費するリソースを監視および分析することができます。

#### リフレッシュタスク実行中のリソース消費の監視

リフレッシュタスク実行中は、[SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md) を使用してリアルタイムでリソース消費状況を監視できます。

返されるすべての情報の中で、以下のフィールドに注目してください：

- `ScanBytes`：スキャンされたデータのサイズ。
- `ScanRows`：スキャンされたデータ行数。
- `MemoryUsage`：使用されたメモリのサイズ。
- `CPUTime`：CPUの時間コスト。
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

#### リフレッシュタスク完了後のリソース消費の分析

リフレッシュタスク完了後は、Query Profile を通じてリソース消費状況を分析できます。

非同期マテリアライズドビューがリフレッシュされている間、INSERT OVERWRITE ステートメントが実行されます。対応する Query Profile をチェックして、リフレッシュタスクが消費した時間とリソースを分析できます。

返されるすべての情報の中で、以下の指標に注目してください：

- `Total`：クエリが消費した総時間。
- `QueryCpuCost`：クエリの総 CPU 時間コスト。CPU 時間コストは並行プロセスに集約されるため、この指標の値はクエリの実際の実行時間よりも大きくなる可能性があります。
- `QueryMemCost`：クエリの総メモリコスト。
- 結合演算子や集約演算子など、各演算子に対する特定の指標。

Query Profile の分析方法と他の指標の理解についての詳細は、[Query Profile の分析を見る](../administration/query_profile.md)を参照してください。

### クエリが非同期マテリアライズドビューによって書き換えられたかどうかの検証

[EXPLAIN](../sql-reference/sql-statements/Administration/EXPLAIN.md) を使用してクエリプランを表示し、クエリが非同期マテリアライズドビューによって書き換えられるかどうかをチェックできます。

クエリプランに `SCAN` 指標が表示され、対応するマテリアライズドビューの名前が表示されている場合、そのクエリはマテリアライズドビューによって書き換えられています。

例１：

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

クエリ書き換え機能が無効にされている場合、StarRocks は通常のクエリプランを使用します。

例２：

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

## 故障の診断と解決

以下に、非同期マテリアライズドビューの使用中に遭遇する可能性のある一般的な問題とその解決策を示します。

### 非同期マテリアライズドビューの作成に失敗

CREATE MATERIALIZED VIEW ステートメントを実行できず、非同期マテリアライズドビューを作成できない場合、以下の点から問題を解決できます：


- **同期物化ビューの作成に誤りがないか確認してください。**

  StarRocksでは、同期物化ビューと非同期物化ビューの2種類の物化ビューが提供されています。

  同期物化ビューを作成する際に使用する基本的なSQL文は次のとおりです：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  AS <query>
  ```

  それに対して、非同期物化ビューを作成する際には、以下のようなSQL文が使用されます：

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  REFRESH ASYNC -- 非同期物化ビューのリフレッシュ戦略。
  DISTRIBUTED BY HASH(<column>) -- 非同期物化ビューのデータ分布戦略。
  AS <query>
  ```

  SQL文以外にも、これら2つの物化ビューの主な違いは、非同期物化ビューがStarRocksで提供されるすべてのクエリ構文をサポートしているのに対し、同期物化ビューは制限された集計関数のみをサポートしていることです。

- **正しい`Partition By`列が指定されているか確認してください。**

  非同期物化ビューを作成する際には、より細かいレベルで物化ビューをリフレッシュするために、パーティション戦略を指定することができます。

  現在、StarRocksは単一列のパーティション列のみをサポートし、Range Partitionのみをサポートしています。パーティション列では、date_trunc()関数を使用してパーティション戦略の粒度レベルを変更することができます。その他の式はサポートされていません。

- **物化ビューを作成するために必要な権限を持っているか確認してください。**

  非同期物化ビューを作成する際には、すべてのクエリオブジェクト（テーブル、ビュー、物化ビュー）のSELECT権限が必要です。クエリでUDFを使用する場合は、関数のUSAGE権限も必要です。

### 物化ビューリフレッシュの失敗

物化ビューリフレッシュが失敗した場合、つまりリフレッシュタスクのステータスがSUCCESSでない場合、以下の点を確認して解決できます：

- **適切なリフレッシュ戦略を使用しているか確認してください。**

  デフォルトでは、物化ビューは作成後すぐにリフレッシュされます。ただし、v2.5およびそれ以前のバージョンでは、MANUALリフレッシュ戦略を使用して作成された物化ビューは自動的にリフレッシュされません。リフレッシュを手動で行うには、REFRESH MATERIALIZED VIEWコマンドを使用する必要があります。

- **リフレッシュタスクがメモリ制限を超えていないか確認してください。**

  通常、非同期物化ビューには大規模な集計や結合計算が関与する場合、多くのメモリリソースを消費します。この問題を解決するためには、次のことができます：

  - 物化ビューにパーティション戦略を指定して、細かいリフレッシュを実現します。
  - リフレッシュタスクの一部の中間結果をディスクに保存する中間結果スピル機能を有効にします。v3.1以降、StarRocksは物化ビューリフレッシュタスクの一部の中間結果をディスクに保存する機能をサポートしています。次のステートメントを実行して、中間結果スピル機能を有効にします：

    ```SQL
    SET enable_spill = true;
    ```

- **リフレッシュタスクがタイムアウトしていないか確認してください。**

  大きな物化ビューは、リフレッシュタスクがタイムアウトするためにリフレッシュできない場合があります。この問題を解決するためには、次のことができます：

  - 物化ビューにパーティション戦略を指定して、細かいリフレッシュを実現します。
  - タイムアウト時間を増やします。

v3.0以降、物化ビューを作成する際に以下の属性（セッション変数）を定義するか、ALTER MATERIALIZED VIEWコマンドを使用して追加できます：

例：

```SQL
-- 物化ビュー作成時に属性を定義します。
CREATE MATERIALIZED VIEW mv1 
REFRESH ASYNC
PROPERTIES ( 'session.enable_spill'='true' )
AS <query>;

-- 属性を追加します。
ALTER MATERIALIZED VIEW mv2 
    SET ('session.enable_spill' = 'true');
```

### 物化ビューが利用できない

物化ビューがクエリの改変やリフレッシュができず、物化ビューの`is_active`ステータスが`false`の場合、基になるテーブルでスキーマの変更が発生している可能性があります。この問題を解決するには、次のようにして物化ビューのステータスを手動でActiveに設定できます：

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

設定が効果がない場合は、物化ビューを削除して再作成する必要があります。

### 物化ビューリフレッシュタスクが多くのリソースを占有している

リフレッシュタスクがシステムリソースを過剰に使用している場合、次の点に注意して解決できます：

- **作成した物化ビューが大きすぎないか確認してください。**

  複数のテーブルを結合するなど、多くの計算が行われると、リフレッシュタスクは多くのリソースを占有します。この問題を解決するには、物化ビューのサイズを評価し、再計画する必要があります。

- **リフレッシュ間隔が頻度が高すぎないか確認してください。**

  物化ビューが固定間隔でリフレッシュされる場合、リフレッシュ頻度を下げることで問題を解決できます。リフレッシュタスクが基になるテーブルのデータ変更によってトリガされる場合、頻繁なデータインポート操作もこの問題の原因になる可能性があります。この問題を解決するには、適切なリフレッシュ戦略を定義する必要があります。

- **物化ビューがパーティションされているか確認してください。**

  パーティションされていない物化ビューは、リフレッシュ時に多くのリソースを消費する可能性があります。なぜなら、StarRocksは常に物化ビュー全体をリフレッシュするからです。この問題を解決するには、物化ビューにパーティション戦略を指定して、細かいリフレッシュを実現する必要があります。

リソースを過剰に使用しているリフレッシュタスクを停止するには、次のことができます：

- 物化ビューのステータスをInactiveに設定して、すべてのリフレッシュタスクを停止します：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- SHOW PROCESSLISTとKILLステートメントを使用して実行中のリフレッシュタスクを終了します：

  ```SQL
  -- 実行中のリフレッシュタスクのConnectionIdを取得します。
  SHOW PROCESSLIST;
  -- 実行中のリフレッシュタスクを終了します。
  KILL QUERY <ConnectionId>;
  ```

### 物化ビューのクエリ改変ができない

物化ビューが関連するクエリを改変できない場合、次の点に注意して解決できます：

- **物化ビューとクエリが一致しているか確認してください。**

  - StarRocksは、物化ビューとクエリをテキストではなく構造ベースのマッチング技術を使用して一致させます。したがって、クエリと物化ビューが見た目上同じであるからといって、必ずしもクエリを改変できるわけではありません。
  - 物化ビューは、SPJG（Select/Projection/Join/Aggregation）タイプのクエリのみを改変できます。ウィンドウ関数、ネストされた集計、またはJoinと集計を組み合わせたクエリは改変できません。
  - 物化ビューは、複雑なJoin述語を含むOuter Joinを改変できません。例えば、`A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id`のような場合、Joinの述語を`WHERE`句で指定することをお勧めします。

  物化ビューのクエリ改変の制限についての詳細情報は、[物化ビュークエリ改変 - 制限](./query_rewrite_with_materialized_views.md#制限)を参照してください。

- **物化ビューのステータスがActiveであるか確認してください。**

  StarRocksは、クエリを改変する前に物化ビューのステータスをチェックします。物化ビューのステータスがActiveでない場合、クエリは改変できません。この問題を解決するには、次のようにして物化ビューのステータスを手動でActiveに設定できます：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **物化ビューがデータの整合性要件を満たしているか確認してください。**

  StarRocksは、物化ビューと基になるテーブルのデータの整合性をチェックします。デフォルトでは、物化ビューのデータが最新である場合にのみクエリを改変できます。この問題を解決するには、次のことができます：

  - 物化ビューに`PROPERTIES('query_rewrite_consistency'='LOOSE')`を追加して整合性チェックを無効にします。
  - 物化ビューに`PROPERTIES('mv_rewrite_staleness_second'='5')`を追加して、一定程度のデータの不整合を許容します。最後のリフレッシュがこの時間間隔以内であれば、基になるテーブルのデータが変更されているかどうかに関係なく、クエリを改変できます。

- **物化ビューのクエリ文に出力列が不足していないか確認してください。**

  範囲クエリやポイントクエリを改変するためには、物化ビューのクエリ文のSELECT式でフィルタリングに使用される述語を指定する必要があります。物化ビューのSELECT文には、クエリの`WHERE`句や`ORDER BY`句で参照される列が含まれていることを確認する必要があります。

例1：物化ビュー`mv1`はネストされた集計を使用しているため、クエリの改変はできません。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

例2：物化ビュー`mv2`はJoinと集計を組み合わせているため、クエリの改変はできません。この問題を解決するには、集計を含む物化ビューを作成し、その物化ビューを基にJoinを含むネストされた物化ビューを作成することができます。

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
例3：物化ビュー `mv3` は `SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx'` のクエリを改写することができません。なぜなら、WHERE句で参照されている列がSELECT式に含まれていないからです。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

この問題を解決するためには、以下のように物化ビューを作成します：

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;
```

