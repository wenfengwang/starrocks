---
displayed_sidebar: English
---

# 非同期マテリアライズドビューのトラブルシューティング

このトピックでは、非同期マテリアライズドビューを調査し、それらを使用中に遭遇した問題を解決する方法について説明します。

> **注意**
>
> 以下に示す機能の一部は、StarRocks v3.1以降でのみサポートされています。

## 非同期マテリアライズドビューを調査する

作業中の非同期マテリアライズドビューの全体像を把握するために、まずその動作状態、更新履歴、リソース消費をチェックします。

### 非同期マテリアライズドビューの動作状態をチェックする

非同期マテリアライズドビューの動作状態は、[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して確認できます。返される情報の中で、特に以下のフィールドに注目します：

- `is_active`: マテリアライズドビューがアクティブ状態かどうか。アクティブなマテリアライズドビューのみがクエリの高速化と書き換えに利用可能です。
- `last_refresh_state`: 最後の更新の状態で、PENDING、RUNNING、FAILED、SUCCESSを含みます。
- `last_refresh_error_message`: 最後の更新が失敗した理由（マテリアライズドビューの状態がアクティブでない場合）。
- `rows`: マテリアライズドビュー内のデータ行数。更新が遅延されることがあるため、この値はマテリアライズドビューの実際の行数と異なる可能性があることに注意してください。

返される他のフィールドの詳細情報については、[SHOW MATERIALIZED VIEWS - Returns](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md#returns)を参照してください。

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

### 非同期マテリアライズドビューの更新履歴を確認する

非同期マテリアライズドビューの更新履歴を確認するには、`information_schema`データベースの`task_runs`テーブルをクエリします。返される情報の中で、特に以下のフィールドに注目します：

- `CREATE_TIME` と `FINISH_TIME`: 更新タスクの開始と終了時刻。
- `STATE`: 更新タスクの状態で、PENDING、RUNNING、FAILED、SUCCESSを含みます。
- `ERROR_MESSAGE`: 更新タスクが失敗した理由。

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

### 非同期マテリアライズドビューのリソース消費をモニタリングする

更新中および更新後に非同期マテリアライズドビューが消費するリソースをモニタリングおよび分析することができます。

#### 更新中のリソース消費をモニタリングする

更新タスク中は、[SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md)を使用してリアルタイムのリソース消費をモニタリングできます。

返される情報の中で、特に以下のフィールドに注目します：

- `ScanBytes`: スキャンされたデータのサイズ。
- `ScanRows`: スキャンされたデータ行の数。
- `MemoryUsage`: 使用されたメモリのサイズ。
- `CPUTime`: CPUの使用時間。
- `ExecTime`: クエリの実行時間。

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

#### 更新後のリソース消費を分析する

更新タスク後は、クエリプロファイルを通じてリソース消費を分析できます。

非同期マテリアライズドビューが自己更新する際には、INSERT OVERWRITEステートメントが実行されます。対応するクエリプロファイルをチェックして、更新タスクによる時間とリソースの消費を分析できます。

返される情報の中で、特に以下のメトリクスに注目します：

- `Total`: クエリによる総消費時間。
- `QueryCpuCost`: クエリの総CPU時間コスト。CPU時間コストは並行プロセスに集約されるため、このメトリックの値はクエリの実際の実行時間を超えることがあります。
- `QueryMemCost`: クエリの総メモリコスト。
- 結合演算子や集約演算子など、個々のオペレーターのその他のメトリクス。

クエリプロファイルのチェック方法とその他のメトリクスの理解についての詳細は、[クエリプロファイルの分析](../administration/query_profile.md)を参照してください。

### クエリが非同期マテリアライズドビューによって書き換えられているかを検証する


クエリが[非同期マテリアライズドビュー](../sql-reference/sql-statements/Administration/EXPLAIN.md)で書き換えられるかどうかは、そのクエリプランを[EXPLAIN](../sql-reference/sql-statements/Administration/EXPLAIN.md)を使用してチェックできます。

クエリプラン内の`SCAN`メトリックが対応するマテリアライズドビューの名前を表示している場合、クエリはマテリアライズドビューによって書き換えられています。

例1:

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

クエリ書き換え機能を無効にすると、StarRocksは通常のクエリプランを採用します。

例2:

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
|         - SCAN [customer] => [1:c_custkey]                                            |
|                 Estimates: {row: 60000, cpu: ?, memory: ?, network: ?, cost: 30000.0} |
|                 partitionRatio: 1/1, tabletRatio: 12/12                               |
+---------------------------------------------------------------------------------------+
```

## 問題の診断と解決

ここでは、非同期マテリアライズドビューを使用している際に遭遇する可能性のある一般的な問題と、それに対する解決策をリストしています。

### 非同期マテリアライズドビューの作成に失敗

非同期マテリアライズドビューの作成に失敗した場合、つまりCREATE MATERIALIZED VIEWステートメントが実行できない場合、以下の点を確認できます。

- **同期マテリアライズドビュー用のSQLステートメントを誤って使用していないか確認してください。**
  
  StarRocksは同期マテリアライズドビューと非同期マテリアライズドビューの2種類を提供しています。

  同期マテリアライズドビューを作成するための基本的なSQLステートメントは以下の通りです。

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  AS <query>
  ```

  しかし、非同期マテリアライズドビューを作成するためのSQLステートメントにはより多くのパラメータが含まれています。

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  REFRESH ASYNC -- 非同期マテリアライズドビューの更新戦略。
  DISTRIBUTED BY HASH(<column>) -- 非同期マテリアライズドビューのデータ分散戦略。
  AS <query>
  ```

  SQLステートメントの違いに加えて、非同期マテリアライズドビューはStarRocksが提供するすべてのクエリ構文をサポートしているのに対し、同期マテリアライズドビューは限られた集約関数のみをサポートするという主な違いがあります。

- **正しい`Partition By`列を指定しているか確認してください。**

  非同期マテリアライズドビューを作成する際には、パーティショニング戦略を指定して、より細かい粒度でマテリアライズドビューを更新できます。

  現在、StarRocksは範囲パーティショニングのみをサポートしており、マテリアライズドビューを構築するためのクエリステートメントのSELECT式から単一の列のみを参照することをサポートしています。date_trunc()関数を使用して列を切り捨て、パーティショニング戦略の粒度レベルを変更することができます。他の式はサポートされていないことに注意してください。

- **マテリアライズドビューを作成するための必要な権限を持っているか確認してください。**

  非同期マテリアライズドビューを作成する際には、クエリされるすべてのオブジェクト（テーブル、ビュー、マテリアライズドビュー）に対するSELECT権限が必要です。クエリにUDFが使用されている場合は、関数のUSAGE権限も必要です。

### マテリアライズドビューの更新が失敗する

マテリアライズドビューの更新が失敗する、つまり更新タスクの状態がSUCCESSではない場合、以下の点を確認できます。

- **不適切な更新戦略を採用していないか確認してください。**

  デフォルトでは、マテリアライズドビューは作成直後に更新されます。しかし、v2.5およびそれ以前のバージョンでは、MANUAL更新戦略を採用したマテリアライズドビューは作成後に更新されません。REFRESH MATERIALIZED VIEWを使用して手動で更新する必要があります。

- **更新タスクがメモリ制限を超えていないか確認してください。**

  通常、非同期マテリアライズドビューは、大規模な集約や結合計算を含み、メモリリソースを使い果たすことがあります。この問題を解決するには、以下の方法があります。

  - マテリアライズドビューにパーティショニング戦略を指定し、一度に1つのパーティションを更新します。
  - 更新タスクにディスクへのスピル機能を有効にします。v3.1以降、StarRocksはマテリアライズドビューの更新時に中間結果をディスクにスピルすることをサポートしています。以下のステートメントを実行してディスクへのスピルを有効にします。

    ```SQL
    SET enable_spill = true;
    ```

- **更新タスクがタイムアウト期間を超えていないか確認してください。**

  大規模なマテリアライズドビューは、更新タスクがタイムアウト期間を超えるために更新に失敗することがあります。この問題を解決するには、以下の方法があります。

  - マテリアライズドビューにパーティショニング戦略を指定し、一度に1つのパーティションを更新します。
  - タイムアウト期間を長く設定します。

v3.0以降、マテリアライズドビューの作成時、またはALTER MATERIALIZED VIEWを使用して追加する際に、以下のプロパティ（セッション変数）を定義できます。

例：

```SQL
-- マテリアライズドビューを作成する際にプロパティを定義する
CREATE MATERIALIZED VIEW mv1 
REFRESH ASYNC
PROPERTIES ( 'session.enable_spill'='true' )
AS <query>;

-- プロパティを追加する。
ALTER MATERIALIZED VIEW mv2 
    SET ('session.enable_spill' = 'true');
```

### マテリアライズドビューの状態がアクティブではない


マテリアライズドビューがクエリの書き換えやリフレッシュができず、マテリアライズドビューの状態が `is_active` が `false` の場合、ベーステーブルのスキーマ変更が原因である可能性があります。この問題を解決するには、次のステートメントを実行して、マテリアライズドビューの状態をアクティブに設定します。

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

マテリアライズドビューの状態をアクティブに設定しても効果がない場合は、マテリアライズドビューを削除して再作成する必要があります。

### マテリアライズドビューのリフレッシュタスクが過剰なリソースを使用している

リフレッシュタスクが過剰なシステムリソースを使用していることがわかった場合、以下の点を確認できます。

- **過大なマテリアライズドビューを作成していないか確認します。**

  多くのテーブルを結合し、大量の計算を引き起こすと、リフレッシュタスクは多くのリソースを占有します。この問題を解決するには、マテリアライズドビューのサイズを評価し、再計画する必要があります。

- **不必要に頻繁なリフレッシュ間隔を設定していないか確認します。**

  固定間隔リフレッシュ戦略を採用している場合、リフレッシュ頻度を下げることで問題を解決できます。リフレッシュタスクがベーステーブルのデータ変更によってトリガーされる場合、データを頻繁にロードしすぎると問題が発生することがあります。この問題を解決するには、マテリアライズドビューに適切なリフレッシュ戦略を定義する必要があります。

- **マテリアライズドビューがパーティション化されているかどうかを確認します。**

  パーティション化されていないマテリアライズドビューは、StarRocksがマテリアライズドビュー全体を毎回リフレッシュするため、リフレッシュにコストがかかる可能性があります。この問題を解決するには、マテリアライズドビューにパーティション戦略を指定し、一度に一つのパーティションだけをリフレッシュするようにします。

リソースを大量に消費するリフレッシュタスクを停止するには：

- マテリアライズドビューの状態を非アクティブに設定し、すべてのリフレッシュタスクを停止します：

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- 実行中のリフレッシュタスクを終了するには、SHOW PROCESSLISTとKILLを使用します：

  ```SQL
  -- 実行中のリフレッシュタスクのConnectionIdを取得します。
  SHOW PROCESSLIST;
  -- 実行中のリフレッシュタスクを終了します。
  KILL QUERY <ConnectionId>;
  ```

### マテリアライズドビューがクエリの書き換えに失敗する

マテリアライズドビューが関連するクエリの書き換えに失敗する場合、以下の点を確認できます。

- **マテリアライズドビューとクエリが一致しているかを確認します。**

  - StarRocksは、テキストベースのマッチングではなく、構造ベースのマッチング技術を使用してマテリアライズドビューとクエリを照合します。そのため、マテリアライズドビューのクエリに似ているからといって、クエリが書き換えられるとは限りません。
  - マテリアライズドビューは、SPJG（Select/Projection/Join/Aggregation）タイプのクエリのみを書き換えることができます。ウィンドウ関数、ネストされた集約、または結合プラス集約を含むクエリはサポートされていません。
  - マテリアライズドビューは、外部結合で複雑な結合述語を含むクエリを書き換えることができません。例えば、`A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id`のような場合、`JOIN ON`句の述語を`WHERE`句に指定することをお勧めします。

  マテリアライズドビュークエリの書き換えの制限についての詳細は、[マテリアライズドビューを使用したクエリの書き換え - 制限事項](./query_rewrite_with_materialized_views.md#limitations)を参照してください。

- **マテリアライズドビューの状態がアクティブであるかを確認します。**

  StarRocksは、クエリを書き換える前にマテリアライズドビューの状態をチェックします。マテリアライズドビューの状態がアクティブである場合のみ、クエリを書き換えることができます。この問題を解決するには、次のステートメントを実行してマテリアライズドビューの状態をアクティブに設定します。

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **マテリアライズドビューがデータ整合性の要件を満たしているかを確認します。**

  StarRocksは、マテリアライズドビューとベーステーブルのデータの整合性をチェックします。デフォルトでは、マテリアライズドビューのデータが最新の場合にのみクエリを書き換えることができます。この問題を解決するには、次の方法があります。

  - マテリアライズドビューに`PROPERTIES('query_rewrite_consistency'='LOOSE')`を追加して整合性チェックを無効にします。
  - データの一定の不整合を許容するために`PROPERTIES('mv_rewrite_staleness_second'='5')`を追加します。最後のリフレッシュがこの時間間隔より前であれば、ベーステーブルのデータが変更されているかどうかにかかわらず、クエリを書き換えることができます。

- **マテリアライズドビューのクエリステートメントに出力列が欠けていないかを確認します。**

  範囲クエリやポイントクエリを書き換えるためには、マテリアライズドビューのクエリステートメントのSELECT式にフィルタリング述語として使用される列を指定する必要があります。マテリアライズドビューのSELECTステートメントを確認し、クエリのWHERE句やORDER BY句で参照される列が含まれていることを確認する必要があります。

例1: マテリアライズドビュー`mv1`はネストされた集約を使用しているため、クエリの書き換えには使用できません。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

例2: マテリアライズドビュー`mv2`は結合プラス集約を使用しているため、クエリの書き換えには使用できません。この問題を解決するには、集約を使用してマテリアライズドビューを作成し、その後に結合を使用してネストされたマテリアライズドビューを作成します。

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


例 3: マテリアライズドビュー `mv3` は、述部で参照された列が SELECT 表現に含まれていないため、`SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx'` のパターンのクエリを書き換えることができません。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

この問題を解決するためには、以下のようにマテリアライズドビューを作成できます。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;
```
