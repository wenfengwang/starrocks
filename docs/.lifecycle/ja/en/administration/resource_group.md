---
displayed_sidebar: English
---

# リソースグループ

このトピックでは、StarRocksのリソースグループ機能について説明します。

![リソースグループ](../assets/resource_group.png)

この機能を使用すると、短いクエリ、アドホッククエリ、ETLジョブなど、複数のワークロードを1つのクラスターで同時に実行して、複数のクラスターをデプロイする追加コストを節約できます。技術的な観点から見ると、実行エンジンはユーザーの指定に従って同時実行ワークロードをスケジュールし、それらの間の干渉を分離します。

リソースグループのロードマップ:

- v2.2以降、StarRocksはクエリのリソース消費を制限し、同じクラスタ内のテナント間でリソースの分離と効率的な使用を実装する機能をサポートしています。
- StarRocks v2.3では、大規模なクエリのリソース消費をさらに制限し、クラスターリソースが過大なクエリリクエストによって枯渇するのを防ぎ、システムの安定性を保証することができます。
- StarRocks v2.5では、データロード(INSERT)の計算リソース消費を制限する機能をサポートしています。

|  | 内部テーブル | 外部テーブル | 大規模クエリ制限 | 短いクエリ | INSERT INTO、Broker Load  | Routine Load、Stream Load、スキーマ変更 |
|---|---|---|---|---|---|---|
| 2.2 | √ | × | × | × | × | × |
| 2.3 | √ | √ | √ | √ | × | × |
| 2.4 | √ | √ | √ | √ | × | × |
| 2.5以降 | √ | √ | √ | √ | √ | × |

## 用語

このセクションでは、リソースグループ機能を使用する前に理解しておくべき用語について説明します。

### リソースグループ

各リソースグループは、特定のBEからのコンピューティングリソースのセットです。クラスターの各BEを複数のリソースグループに分割することができます。クエリがリソースグループに割り当てられると、StarRocksはリソースグループに指定されたリソースクォータに基づいてCPUとメモリリソースを割り当てます。

BE上のリソースグループのCPUとメモリリソースクォータは、以下のパラメータを使用して指定できます：

- `cpu_core_limit`

  このパラメータは、BE上のリソースグループに割り当てることができるCPUコア数のソフトリミットを指定します。有効な値：0以外の正の整数。範囲：(1, `avg_be_cpu_cores`]、ここで`avg_be_cpu_cores`はすべてのBEのCPUコアの平均数を表します。

  実際のビジネスシナリオでは、リソースグループに割り当てられるCPUコアは、BE上のCPUコアの可用性に基づいて比例的にスケーリングされます。

  > **注記**
  >
  > 例えば、16個のCPUコアを提供するBEに3つのリソースグループ（rg1、rg2、rg3）を設定するとします。3つのリソースグループの`cpu_core_limit`の値はそれぞれ`2`、`6`、`8`です。
  >
  > BEのすべてのCPUコアが占有されている場合、以下の計算に基づいて、3つのリソースグループに割り当てられるCPUコアの数はそれぞれ2、6、8です。
  >
  > - rg1のCPUコア数 = BEのCPUコア総数 × (2/16) = 2
  > - rg2のCPUコア数 = BEのCPUコア総数 × (6/16) = 6
  > - rg3のCPUコア数 = BEのCPUコア総数 × (8/16) = 8
  >
  > BEのすべてのCPUコアが占有されていない場合、例えばrg1とrg2がロードされているがrg3はロードされていない場合、以下の計算に基づいてrg1とrg2に割り当てられるCPUコアの数はそれぞれ4と12です。
  >
  > - rg1のCPUコア数 = BEのCPUコア総数 × (2/8) = 4
  > - rg2のCPUコア数 = BEのCPUコア総数 × (6/8) = 12

- `mem_limit`

  このパラメータは、BEが提供する総メモリのうち、クエリに使用できるメモリの割合を指定します。有効な値：(0, 1)。

  > **注記**
  >
  > クエリに使用できるメモリ量は、`query_pool`パラメータによって示されます。このパラメータの詳細については、[メモリ管理](Memory_management.md)を参照してください。

