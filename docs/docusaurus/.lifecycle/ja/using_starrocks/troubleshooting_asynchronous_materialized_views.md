---
displayed_sidebar: "Japanese"
---

# 非同期マテリアライズドビューのトラブルシューティング

このトピックでは、非同期マテリアライズドビューを調査し、それらを使用する際に遭遇した問題を解決する方法について説明します。

> **注意**
>
> 下記に示されている機能のうち、一部はStarRocks v3.1以降でのみサポートされています。

## 非同期マテリアライズドビューの調査

非同期マテリアライズドビューを完全に把握し、問題の解決に役立てるために、まずそれらの動作状態、リフレッシュ履歴、リソース消費をチェックできます。

### 非同期マテリアライズドビューの動作状態を確認する

[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用して非同期マテリアライズドビューの動作状態を確認できます。返される情報の中で特に注目すべきフィールドは次のとおりです。

- `is_active`：マテリアライズドビューの状態がアクティブかどうか。アクティブなマテリアライズドビューのみがクエリの高速化および書き換えに使用されます。
- `last_refresh_state`：最後のリフレッシュの状態（PENDING、RUNNING、FAILED、SUCCESSなど）。
- `last_refresh_error_message`：最後のリフレッシュが失敗した理由（マテリアライズドビューの状態がアクティブでない場合）。
- `rows`：マテリアライズドビュー内のデータ行数。この値は、更新が遅延される可能性があるため、マテリアライズドビューの実際の行数と異なることに注意してください。

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

### 非同期マテリアライズドビューのリフレッシュ履歴を表示する

非同期マテリアライズドビューのリフレッシュ履歴は、データベース`information_schema`のテーブル`task_runs`をクエリすることで表示できます。返される情報の中で特に注目すべきフィールドは次のとおりです。

- `CREATE_TIME`および`FINISH_TIME`：リフレッシュタスクの開始時刻と終了時刻。
- `STATE`：リフレッシュタスクの状態（PENDING、RUNNING、FAILED、SUCCESSなど）。
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

非同期マテリアライズドビューのリフレッシュ中およびその後に消費されるリソースを監視および分析できます。

#### リフレッシュ中のリソース消費を監視する

リフレッシュタスク中に、リアルタイムのリソース消費を[SHOW PROC '/current_queries'](../sql-reference/sql-statements/Administration/SHOW_PROC.md)を使用して監視できます。

返される情報の中で特に注目すべきフィールドは次のとおりです。

- `ScanBytes`：スキャンされたデータのサイズ。
- `ScanRows`：スキャンされたデータ行数。
- `MemoryUsage`：使用されるメモリのサイズ。
- `CPUTime`：CPU時間。
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

リフレッシュタスク後に、クエリプロファイルを使用してリソースの消費を分析できます。

非同期マテリアライズドビューが自動的にリフレッシュされる間に、INSERT OVERWRITEステートメントが実行されます。このリフレッシュタスクによって消費された時間とリソースを分析するために、対応するクエリプロファイルを確認できます。

返される情報の中で特に注目すべきメトリクスは次のとおりです。

- `Total`：クエリによって消費された合計時間。
- `QueryCpuCost`：クエリの合計CPU時間コスト。CPU時間コストは並行プロセスのために集約されます。そのため、このメトリクスの値はクエリの実行時間よりも大きくなる場合があります。
- `QueryMemCost`：クエリの合計メモリコスト。
- 結合演算子や集約演算子など、個々の演算子のその他のメトリクス。

クエリプロファイルの確認方法やその他のメトリクスの理解についての詳細は、[クエリプロファイルの分析](../administration/query_profile.md)を参照してください。

### 非同期マテリアライズドビューによってクエリが書き換えられたか確認する

[EXPLAIN](../sql-reference/sql-statements/Administration/EXPLAIN.md)を使用して、クエリプランから非同期マテリアライズドビューによってクエリが書き換えられたかどうかを確認できます。

クエリプランの`SCAN`が対応するマテリアライズドビューの名前を示している場合、クエリはマテリアライズドビューによって書き換えられています。

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
```
```markdown
+ 制限事項

ここでは、非同期マテリアライズド・ビューを使用する際に遭遇するかもしれない一般的な問題と、それに対応する解決策をリストアップしています。

### 非同期マテリアライズド・ビューの作成に失敗した場合

非同期マテリアライズド・ビューの作成に失敗した場合、つまり、CREATE MATERIALIZED VIEW ステートメントを実行できなかった場合は、以下の点を確認できます。

- **同期マテリアライズド・ビュー用の SQL ステートメントを誤って使用していないかどうかを確認します。**

  StarRocksは、同期マテリアライズド・ビューと非同期マテリアライズド・ビューの2つの異なるマテリアライズド・ビューを提供しています。

  同期マテリアライズド・ビューを作成するための基本的な SQL ステートメントは以下の通りです:

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  AS <query>
  ```

  一方、非同期マテリアライズド・ビューを作成するための SQL ステートメントにはより多くのパラメータが含まれます:

  ```SQL
  CREATE MATERIALIZED VIEW <mv_name> 
  REFRESH ASYNC -- 非同期マテリアライズド・ビューのリフレッシュ戦略
  DISTRIBUTED BY HASH(<column>) -- 非同期マテリアライズド・ビューのデータ分散戦略
  AS <query>
  ```

  SQL ステートメントの他にも、同期マテリアライズド・ビューと非同期マテリアライズド・ビューの主な違いは、非同期マテリアライズド・ビューがStarRocksが提供するすべてのクエリ構文をサポートする一方、同期マテリアライズド・ビューは集計関数の限られた選択のみをサポートする点です。

- **正しい `Partition By` 列を指定しているかどうかを確認します。**

  非同期マテリアライズド・ビューを作成する際、リフレッシュ戦略を指定して、より細かい粒度でマテリアライズド・ビューをリフレッシュできるパーティション戦略を指定できます。

  現時点では、StarRocksでは範囲パーティショニングのみがサポートされており、マテリアライズド・ビューを構築するクエリ文でSELECT式からの単一列の参照のみをサポートしています。パーティショニング戦略の粒度レベルを変更するためには、date_trunc() 関数を使用して列を切り詰めることができます。その他の式はサポートされていないことに注意してください。

- **マテリアライズド・ビューを作成するために必要な権限を持っているかどうかを確認します。**

  非同期マテリアライズド・ビューを作成する際には、クエリされるすべてのオブジェクト（テーブル、ビュー、マテリアライズド・ビュー）のSELECT権限が必要です。クエリでUDFを使用する場合は、その関数のUSAGE権限も必要です。

### マテリアライズド・ビューのリフレッシュに失敗した場合

マテリアライズド・ビューのリフレッシュに失敗した場合、つまり、リフレッシュタスクのステータスがSUCCESSでない場合は、以下の点を確認できます。

- **適切でないリフレッシュ戦略を採用していないかどうかを確認します。**

  デフォルトでは、マテリアライズド・ビューは作成後すぐにリフレッシュされます。ただし、v2.5以前のバージョンでは、MANUALリフレッシュ戦略を採用したマテリアライズド・ビューは作成後にリフレッシュされません。その場合は、REFRESH MATERIALIZED VIEWを使用して手動でリフレッシュする必要があります。

- **リフレッシュタスクがメモリ制限を超えていないかどうかを確認します。**

  通常、非同期マテリアライズド・ビューにはメモリリソースを消費する大規模な集計や結合演算が含まれています。この問題を解決するために、以下を行うことができます:

  - リフレッシュタスクで一度に1つのパーティションをリフレッシュできるように、マテリアライズド・ビューにパーティション戦略を指定します。
  - マテリアライズド・ビューをリフレッシュする際、中間結果をディスクにスピルするSpill to Disk機能を有効にします。v3.1以降、次のステートメントを実行してSpill to Diskを有効にできます:

    ```SQL
    SET enable_spill = true;
    ```

  - マテリアライズド・ビューを作成する際やALTER MATERIALIZED VIEWを使用して、以下のプロパティ（セッション変数）を定義できます。

例:

```SQL
-- マテリアライズド・ビューを作成する際にプロパティを定義します
CREATE MATERIALIZED VIEW mv1 
REFRESH ASYNC
PROPERTIES ( 'session.enable_spill'='true' )
AS <query>;

-- プロパティを追加します。
ALTER MATERIALIZED VIEW mv2 
    SET ('session.enable_spill' = 'true');
```

### マテリアライズド・ビューのステートがアクティブでない場合

マテリアライズド・ビューがクエリをリライトしたりリフレッシュしたりできず、マテリアライズド・ビューのステートが `is_active` で `false` である場合は、ベーステーブルのスキーマ変更が原因である可能性があります。この問題を解決するためには、次のステートメントを実行してマテリアライズド・ビューのステートをアクティブに手動で設定できます:

```SQL
ALTER MATERIALIZED VIEW mv1 ACTIVE;
```

マテリアライズド・ビューのステートをアクティブに設定しても効果がない場合は、マテリアライズド・ビューを削除してから再作成する必要があります。

### マテリアライズド・ビューのリフレッシュタスクが過剰なリソースを使用している場合

リフレッシュタスクが過剰なシステムリソースを使用していることがわかった場合は、以下の点を確認できます。

- **マテリアライズド・ビューを過大に作成していないかどうかを確認します。**

  大量の計算を引き起こすテーブルの結合が行われている場合、リフレッシュタスクは多くのリソースを占有します。この問題を解決するためには、マテリアライズド・ビューのサイズを評価し、再計画する必要があります。

- **不必要に頻繁なリフレッシュ間隔を設定していないかどうかを確認します。**

  固定間隔のリフレッシュ戦略を採用している場合、問題を解決するためにはより低いリフレッシュ頻度を設定できます。リフレッシュタスクがベーステーブルのデータ変更によってトリガーされる場合は、データをあまり頻繁にロードするとこの問題が発生する可能性があります。この問題を解決するためには、マテリアライズド・ビューに適切なリフレッシュ戦略を定義する必要があります。

- **マテリアライズド・ビューにパーティションがあるかどうかを確認します。**

  パーティションがないマテリアライズド・ビューはリフレッシュにコストがかかる場合があります。この問題を解決するためには、マテリアライズド・ビューにパーティション戦略を指定して、1度に1つのパーティションをリフレッシュできるようにする必要があります。

リソースを多く使用するリフレッシュタスクを停止するためには、以下のことができます:

- マテリアライズド・ビューのステートを非アクティブに設定することで、すべてのリフレッシュタスクを停止します:

  ```SQL
  ALTER MATERIALIZED VIEW mv1 INACTIVE;
  ```

- SHOW PROCESSLIST を使用して実行中のリフレッシュタスクを取得し、KILL を使用してそれを終了します:

  ```SQL
  -- 実行中のリフレッシュタスクのConnectionIdを取得します。
  SHOW PROCESSLIST;
  -- 実行中のリフレッシュタスクを終了します。
  KILL QUERY <ConnectionId>;
  ```

### マテリアライズド・ビューがクエリをリライトできない場合

マテリアライズド・ビューが関連するクエリをリライトできない場合は、以下の点を確認できます。

- **マテリアライズド・ビューとクエリのマッチングを確認します。**

  - StarRocksは、マテリアライズド・ビューとクエリを、テキストベースのマッチングではなく、構造ベースのマッチング技術を用いてマッチングします。そのため、クエリが似ているからといって、必ずしもマテリアライズド・ビューがリライトできるとは限りません。
  - マテリアライズド・ビューは、SPJG（Select/Projection/Join/Aggregation）タイプのクエリのみをリライトできます。ウィンドウ関数やネストされた集計、または結合と集計を含むクエリはサポートされていません。
  - 外部結合で複雑な結合述語を含むクエリをリライトすることはできません。たとえば、`A LEFT JOIN B ON A.dt > '2023-01-01' AND A.id = B.id` のようなケースでは、`JOIN ON` 句の述語を `WHERE` 句で指定することを推奨します。

  マテリアライズド・ビューのクエリリライトの制限に関する詳細情報については、[Query rewrite with materialized views - Limitations](./query_rewrite_with_materialized_views.md#limitations) を参照してください。

- **マテリアライズド・ビューのステートがアクティブであるかどうかを確認します。**

  StarRocksは、クエリをリライトする前にマテリアライズド・ビューのステータスをチェックします。マテリアライズド・ビューのステートがアクティブである場合のみ、クエリはリライトできます。この問題を解決するためには、次のステートメントを実行してマテリアライズド・ビューのステートをアクティブに手動で設定できます:

  ```SQL
  ALTER MATERIALIZED VIEW mv1 ACTIVE;
  ```

- **マテリアライズド・ビューがデータの整合性要件を満たしているかどうかを確認します。**

  StarRocksは、マテリアライズド・ビュー内のデータとベーステーブルのデータの整合性を確認します。デフォルトでは、データがマテリアライズド・ビュー内で最新である場合のみクエリをリライトできます。この問題を解決するためには、以下を行うことができます:
```
- マテリアライズド ビューに`PROPERTIES('query_rewrite_consistency'='LOOSE')`を追加して、整合性チェックを無効にします。
- マテリアライズド ビューに`PROPERTIES('mv_rewrite_staleness_second'='5')`を追加して、ある程度のデータ整合性を許容します。クエリは、最後のリフレッシュがこの時間間隔より前である場合に書き直すことができ、ベーステーブルのデータが変更されているかどうかに関係なく、再構築されます。

- **マテリアライズド ビューのクエリ文に出力列が不足しているかどうかをチェックします。**

範囲クエリやポイントクエリを書き直すためには、マテリアライズド ビューのクエリ文のSELECT式でフィルタリング述語として使用される列を指定する必要があります。クエリのWHERE句およびORDER BY句で参照されている列がマテリアライズド ビューのSELECT文に含まれているかを確認する必要があります。

例1: マテリアライズド ビュー `mv1` は入れ子の集計を使用しているため、クエリを書き直すことはできません。

```SQL
CREATE MATERIALIZED VIEW mv1 REFRESH ASYNC AS
select count(distinct cnt) 
from (
    select c_city, count(*) cnt 
    from customer 
    group by c_city
) t;
```

例2: マテリアライズド ビュー `mv2` は結合と集計を使用しているため、クエリを書き直すことはできません。この問題を解決するためには、集計を使用するマテリアライズド ビューを作成し、それに基づいて前のものに結合を使用する入れ子のマテリアライズド ビューを作成することができます。

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

例3: マテリアライズド ビュー `mv3` は、`SELECT c_city, sum(tax) FROM tbl WHERE dt='2023-01-01' AND c_city = 'xxx'`のパターンのクエリを書き直すことができません。なぜなら述語が参照する列がSELECT式に含まれていないからです。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT c_city, sum(tax) FROM tbl GROUP BY c_city;
```

この問題を解決するためには、次のようにマテリアライズド ビューを作成することができます。

```SQL
CREATE MATERIALIZED VIEW mv3 REFRESH ASYNC AS
SELECT dt, c_city, sum(tax) FROM tbl GROUP BY dt, c_city;
```