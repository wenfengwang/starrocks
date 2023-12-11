```yaml
    displayed_sidebar: "English"
```

# 非同期マテリアライズドビューのトラブルシューティング

このトピックでは、非同期マテリアライズドビューを調査し、それらを使用する際に遭遇した問題を解決する方法について説明します。

> **注意**
>
> 以下に示す機能の一部は、StarRocks v3.1以降でのみサポートされています。

## 非同期マテリアライズドビューの調査

非同期マテリアライズドビューの全体像を把握するために、動作状態、リフレッシュ履歴、リソース消費などを最初に確認することができます。

### 非同期マテリアライズドビューの動作状態を確認する

[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して、非同期マテリアライズドビューの動作状態を確認できます。返される情報のうち、以下のフィールドに注目できます。

- `is_active`：マテリアライズドビューの状態がアクティブかどうか。アクティブなマテリアライズドビューだけがクエリの高速化および書き換えに使用できます。
- `last_refresh_state`：直近のリフレッシュの状態。PENDING、RUNNING、FAILED、SUCCESSのいずれかです。
- `last_refresh_error_message`：最後のリフレッシュが失敗した理由（マテリアライズドビューの状態がアクティブでない場合）。
- `rows`：マテリアライズドビュー内のデータ行数。この値は、更新が延期されることがあるため、実際の行数と異なることに注意してください。

返される他のフィールドの詳細については、[SHOW MATERIALIZED VIEWS - Returns](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md#returns)を参照してください。

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

情報スキーマデータベースの`task_runs`テーブルをクエリし、非同期マテリアライズドビューのリフレッシュ履歴を表示できます。返される情報のうち、以下のフィールドに注目できます。

- `CREATE_TIME` および `FINISH_TIME`：リフレッシュタスクの開始時刻および終了時刻。
- `STATE`：リフレッシュタスクの状態。PENDING、RUNNING、FAILED、SUCCESSのいずれかです。
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

### 非同期マテリアライズドビューのリソース消費を監視する

非同期マテリアライズドビューのリフレッシュ中およびリフレッシュ後に消費されるリソースを監視し、分析することができます。

#### リフレッシュ中のリソース消費を監視する

リフレッシュタスク中、[SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md)を使用して、リアルタイムのリソース消費を監視できます。

返される情報のうち、以下のフィールドに注目できます。

- `ScanBytes`：スキャンされたデータのサイズ。
- `ScanRows`：スキャンされたデータ行数。
- `MemoryUsage`：使用されているメモリサイズ。
- `CPUTime`：CPU時間の費用。
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

リフレッシュタスク後、クエリプロファイルを使用して、リフレッシュタスクによって消費された時間とリソースを分析できます。

非同期マテリアライズドビューが自動的にリフレッシュされる間、INSERT OVERWRITEステートメントが実行されます。対応するクエリプロファイルをチェックして、リフレッシュタスクによる時間およびリソースの消費を分析できます。

返される情報のうち、以下のメトリクスに注目できます。

- `Total`：クエリによって消費された総時間。
- `QueryCpuCost`：クエリの総CPU時間。CPU時間の費用は並行プロセスに対して集計されます。そのため、このメトリックの値はクエリの実行時間よりも大きくなる可能性があります。
- `QueryMemCost`：クエリの総メモリコスト。
- その他、個々の演算子（結合演算子や集約演算子など）のためのメトリクス。

クエリプロファイルのチェック方法および他のメトリクスの理解についての詳細については、[クエリプロファイルの分析](../administration/query_profile.md)を参照してください。

### 非同期マテリアライズドビューによってクエリが書き換えられたかどうかを確認する

[EXPLAIN](../sql-reference/sql-statements/Administration/EXPLAIN.md)を使用して、クエリプランから非同期マテリアライズドビューによってクエリが書き換えられたかどうかを確認できます。

クエリプランの`SCAN`メトリクスに対応する非同期マテリアライズドビューの名前が表示されている場合、クエリがマテリアライズドビューによって書き換えられています。

例 1：

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
```
```Plain
MySQL > SET enable_materialized_view_rewrite = false;
MySQL > EXPLAIN LOGICAL SELECT `customer`.`c_custkey`
                      -> FROM `ssb_1g`.`customer`
                      -> GROUP BY `customer`.`c_custkey`;
+-----------------------------------------------------------------------------------+
| 説明文字                                                                      |
+-----------------------------------------------------------------------------------+
| - 出力 => [1:c_custkey]                                                   |
|     - AGGREGATE(GLOBQL) [1:c_custkey]                                |
|             推定: {row: 15000, cpu: ?, memory: ?, network: ?, cost: 120000.0}    |
|         - SCAN [mv_bitmap] => [1:c_custkey]                             |
|                 推定: {row: 60000, cpu: ?, memory: ?, network: ?, cost: 30000.0} |
|                 partitionRatio: 1/1, tabletRatio: 12/12          |
+-----------------------------------------------------------------------------------+
```

## 問題の診断と解決

ここでは、非同期マテリアライズドビューを使用する際に遭遇するかもしれない一般的な問題と対応策をリストアップします。

### 非同期マテリアライズドビューの作成に失敗する

非同期マテリアライズドビューを作成できない場合、つまりCREATE MATERIALIZED VIEWステートメントを実行できない場合、次の側面を調べることができます。

- **同期マテリアライズドビューのSQLステートメントを誤って使用していないかどうかを確認してください。**

  StarRocksは、同期マテリアライズドビューと非同期マテリアライズドビューの2つの異なるマテリアライズドビューを提供しています。

  同期マテリアライズドビューを作成するために使用される基本的なSQLステートメントは次のとおりです。

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  AS <query>
  ```

  ただし、非同期マテリアライズドビューを作成するために使用されるSQLステートメントには、より多くのパラメータが含まれています。

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  REFRESH ASYNC -- 非同期マテリアライズドビューの更新戦略。
  DISTRIBUTED BY HASH(<column>) -- 非同期マテリアライズドビューのデータ分配戦略。
  AS <query>
  ```

  SQLステートメントに加えて、2つのマテリアライズドビューの主な違いは、非同期マテリアライズドビューがStarRocksが提供するすべてのクエリ構文をサポートするのに対し、同期マテリアライズドビューは集約関数の限られた選択肢のみをサポートする点です。

- **`Partition By`  カラムを正しく指定しているかどうかを確認してください。**

  非同期マテリアライズドビューを作成する際、更新戦略を指定でき、これによりマテリアライズドビューをより細かい粒度でリフレッシュすることが可能になります。

  現在、StarRocksは範囲パーティショニングのみをサポートしており、クエリステートメントで使用されるSELECT式から単一の列を参照することしかサポートしていません。partitioning strategyの粒度レベルを変更するにはdate_trunc() 関数を使用できます。他の式はサポートされていないことに留意してください。

- **マテリアライズドビューを作成するための必要な権限があるかどうかを確認してください。**

  非同期マテリアライズドビューを作成する際、クエリされるすべてのオブジェクト（テーブル、ビュー、マテリアライズドビュー）のSELECT権限が必要です。クエリでUDFが使用される場合は、その関数のUSAGE権限も必要です。

### マテリアライズドビューのリフレッシュに失敗する

マテリアライズドビューのリフレッシュに失敗した場合、つまりリフレッシュタスクの状態がSUCCESSではない場合、次の側面を調べることができます。

- **不適切なリフレッシュ戦略を採用していないかどうかを確認してください。**

  デフォルトでは、マテリアライズドビューは作成直後にリフレッシュされます。ただし、v2.5ないしそれ以前のバージョンでは、MANUALリフレッシュ戦略を採用したマテリアライズドビューは作成後にリフレッシュされません。マニュアルでREFRESH MATERIALIZED VIEWを使用してリフレッシュする必要があります。

- **リフレッシュタスクがメモリ制限を超えていないかどうかを確認してください。**

  通常、非同期マテリアライズドビューには大規模な集計や結合計算が関与する場合、そのリフレッシュタスクはメモリリソースを使い切ることがあります。この問題を解決するためには、次のことができます。

  - マテリアライズドビューにパーティショニング戦略を指定し、1回に1つのパーティションをリフレッシュする。
  - マテリアライズドビューのリフレッシュタスクにSpill to Disk機能を有効にします。v3.1以降、StarRocksはマテリアライズドビューのリフレッシュ時に中間結果をディスクにスピルすることをサポートしています。Spill to Diskを有効にするには、次のステートメントを実行してください。

    ```SQL
    SET enable_spill = true;
    ```

- **リフレッシュタスクがタイムアウト期間を超えていないかどうかを確認してください。**

  大規模なマテリアライズドビューはリフレッシュタスクがタイムアウト期間を超過することがあります。この問題を解決するためには、次のことができます。

  - マテリアライズドビューにパーティショニング戦略を指定し、1回に1つのパーティションをリフレッシュする。
  - より長いタイムアウト期間を設定する。

v3.0以降では、マテリアライズドビューを作成する際またはALTER MATERIALIZED VIEWを使用して次のプロパティ（セッション変数）を定義することができます。

例:

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

### マテリアライズドビューのステータスがアクティブでない

マテリアライズドビューがクエリの書き換えやリフレッシュに失敗し、マテリアライズドビューのステータスが`false`である場合は、基本テーブルのスキーマ変更の結果である可能性があります。この問題を解決するためには、次のステートメントを実行してマテリアライズドビューのステートを手動でアクティブに設定することができます。

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

マテリアライズドビューのステートをアクティブに設定しても効果がない場合は、マテリアライズドビューを削除して再作成する必要があります。

### マテリアライズドビューのリフレッシュタスクが過剰なリソースを使用している

リフレッシュタスクが過剰なシステムリソースを使用していることがわかった場合、次の側面を調べることができます。

- **過大なマテリアライズドビューを作成していないかどうかを確認してください。**

  計算量が著しい多数のテーブルを結合している場合、リフレッシュタスクは多くのリソースを占有するでしょう。この問題を解決するためには、マテリアライズドビューのサイズを評価し、再計画する必要があります。

- **不必要に頻繁なリフレッシュ間隔を設定していないかどうかを確認してください。**

  固定間隔のリフレッシュ戦略を採用した場合、この問題を解決するためには適切なリフレッシュ戦略を定義する必要があります。基本テーブルのデータ変更によってリフレッシュタスクがトリガされる場合、頻繁すぎるデータのローディングもこの問題の原因となります。

- **マテリアライズドビューがパーティション化されているかどうかを確認してください。**

  パーティション化されていないマテリアライズドビューはリフレッシュに多くのリソースを要する場合があります。この問題を解決するためには、マテリアライズドビューにパーティショニング戦略を指定し、1回に1つのパーティションをリフレッシュする必要があります。

リソースを過剰に使用しているリフレッシュタスクを停止するためには、次のように行うことができます。

- マテリアライズドビューのステートを非アクティブに設定し、そのリフレッシュタスクをすべて停止します。

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- SHOW PROCESSLISTおよびKILLを使用して、実行中のリフレッシュタスクを終了します。

  ```SQL
  -- 実行中のリフレッシュタスクのConnectionIdを取得します。
  SHOW PROCESSLIST;
  -- 実行中のリフレッシュタスクを終了します。
  KILL QUERY <ConnectionId>;
  ```

### マテリアライズドビューが関連するクエリを書き換えることに失敗する

マテリアライズドビューが関連するクエリを書き換えることに失敗した場合、次の側面を調べることができます。

- **マテリアライズドビューとクエリが一致しているかどうかを確認してください。**

  - StarRocksは、マテリアライズドビューとクエリをテキストベースではなく、構造ベースのマッチング手法で一致させます。そのため、似ているからといってクエリが書き換えられることが保証されているわけではありません。
  - マテリアライズドビューはSPJG（選択/プロジェクション/結合/集計）タイプのクエリのみを書き換えることができます。ウィンドウ関数や入れ子の集計、または結合と集計を含むクエリはサポートされていません。
  - 外部結合で複雑なJOIN述語を含むクエリをマテリアライズドビューは書き換えられません。たとえば、`A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id`のようなケースでは、`JOIN ON`句の述語を`WHERE`句に記述することを推奨します。

  マテリアライズドビューのクエリ書き換えの制限事項についての詳細情報は、[マテリアライズドビューを使用したクエリ書き換え - 制限事項](./query_rewrite_with_materialized_views.md#limitations)を参照してください。

- **マテリアライズドビューのステートがアクティブであるかどうかを確認してください。**

  StarRocksは、クエリ書き換えを行う前にマテリアライズドビューの状態を確認します。マテリアライズドビューの状態がアクティブでないときにのみ、クエリを書き換えることができます。この問題を解決するためには、次のステートメントを実行してマテリアライズドビューのステートを手動でアクティブに設定することができます。

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **マテリアライズドビューがデータの整合性要件を満たしているかを確認してください。**

  StarRocksは、マテリアライズドビューのデータと基本テーブルのデータの整合性を確認します。デフォルトでは、データが最新である場合にのみクエリを書き換えることができます。この問題を解決するためには、次のことができます。

- マテリアライズドビューに`PROPERTIES（'query_rewrite_consistency'='LOOSE'）`を追加して、整合性チェックを無効にします。
- ある程度のデータの不整合を許容するために、マテリアライズドビューに`PROPERTIES（'mv_rewrite_staleness_second'='5'）`を追加します。基底テーブルのデータが変更されても、最後の更新がこの時間間隔よりも前である場合、クエリは書き換えられる可能性があります。

- **マテリアライズドビューのクエリステートメントに出力列が不足しているかどうかを確認します。**

  範囲およびポイントクエリを書き換えるには、マテリアライズドビューのクエリステートメントのSELECT式で、フィルタリング述語として使用される列を指定する必要があります。クエリのWHEREおよびORDER BY句で参照される列がマテリアライズドビューのSELECTステートメントに含まれていることを確認する必要があります。

例1：マテリアライズドビュー`mv1`はネストされた集計を使用しているため、クエリの書き換えに使用することはできません。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

例2：マテリアライズドビュー`mv2`は結合に加えて集計を使用しているため、クエリの書き換えに使用することはできません。この問題を解決するためには、集計を使用したマテリアライズドビューを作成し、それを基に結合を使用したネストされたマテリアライズドビューを作成できます。

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

例3：マテリアライズドビュー`mv3`は、述語が参照する列がSELECT式に含まれていないため、`SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx'`のパターンでクエリを書き換えることができません。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

この問題を解決するためには、次のようにマテリアライズドビューを作成できます。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;
```