- `concurrency_limit`

  このパラメータは、リソースグループ内の同時クエリの上限を指定します。これは、同時に多数のクエリが発生することによるシステムの過負荷を避けるために使用されます。このパラメータは、0より大きい値に設定された場合のみ有効です。デフォルト：0。

- `max_cpu_cores`

  単一のBEノード上でこのリソースグループのCPUコアの上限です。これは、0より大きい値に設定された場合のみ有効です。範囲：[0, `avg_be_cpu_cores`]、ここで`avg_be_cpu_cores`はすべてのBEノードのCPUコアの平均数を表します。デフォルト：0。

上記のリソース消費制限に基づいて、次のパラメータを使用して大規模なクエリのリソース消費をさらに制限することができます：

- `big_query_cpu_second_limit`: このパラメータは、単一のBE上での大規模なクエリのCPU使用時間の上限を指定します。同時クエリは時間を合計します。単位は秒です。このパラメータは、0より大きい値に設定された場合のみ有効です。デフォルト：0。
- `big_query_scan_rows_limit`: このパラメータは、単一のBE上での大規模なクエリのスキャン行数の上限を指定します。このパラメータは、0より大きい値に設定された場合のみ有効です。デフォルト：0。
- `big_query_mem_limit`: このパラメータは、単一のBE上での大規模なクエリのメモリ使用量の上限を指定します。単位はバイトです。このパラメータは、0より大きい値に設定された場合のみ有効です。デフォルト：0。

> **注記**
>
> リソースグループで実行されているクエリが上記の大規模クエリ制限を超えると、クエリはエラーで終了されます。FEノードの**fe.audit.log**の`ErrorCode`列でエラーメッセージを確認することもできます。

リソースグループの`type`は`short_query`または`normal`に設定できます。

- デフォルト値は`normal`です。`type`パラメータで`normal`を指定する必要はありません。
- クエリが`short_query`リソースグループにヒットした場合、BEノードは`short_query.cpu_core_limit`で指定されたCPUリソースを予約します。`normal`リソースグループにヒットしたクエリ用に予約されたCPUリソースは、`BE core - short_query.cpu_core_limit`に制限されます。
- `short_query`リソースグループにクエリがヒットしない場合、`normal`リソースグループのリソースに制限はありません。

> **注意**
>
> - StarRocksクラスターでは、最大で1つのショートクエリリソースグループを作成できます。
> - StarRocksは`short_query`リソースグループのCPUリソースにハード上限を設定していません。

### 分類子

各分類子は、クエリのプロパティに一致することができる1つ以上の条件を保持します。StarRocksは、一致条件に基づいて各クエリに最も適合する分類子を識別し、クエリの実行に必要なリソースを割り当てます。

分類子は、次の条件をサポートします：

- `user`: ユーザー名。
- `role`: ユーザーの役割。

- `query_type`: クエリのタイプです。`SELECT`と`INSERT`（v2.5以降）がサポートされています。`INSERT INTO`または`BROKER LOAD`タスクが`query_type`が`insert`であるリソースグループにヒットした場合、BEノードは指定されたCPUリソースをタスクに予約します。
- `source_ip`: クエリが開始されるCIDRブロックです。
- `db`: クエリがアクセスするデータベースです。カンマ`,`で区切られた文字列で指定できます。
- `plan_cpu_cost_range`: クエリの推定CPUコスト範囲です。形式は`(DOUBLE, DOUBLE]`です。デフォルト値はNULLで、制限がないことを示します。`fe.audit.log`の`PlanCpuCost`列は、クエリのCPUコストに対するシステムの推定値を表します。このパラメータはv3.1.4以降でサポートされています。
- `plan_mem_cost_range`: クエリのシステム推定メモリコスト範囲です。形式は`(DOUBLE, DOUBLE]`です。デフォルト値はNULLで、制限がないことを示します。`fe.audit.log`の`PlanMemCost`列は、クエリのメモリコストに対するシステムの推定値を表します。このパラメータはv3.1.4以降でサポートされています。

分類器は、クエリに関する情報と一致する分類器の1つまたはすべての条件が一致する場合にのみ、クエリと一致します。複数の分類器がクエリに一致する場合、StarRocksはクエリと各分類器の一致度を計算し、最も一致度が高い分類器を特定します。

> **注記**
>
> クエリが属するリソースグループは、FEノードの**fe.audit.log**の`ResourceGroup`列で確認できます。または、[クエリのリソースグループを表示する](#view-the-resource-group-of-a-query)に記載されているように`EXPLAIN VERBOSE <query>`を実行して確認できます。

StarRocksは、以下のルールを使用してクエリと分類器の一致度を計算します：

- 分類器がクエリと同じ`user`の値を持つ場合、分類器の一致度は1増加します。
- 分類器がクエリと同じ`role`の値を持つ場合、分類器の一致度は1増加します。
- 分類器がクエリと同じ`query_type`の値を持つ場合、分類器の一致度は1に加えて、以下の計算から得られる数値を加えたものになります：1 / 分類器内の`query_type`フィールドの数。
- 分類器がクエリと同じ`source_ip`の値を持つ場合、分類器の一致度は1に加えて、以下の計算から得られる数値を加えたものになります：(32 - `cidr_prefix`) / 64。
- 分類器がクエリと同じ`db`の値を持つ場合、分類器の一致度は10増加します。
- クエリのCPUコストが`plan_cpu_cost_range`の範囲内にある場合、分類器の一致度は1増加します。
- クエリのメモリコストが`plan_mem_cost_range`の範囲内にある場合、分類器の一致度は1増加します。

複数の分類器がクエリに一致する場合、条件の数が多い分類器の一致度が高くなります。

```Plain
-- 分類器Bは分類器Aよりも多くの条件を持っています。したがって、分類器Bは分類器Aよりも一致度が高くなります。


classifier A (user='Alice')


classifier B (user='Alice', source_ip = '192.168.1.0/24')
```

一致する分類器が同じ数の条件を持つ場合、条件がより正確に記述されている分類器の一致度が高くなります。

```Plain
-- 分類器Bで指定されているCIDRブロックは、分類器Aよりも範囲が狭いです。したがって、分類器Bは分類器Aよりも一致度が高くなります。
classifier A (user='Alice', source_ip = '192.168.1.0/16')
classifier B (user='Alice', source_ip = '192.168.1.0/24')

-- 分類器Cは分類器Dよりも指定されているクエリタイプが少ないです。したがって、分類器Cは分類器Dよりも一致度が高くなります。
classifier C (user='Alice', query_type in ('select'))
classifier D (user='Alice', query_type in ('insert', 'select'))
```

複数の分類器の一致度が同じ場合、分類器の1つがランダムに選択されます。

```Plain
-- クエリが同時にdb1とdb2を問い合わせ、分類器EとFがヒットした分類器の中で最も一致度が高い場合、EとFのどちらかがランダムに選択されます。
classifier E (db='db1')
classifier F (db='db2')
```

## コンピューティングリソースの分離

リソースグループと分類器を設定することで、クエリ間でコンピューティングリソースを分離できます。

### リソースグループの有効化

リソースグループを使用するには、StarRocksクラスターでPipeline Engineを有効にする必要があります。

```SQL
-- 現在のセッションでPipeline Engineを有効にします。
SET enable_pipeline_engine = true;
-- グローバルにPipeline Engineを有効にします。
SET GLOBAL enable_pipeline_engine = true;
```

ローディングタスクについては、FE設定項目`enable_pipeline_load`を設定して、ローディングタスク用のPipeline Engineを有効にする必要があります。この項目はv2.5.0以降でサポートされています。

```sql
ADMIN SET FRONTEND CONFIG ("enable_pipeline_load" = "true");
```

> **注記**
>
> v3.1.0以降、リソースグループはデフォルトで有効になり、セッション変数`enable_resource_group`は廃止されました。

### リソースグループと分類器の作成

以下のステートメントを実行して、リソースグループを作成し、分類器を関連付けて、リソースグループにコンピューティングリソースを割り当てます。

```SQL
CREATE RESOURCE GROUP <group_name> 
TO (
    user='string', 
    role='string', 
    query_type in ('select'), 
    source_ip='cidr'
) -- 分類器を作成します。複数の分類器を作成する場合は、カンマ(`,`)で分類器を区切ります。
WITH (
    "cpu_core_limit" = "INT",
    "mem_limit" = "m%",
    "concurrency_limit" = "INT",
    "type" = "str" -- リソースグループのタイプです。値をnormalに設定します。
);
```

例：

```SQL
CREATE RESOURCE GROUP rg1
TO 
    (user='rg1_user1', role='rg1_role1', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user2', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user3', source_ip='192.168.x.x/24'),
    (user='rg1_user4'),
    (db='db1')
WITH (
    'cpu_core_limit' = '10',
    'mem_limit' = '20%',
    'big_query_cpu_second_limit' = '100',
    'big_query_scan_rows_limit' = '100000',
    'big_query_mem_limit' = '1073741824'
);
```

### リソースグループの指定（オプション）

現在のセッションに直接リソースグループを指定できます。

```SQL
SET resource_group = 'group_name';
```

### リソースグループと分類器の表示

以下のステートメントを実行して、すべてのリソースグループと分類器を照会します。

```SQL
SHOW RESOURCE GROUPS ALL;
```

以下のステートメントを実行して、ログインユーザーのリソースグループと分類器を照会します。

```SQL
SHOW RESOURCE GROUPS;
```

以下のステートメントを実行して、特定のリソースグループとその分類器を照会します。

```SQL
SHOW RESOURCE GROUP group_name;
```

例：

```plain
mysql> SHOW RESOURCE GROUPS ALL;
+------+--------+--------------+----------+------------------+--------+------------------------------------------------------------------------------------------------------------------------+
| Name | Id     | CPUCoreLimit | MemLimit | ConcurrencyLimit | Type   | Classifiers                                                                                                            |
+------+--------+--------------+----------+------------------+--------+------------------------------------------------------------------------------------------------------------------------+
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300040, weight=4.409375, user=rg1_user1, role=rg1_role1, query_type in (SELECT), source_ip=192.168.2.1/24)         |
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300041, weight=3.459375, user=rg1_user2, query_type in (SELECT), source_ip=192.168.3.1/24)                         |
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300042, weight=2.359375, user=rg1_user3, source_ip=192.168.4.1/24)                                                 |
| rg1  | 300039 | 10           | 20.0%    | 11               | NORMAL | (id=300043, weight=1.0, user=rg1_user4)                                                                                |
+------+--------+--------------+----------+------------------+--------+------------------------------------------------------------------------------------------------------------------------+
```

> **注記**
>
> 上記の例では、`weight`は一致度を示しています。

### リソースグループと分類器の管理

リソースグループごとにリソース割り当てを変更することができます。また、リソースグループから分類器を追加または削除することもできます。

以下のステートメントを実行して、既存のリソースグループのリソース割り当てを変更します。

```SQL
ALTER RESOURCE GROUP group_name WITH (
    'cpu_core_limit' = 'INT',
    'mem_limit' = 'm%'
);
```

以下のステートメントを実行して、リソースグループを削除します。

```SQL
DROP RESOURCE GROUP group_name;
```

以下のステートメントを実行して、リソースグループに分類器を追加します。

```SQL

ALTER RESOURCE GROUP <group_name> ADD (user='string', role='string', query_type in ('select'), source_ip='cidr');
```

リソースグループから分類子を削除するには、次のステートメントを実行します。

```SQL
ALTER RESOURCE GROUP <group_name> DROP (CLASSIFIER_ID_1, CLASSIFIER_ID_2, ...);
```

リソースグループのすべての分類子を削除するには、次のステートメントを実行します。

```SQL
ALTER RESOURCE GROUP <group_name> DROP ALL;
```

## リソースグループの監視

### クエリのリソースグループを確認

クエリがヒットしたリソースグループは、**fe.audit.log**の`ResourceGroup`列、または`EXPLAIN VERBOSE <query>`を実行した後に返される`RESOURCE GROUP`列から確認できます。これらは、特定のクエリタスクが一致したリソースグループを示します。

- クエリがリソースグループの管理下にない場合、列の値は空文字列`""`です。
- クエリがリソースグループの管理下にあるが、どの分類子にも一致しない場合、列の値は空文字列`""`です。しかし、このクエリはデフォルトリソースグループ`default_wg`に割り当てられます。

`default_wg`のリソース制限は以下の通りです：

- `cpu_core_limit`: 1（v2.3.7以前）またはBEのCPUコア数（v2.3.7以降のバージョン）。
- `mem_limit`: 100%。
- `concurrency_limit`: 0。
- `big_query_cpu_second_limit`: 0。
- `big_query_scan_rows_limit`: 0。
- `big_query_mem_limit`: 0。

### リソースグループのモニタリング

リソースグループに対して[モニタリングとアラート](Monitor_and_Alert.md)を設定できます。

リソースグループ関連のFEおよびBEメトリクスは以下の通りです。以下のすべてのメトリクスには、対応するリソースグループを示す`name`ラベルがあります。

### FEメトリクス

以下のFEメトリクスは、現在のFEノード内の統計のみを提供します：

| メトリック                                          | 単位 | 種類          | 説明                                                        |
| --------------------------------------------------- | ---- | ------------- | ----------------------------------------------------------- |
| starrocks_fe_query_resource_group                   | Count | Instantaneous | このリソースグループで履歴的に実行されたクエリの数（現在実行中のクエリを含む）。 |
| starrocks_fe_query_resource_group_latency           | ms    | Instantaneous | このリソースグループのクエリ待機時間パーセンタイル。ラベル`type`は特定のパーセンタイルを示し、`mean`、`75_quantile`、`95_quantile`、`98_quantile`、`99_quantile`、`999_quantile`を含みます。 |
| starrocks_fe_query_resource_group_err               | Count | Instantaneous | このリソースグループでエラーが発生したクエリの数。 |
| starrocks_fe_resource_group_query_queue_total       | Count | Instantaneous | このリソースグループで履歴的にキューに入れられたクエリの合計数（現在実行中のクエリを含む）。このメトリックはv3.1.4以降でサポートされています。クエリキューが有効な場合にのみ有効です。詳細は[クエリキュー](query_queues.md)を参照してください。 |
| starrocks_fe_resource_group_query_queue_pending     | Count | Instantaneous | このリソースグループのキューに現在入っているクエリの数。このメトリックはv3.1.4以降でサポートされています。クエリキューが有効な場合にのみ有効です。詳細は[クエリキュー](query_queues.md)を参照してください。 |
| starrocks_fe_resource_group_query_queue_timeout     | Count | Instantaneous | このリソースグループでキュー内でタイムアウトしたクエリの数。このメトリックはv3.1.4以降でサポートされています。クエリキューが有効な場合にのみ有効です。詳細は[クエリキュー](query_queues.md)を参照してください。 |

### BEメトリクス

| メトリック                                      | 単位     | 種類          | 説明                                                        |
| ----------------------------------------------- | -------- | ------------- | ----------------------------------------------------------- |
| resource_group_running_queries                  | Count    | Instantaneous | このリソースグループで現在実行されているクエリの数。       |
| resource_group_total_queries                    | Count    | Instantaneous | このリソースグループで履歴的に実行されたクエリの数（現在実行中のクエリを含む）。 |
| resource_group_bigquery_count                   | Count    | Instantaneous | 大クエリ制限をトリガーしたこのリソースグループのクエリの数。 |
| resource_group_concurrency_overflow_count       | Count    | Instantaneous | `concurrency_limit`制限をトリガーしたこのリソースグループのクエリの数。 |
| resource_group_mem_limit_bytes                  | Bytes    | Instantaneous | このリソースグループのメモリ制限。                         |
| resource_group_mem_inuse_bytes                  | Bytes    | Instantaneous | このリソースグループで現在使用中のメモリ。                 |
| resource_group_cpu_limit_ratio                  | Percentage | Instantaneous | このリソースグループの`cpu_core_limit`が全リソースグループの`cpu_core_limit`の合計に対する比率。 |
| resource_group_inuse_cpu_cores                  | Count     | Average       | このリソースグループで使用されている推定CPUコア数。この値は概算であり、2回の連続するメトリック収集の統計に基づいて計算された平均値を表します。このメトリックはv3.1.4以降でサポートされています。 |
| resource_group_cpu_use_ratio                    | Percentage | Average       | **非推奨** このリソースグループで使用されるパイプラインスレッドのタイムスライスの比率が、全リソースグループで使用されるパイプラインスレッドのタイムスライスの合計に対してどの程度かを示します。これは2回の連続するメトリック収集の統計に基づいて計算された平均値を表します。 |
| resource_group_connector_scan_use_ratio         | Percentage | Average       | **非推奨** このリソースグループで使用される外部テーブルスキャンスレッドのタイムスライスの比率が、全リソースグループで使用されるパイプラインスレッドのタイムスライスの合計に対してどの程度かを示します。これは2回の連続するメトリック収集の統計に基づいて計算された平均値を表します。 |
| resource_group_scan_use_ratio                   | Percentage | Average       | **非推奨** このリソースグループで使用される内部テーブルスキャンスレッドのタイムスライスの比率が、全リソースグループで使用されるパイプラインスレッドのタイムスライスの合計に対してどの程度かを示します。これは2回の連続するメトリック収集の統計に基づいて計算された平均値を表します。 |

### リソースグループの使用情報を表示

v3.1.4以降、StarRocksはSQLステートメント[SHOW USAGE RESOURCE GROUPS](../sql-reference/sql-statements/Administration/SHOW_USAGE_RESOURCE_GROUPS.md)をサポートしており、BE全体で各リソースグループの使用情報を表示するために使用されます。各フィールドの説明は以下の通りです：

- `Name`: リソースグループの名前。
- `Id`: リソースグループのID。
- `Backend`: BEのIPまたはFQDN。
- `BEInUseCpuCores`: このBEでこのリソースグループが現在使用しているCPUコアの数。この値は概算です。
- `BEInUseMemBytes`: このBEでこのリソースグループが現在使用しているメモリバイト数。
- `BERunningQueries`: このリソースグループに属するクエリがこのBEで実行中の数。

ご注意ください：

- BEは`report_resource_usage_interval_ms`で指定された間隔（デフォルトは1秒）で、このリソース使用量情報をリーダーFEに定期的に報告します。
- 結果は`BEInUseCpuCores`/`BEInUseMemBytes`/`BERunningQueries`の少なくとも1つが正の数値である行のみを表示します。つまり、リソースグループがBE上でアクティブにリソースを使用している場合にのみ情報が表示されます。

例：

```Plain
MySQL [(none)]> SHOW USAGE RESOURCE GROUPS;
+------------+----+-----------+-----------------+-----------------+------------------+
| Name       | Id | Backend   | BEInUseCpuCores | BEInUseMemBytes | BERunningQueries |
+------------+----+-----------+-----------------+-----------------+------------------+
| default_wg | 0  | 127.0.0.1 | 0.100           | 1               | 5                |
+------------+----+-----------+-----------------+-----------------+------------------+
| default_wg | 0  | 127.0.0.2 | 0.200           | 2               | 6                |
+------------+----+-----------+-----------------+-----------------+------------------+
| wg1        | 0  | 127.0.0.1 | 0.300           | 3               | 7                |
+------------+----+-----------+-----------------+-----------------+------------------+
| wg2        | 0  | 127.0.0.1 | 0.400           | 4               | 8                |
+------------+----+-----------+-----------------+-----------------+------------------+
```

## 次に行うこと

リソースグループを設定した後、メモリリソースとクエリの管理が可能になります。詳細情報については、以下のトピックを参照してください。

- [メモリ管理](../administration/Memory_management.md)

- [クエリ管理](../administration/Query_management.md)
