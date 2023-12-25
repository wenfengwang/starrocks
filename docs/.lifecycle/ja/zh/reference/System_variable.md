---
displayed_sidebar: Chinese
---

# システム変数

StarRocks は、ビジネスの状況に応じて調整できる複数のシステム変数を提供しています。この文書では、StarRocks がサポートする変数について説明します。MySQL クライアントで [SHOW VARIABLES](../sql-reference/sql-statements/Administration/SHOW_VARIABLES.md) コマンドを使用して現在の変数を確認できます。また、[SET](../sql-reference/sql-statements/Administration/SET.md) コマンドを使用して変数を動的に設定または変更することもできます。変数はシステム全体（global）で有効にすることも、現在のセッション（session）でのみ有効にすることも、または単一のクエリステートメントでのみ有効にすることもできます。

StarRocks の変数は MySQL の変数設定を参照していますが、**一部の変数は MySQL クライアントプロトコルとの互換性のためだけに使用され、MySQL データベースでの実際の意味を持ちません**。

> **説明**
>
> どのユーザーでも SHOW VARIABLES を使用して変数を確認する権限があります。どのユーザーでもセッションレベルで変数を設定する権限があります。システムレベルの OPERATE 権限を持つユーザーのみが変数をグローバルに設定することができます。グローバルに設定した後、以降のすべての新しいセッションでは新しい設定が使用され、現在のセッションは以前の設定を引き続き使用します。

## 変数の確認

`SHOW VARIABLES [LIKE 'xxx'];` を使用して、すべてまたは指定された変数を確認できます。例えば：

```SQL

-- システム内のすべての変数を確認。
SHOW VARIABLES;

-- マッチングルールに合致する変数を確認。
SHOW VARIABLES LIKE '%time_zone%';
```

## 変数のレベルとタイプ

StarRocks は、3種類の変数（レベル）をサポートしています：グローバル変数、セッション変数、および `SET_VAR` ヒント。それらのレベル関係は以下の通りです：

* グローバル変数はグローバルレベルで有効であり、セッション変数と `SET_VAR` ヒントによって上書きされる可能性があります。
* セッション変数は現在のセッションでのみ有効であり、`SET_VAR` ヒントによって上書きされる可能性があります。
* `SET_VAR` ヒントは現在のクエリステートメントでのみ有効です。

## 変数の設定

### 変数をグローバルに設定するかセッションで設定する

変数は通常、**グローバルに有効**または**現在のセッションのみに有効**に設定できます。グローバルに設定した後、**以降のすべての新しいセッション**では新しい設定が使用されますが、現在のセッションは以前の設定を引き続き使用します。セッションのみに有効に設定した場合、変数は現在のセッションにのみ影響します。

`SET <var_name> = xxx;` ステートメントで設定された変数は現在のセッションでのみ有効です。例：

```SQL
SET query_mem_limit = 137438953472;

SET forward_to_master = true;

SET time_zone = "Asia/Shanghai";
```

`SET GLOBAL <var_name> = xxx;` ステートメントで設定された変数はグローバルに有効です。例：

 ```SQL
SET GLOBAL query_mem_limit = 137438953472;
 ```

以下の変数はグローバルにのみ有効であり、セッションレベルで設定することはできません。`SET GLOBAL <var_name> = xxx;` を使用する必要があり、`SET <var_name> = xxx;` を使用するとエラーが返されます。

* activate_all_roles_on_login
* character_set_database
* default_rowset_type
* enable_query_queue_select
* enable_query_queue_statistic
* enable_query_queue_load
* init_connect
* lower_case_table_names
* license
* language
* query_cache_size
* query_queue_fresh_resource_usage_interval_ms
* query_queue_concurrency_limit
* query_queue_mem_used_pct_limit
* query_queue_cpu_used_permille_limit
* query_queue_pending_timeout_second
* query_queue_max_queued_queries
* system_time_zone
* version_comment
* version

セッションレベルの変数はグローバルにもセッションレベルにも設定できます。

さらに、変数の設定は定数式もサポートしています。例：

```SQL
SET query_mem_limit = 10 * 1024 * 1024 * 1024;
```

```SQL
SET forward_to_master = concat('tr', 'u', 'e');
```

### 単一のクエリステートメントで変数を設定する

特定のクエリに対して特別に変数を設定する必要がある場合があります。SET_VAR ヒント（hint）を使用して、クエリ内で単一のステートメント内でのみ有効なセッション変数を設定できます。例：

```sql
SELECT /*+ SET_VAR(query_mem_limit = 8589934592) */ name FROM people ORDER BY name;

SELECT /*+ SET_VAR(query_timeout = 1) */ sleep(3);
```

> **注意**
>
> `SET_VAR` は SELECT キーワードの直後にのみ使用でき、`/*+` で始まり `*/` で終わる必要があります。

StarRocks は、単一のステートメントで複数の変数を設定することもサポートしています。以下の例を参照してください：

```sql
SELECT /*+ SET_VAR
  (
  exec_mem_limit = 515396075520,
  query_timeout=10000000,
  batch_size=4096,
  parallel_fragment_exec_instance_num=32
  )
  */ * FROM TABLE;
```

## サポートされる変数

このセクションでは、変数をアルファベット順に説明します。`global` マークが付いた変数はグローバル変数であり、グローバルにのみ有効です。その他の変数はグローバルにもセッションレベルにも設定できます。

### activate_all_roles_on_login（グローバル）（3.0 以降）

ユーザーがログインしたときに、デフォルトですべてのロール（デフォルトロールと付与されたロールを含む）をアクティブにするかどうかを制御します。

* 有効にすると、ユーザーがログインするときにデフォルトですべてのロールがアクティブになり、[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md) で設定されたロールよりも優先されます。

* 有効にしない場合は、SET DEFAULT ROLE で設定されたロールがデフォルトでアクティブになります。

デフォルト値：false、つまり有効にしない。

現在のセッションでロールをアクティブにするには、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md) を使用できます。

### auto_increment_increment

MySQL クライアントとの互換性のために使用されます。実際の効果はありません。デフォルト値は 1 です。

### autocommit

MySQL クライアントとの互換性のために使用されます。実際の効果はありません。デフォルト値は true です。

### batch_size

クエリ実行中に各ノードが転送する単一のデータパケットの行数を指定するために使用されます。デフォルトでは、データパケットは 1024 行で、ソースノードが 1024 行のデータを生成するたびに、目的ノードにパッケージされて送信されます。より多くの行数は、大量のデータをスキャンするシナリオでクエリのスループットを向上させる可能性がありますが、小さなクエリのシナリオではクエリの遅延を増加させる可能性があります。また、クエリのメモリオーバーヘッドも増加します。設定範囲は 1024 から 4096 までを推奨します。

### big_query_profile_second_threshold（3.1 以降）

セッション変数 `enable_profile` が `false` に設定され、クエリ時間が `big_query_profile_second_threshold` で設定された閾値を超えた場合、プロファイルが生成されます。

### cbo_decimal_cast_string_strict（2.5.14 以降）

オプティマイザが DECIMAL 型を STRING 型に変換する動作を制御するために使用されます。`true` に設定すると、v2.5.x 以降のバージョンの処理ロジックが使用され、スケールに従って厳密に変換されます（スケールに従って `0` で埋める）；`false` に設定すると、v2.5.x 以前のバージョンの処理ロジックが保持されます（有効な数字に従って処理されます）。デフォルト値は `true` です。

### cbo_enable_low_cardinality_optimize

低基数のグローバル辞書最適化を有効にするかどうか。有効にすると、STRING 列のクエリ速度が約 3 倍向上します。デフォルト値：true。

### cbo_eq_base_type（2.5.14 以降）

DECIMAL 型と STRING 型のデータ比較時に強制される型を指定するために使用されます。デフォルトは `VARCHAR` で、`DECIMAL` が選択可能です。

### character_set_database（グローバル）

StarRocks データベースがサポートする文字セットで、現在は UTF8 エンコーディング（`utf8`）のみをサポートしています。

### connector_io_tasks_per_scan_operator（2.5 以降）

外部テーブルクエリ時に各スキャンオペレータが同時に発行できる I/O タスクの最大数。整数値で、デフォルトは 16 です。現在、外部テーブルクエリでは、並行 I/O タスクの数を調整するために自己適応アルゴリズムが使用されており、`enable_connector_adaptive_io_tasks` スイッチによって制御されます。デフォルトはオンです。

### count_distinct_column_buckets（2.5 以降）

group-by-count-distinct クエリで count distinct 列に設定されるバケット数。この変数は、`enable_distinct_column_bucketization` が `true` に設定されている場合にのみ有効です。デフォルト値：1024。

### default_rowset_type（グローバル）

グローバル変数であり、グローバルにのみ有効です。計算ノードのストレージエンジンのデフォルトのストレージフォーマットを設定するために使用されます。現在サポートされているストレージフォーマットには、alpha/beta があります。

### default_table_compression（3.0 以降）

テーブルデータの保存時に使用されるデフォルトの圧縮アルゴリズムで、LZ4、Zstandard（または zstd）、zlib、Snappy をサポートしています。デフォルト値：lz4_frame。テーブル作成時に PROPERTIES で `compression` を設定した場合は、`compression` で指定された圧縮アルゴリズムが有効になります。

### disable_colocate_join

Colocate Join 機能を有効にするかどうかを制御します。デフォルトは false で、機能が有効になっています。true は機能を無効にします。この機能が無効になると、クエリプランは Colocate Join を実行しようとしません。

### disable_streaming_preaggregations

ストリーミングプリアグリゲーションを有効にするかどうかを制御します。デフォルトは `false` で、有効になっています。

### div_precision_increment


用于兼容 MySQL 客户端，无实际作用。

### enable_connector_adaptive_io_tasks（2.5 及以后）

外部表查询时是否使用自适应策略来调整 I/O 任务的并发数。默认开启。如果未开启自适应策略，可以通过 `connector_io_tasks_per_scan_operator` 变量来手动设置外部表查询时的 I/O 任务并发数。

### enable_distinct_column_bucketization（2.5 及以后）

是否在 group-by-count-distinct 查询中开启对 count distinct 列的分桶优化。在类似 `select a, count(distinct b) from t group by a;` 的查询中，如果 group by 列 a 为低基数列，count distinct 列 b 为高基数列且发生严重数据倾斜时，会引发查询性能瓶颈。可以通过对 count distinct 列进行分桶来平衡数据，规避数据倾斜。

默认值：false，表示不开启。该变量需要与 `count_distinct_column_buckets` 配合使用。

您也可以通过添加 `skew` hint 来开启 count distinct 列的分桶优化，例如 `select a, count(distinct [skew] b) from t group by a;`。

### enable_group_level_query_queue（3.1.4 及以后）

是否开启资源组粒度的[查询队列](../administration/query_queues.md)。

默认值：false，表示不开启。

### enable_insert_strict

用于设置通过 INSERT 语句进行数据导入时，是否开启严格模式（Strict Mode）。默认为 `true`，即开启严格模式。关于该模式的介绍，可以参阅[严格模式](../loading/load_concept/strict_mode.md)。

### enable_materialized_view_union_rewrite（2.5 及以后）

是否开启物化视图 Union 改写。默认值：`true`。

### enable_rule_based_materialized_view_rewrite（2.5 及以后）

是否开启基于规则的物化视图查询改写功能，主要用于处理单表查询改写。默认值：`true`。

### enable_spill（3.0 及以后）

是否启用中间结果落盘。默认值：`false`。如果将其设置为 `true`，StarRocks 会将中间结果落盘，以减少在查询中处理聚合、排序或连接算子时的内存使用量。

### enable_profile

用于设置是否需要查看查询的 profile。默认为 `false`，即不需要查看 profile。2.5 版本之前，该变量名称为 `is_report_success`，2.5 版本之后更名为 `enable_profile`。

默认情况下，只有在查询发生错误时，BE 才会发送 profile 给 FE，用于查看错误。正常结束的查询不会发送 profile。发送 profile 会产生一定的网络开销，对高并发查询场景不利。当用户希望对一个查询的 profile 进行分析时，可以将这个变量设为 `true` 后，发送查询。查询结束后，可以通过在当前连接的 FE 的 web 页面（地址：fe_host:fe_http_port/query）查看 profile。该页面会显示最近 100 条开启了 `enable_profile` 的查询的 profile。

### enable_query_queue_load（global）

布尔值，用于控制是否为导入任务启用查询队列。默认值：`false`。

### enable_query_queue_select（global）

布尔值，用于控制是否为 SELECT 查询启用查询队列。默认值：`false`。

### enable_query_queue_statistic（global）

布尔值，用于控制是否为统计信息查询启用查询队列。默认值：`false`。

### enable_query_tablet_affinity（2.5 及以后）

布尔值，用于控制在多次查询同一个 tablet 时是否倾向于选择固定的同一个副本。

如果待查询的表中存在大量 tablet，开启该特性会对性能有提升，因为会更快地将 tablet 的元信息以及数据缓存在内存中。但是，如果查询存在一些热点 tablet，开启该特性可能会导致性能有所退化，因为该特性倾向于将一个热点 tablet 的查询调度到相同的 BE 上，在高并发的场景下无法充分利用多台 BE 的资源。

默认值：`false`，表示使用原来的机制，即每次查询会从多个副本中选择一个。自 2.5.6、3.0.8、3.1.4、3.2.0 版本起，StarRocks 支持该参数。

### enable_scan_datacache（2.5 及以后）

是否开启 Data Cache 特性。该特性开启之后，StarRocks 通过将外部存储系统中的热数据缓存成多个 block，加速数据查询和分析。更多信息，参见 [Data Cache](../data_source/data_cache.md)。该特性从 2.5 版本开始支持。在 3.2 之前各版本中，对应变量为 `enable_scan_block_cache`。

### enable_populate_datacache（2.5 及以后）

StarRocks 从外部存储系统读取数据时，是否将数据进行缓存。如果只想读取，不进行缓存，可以将该参数设置为 `false`。默认值为 `true`。在 3.2 之前各版本中，对应变量为 `enable_populate_block_cache`。

### enable_tablet_internal_parallel

是否开启自适应 Tablet 并行扫描，使用多个线程并行分段扫描一个 Tablet，可以减少 Tablet 数量对查询能力的限制。默认值为 `true`。自 2.3 版本起，StarRocks 支持该参数。

### enable_query_cache（2.5 及以后）

是否开启 Query Cache。取值范围：true 和 false。true 表示开启，false 表示关闭（默认值）。开启该功能后，只有当查询满足[Query Cache](../using_starrocks/query_cache.md#应用场景) 所述条件时，才会启用 Query Cache。

### enable_adaptive_sink_dop（2.5 及以后）

是否开启导入自适应并行度。开启后 INSERT INTO 和 Broker Load 自动设置导入并行度，保持和 `pipeline_dop` 一致。新部署的 2.5 版本默认值为 `true`，从 2.4 版本升级上来的默认值为 `false`。

### enable_pipeline_engine

是否启用 Pipeline 执行引擎。true：启用（默认），false：不启用。

### enable_sort_aggregate（2.5 及以后）

是否开启 sorted streaming 聚合。`true` 表示开启 sorted streaming 聚合功能，对流中的数据进行排序。

### enable_global_runtime_filter

Global runtime filter 开关。Runtime Filter（简称 RF）在运行时对数据进行过滤，过滤通常发生在 Join 阶段。当多表进行 Join 时，往往伴随着谓词下推等优化手段进行数据过滤，以减少 Join 表的数据扫描以及 shuffle 等阶段产生的 IO，从而提升查询性能。StarRocks 中有两种 RF，分别是 Local RF 和 Global RF。Local RF 应用于 Broadcast Hash Join 场景。Global RF 应用于 Shuffle Join 场景。

默认值 `true`，表示打开 global runtime filter 开关。关闭该开关后，不生成 Global RF，但依然会生成 Local RF。

### enable_multicolumn_global_runtime_filter

多列 Global runtime filter 开关。默认值为 false，表示关闭该开关。

对于 Broadcast 和 Replicated Join 类型之外的其他 Join，当 Join 的等值条件有多个的情况下：

* 如果该选项关闭，则只会产生 Local RF。
* 如果该选项打开，则会生成 multi-part GRF，并且该 GRF 需要携带 multi-column 作为 partition-by 表达式。

### enable_strict_type（3.1 及以后）

是否对所有复合谓词以及 WHERE 子句中的表达式进行隐式转换。默认值：false。

### ENABLE_WRITE_HIVE_EXTERNAL_TABLE（3.2 及以后）

是否开启往 Hive 的 External Table 写数据的功能。默认值：`false`。

### event_scheduler

用于兼容 MySQL 客户端。无实际作用。

### force_streaming_aggregate

用于控制聚合节点是否启用流式聚合计算策略。默认为 false，表示不启用该策略。

### forward_to_master

用于设置是否将一些命令转发到 Leader FE 节点执行。默认为 false，即不转发。StarRocks 中存在多个 FE 节点，其中一个为 Leader 节点。通常用户可以连接任意 FE 节点进行全功能操作。但部分信息查看指令只有从 Leader FE 节点才能获取详细信息。

如 `SHOW BACKENDS;` 命令，如果不转发到 Leader FE 节点，则仅能看到节点是否存活等一些基本信息，而转发到 Leader FE 则可以获取包括节点启动时间、最后一次心跳时间等更详细的信息。

当前受该参数影响的命令如下：

* SHOW FRONTENDS;

  转发到 Leader 可以查看最后一次心跳信息。

* SHOW BACKENDS;

  Leader への転送により、起動時間、最後のハートビート情報、ディスク容量情報を確認できます。

* SHOW BROKER;

  Leader への転送により、起動時間、最後のハートビート情報を確認できます。

* SHOW TABLET;
* ADMIN SHOW REPLICA DISTRIBUTION;
* ADMIN SHOW REPLICA STATUS;

  Leader への転送により、Leader FE のメタデータに保存されている tablet 情報を確認できます。通常、異なる FE のメタデータにある tablet 情報は一致しているべきです。問題が発生した場合、この方法で現在の FE と Leader FE のメタデータの差異を比較できます。

* SHOW PROC;

  Leader への転送により、Leader FE のメタデータに保存されている関連 PROC の情報を確認できます。主にメタデータの比較に使用されます。

### group_concat_max_len

[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数が返す文字列の最大長さを指定します。単位は文字です。デフォルト値は 1024、最小値は 4 です。

### hash_join_push_down_right_table

Join クエリで右側のテーブルに対するフィルタ条件を使用して左側のテーブルのデータをフィルタリングするかどうかを制御します。これにより、Join 処理中に処理する必要のある左側のテーブルのデータ量を減らすことができます。true に設定すると、この操作が許可され、システムは実際の状況に基づいて左側のテーブルをフィルタリングするかどうかを決定します。false に設定すると、この操作が禁止されます。デフォルト値は true です。

### init_connect (グローバル)

MySQL クライアントとの互換性のために使用されますが、実際の効果はありません。

### interactive_timeout

MySQL クライアントとの互換性のために使用されますが、実際の効果はありません。

### io_tasks_per_scan_operator (2.5 以降)

各 Scan オペレータが同時に発行できる I/O タスクの数です。リモートストレージシステム（例：HDFS や S3）を使用していてレイテンシが長い場合は、この値を増やすことができます。ただし、値が大きすぎるとメモリ消費が増加します。

整数値を取ります。デフォルト値は 4 です。

### language (グローバル)

MySQL クライアントとの互換性のために使用されますが、実際の効果はありません。

### license (グローバル)

StarRocks のライセンスを表示しますが、他の効果はありません。

### load_mem_limit

インポート操作のメモリ制限を指定します。単位はバイトです。デフォルト値は 0 で、この変数を使用せずに `query_mem_limit` をインポート操作のメモリ制限として使用することを意味します。

この変数は INSERT 操作にのみ使用されます。INSERT 操作はクエリとインポートの二つの部分を含むため、この変数を設定しない場合、クエリとインポート操作の両方のメモリ制限は `query_mem_limit` になります。それ以外の場合、INSERT のクエリ部分のメモリ制限は `query_mem_limit` になり、インポート部分の制限は `load_mem_limit` になります。

Broker Load、STREAM LOAD などの他のインポート方法のメモリ制限は引き続き `query_mem_limit` を使用します。

### log_rejected_record_num（3.1 以降）

データ品質が不適合でフィルタリングされたデータ行の最大記録数を指定します。範囲は `0`、`-1`、0 より大きい整数です。デフォルト値は `0` です。

* `0` はフィルタリングされたデータ行を記録しないことを意味します。
* `-1` はすべてのフィルタリングされたデータ行を記録することを意味します。
* 0 より大きい整数（例：`n`）は、各 BE ノードで最大 `n` 行のフィルタリングされたデータ行を記録できることを意味します。

### lower_case_table_names (グローバル)

MySQL クライアントとの互換性のために使用されますが、実際の効果はありません。StarRocks ではテーブル名は大文字と小文字を区別します。

### materialized_view_rewrite_mode（3.2 以降）

非同期マテリアライズドビューのクエリ改写モードを指定します。有効な値は以下の通りです：

* `disable`：非同期マテリアライズドビューの自動クエリ改写を無効にします。
* `default`（デフォルト値）：非同期マテリアライズドビューの自動クエリ改写を有効にし、オプティマイザーがコストに基づいてマテリアライズドビューを使用してクエリを改写できるかどうかを決定します。クエリが改写できない場合は、基本テーブルのデータを直接クエリします。
* `default_or_error`：非同期マテリアライズドビューの自動クエリ改写を有効にし、オプティマイザーがコストに基づいてマテリアライズドビューを使用してクエリを改写できるかどうかを決定します。クエリが改写できない場合はエラーを返します。
* `force`：非同期マテリアライズドビューの自動クエリ改写を有効にし、オプティマイザーが優先的にマテリアライズドビューを使用してクエリを改写します。クエリが改写できない場合は、基本テーブルのデータを直接クエリします。
* `force_or_error`：非同期マテリアライズドビューの自動クエリ改写を有効にし、オプティマイザーが優先的にマテリアライズドビューを使用してクエリを改写します。クエリが改写できない場合はエラーを返します。

### max_allowed_packet

JDBC コネクションプール C3P0 との互換性のために使用されます。この変数の値は、サーバーがクライアントに送信する、またはクライアントがサーバーに送信する最大パケットサイズを決定します。単位はバイトで、デフォルト値は 32 MB です。クライアントが `PacketTooBigException` 例外を報告した場合は、この値を増やすことを検討してください。

### max_pushdown_conditions_per_column

この変数の具体的な意味については、[BE 設定項目](../administration/BE_configuration.md#配置-be-动态参数)の `max_pushdown_conditions_per_column` の説明を参照してください。デフォルト値は -1 で、`be.conf` の設定値を使用します。0 より大きい値を設定すると、`be.conf` の設定値は無視されます。

### max_scan_key_num

この変数の具体的な意味については、[BE 設定項目](../administration/BE_configuration.md#配置-be-动态参数)の `max_scan_key_num` の説明を参照してください。デフォルト値は -1 で、`be.conf` の設定値を使用します。0 より大きい値を設定すると、`be.conf` の設定値は無視されます。

### nested_mv_rewrite_max_level

クエリ改写に使用できるネストされたマテリアライズドビューの最大レベルです。タイプ：INT。範囲：[1, +∞)。デフォルト値は `3` です。`1` に設定すると、基本テーブルに基づいて作成されたマテリアライズドビューのみをクエリ改写に使用できます。

### net_buffer_length

MySQL クライアントとの互換性のために使用されますが、実際の効果はありません。

### net_read_timeout

MySQL クライアントとの互換性のために使用されますが、実際の効果はありません。

### net_write_timeout

MySQL クライアントとの互換性のために使用されますが、実際の効果はありません。

### new_planner_optimize_timeout

クエリオプティマイザのタイムアウト時間です。通常、Join が多いクエリではタイムアウトが発生しやすいです。タイムアウト後はエラーが報告され、クエリが停止し、クエリのパフォーマンスに影響を与えます。クエリの具体的な状況に応じてこのパラメータを増やすか、問題を StarRocks の技術サポートに報告して調査してもらうことができます。

単位は秒。デフォルト値は 3000。

### parallel_exchange_instance_num

実行計画で、上層ノードが下層ノードのデータを受け取るために使用する Exchange Node の数を設定します。デフォルトは -1 で、Exchange Node の数は下層ノードの実行インスタンスの数と同じです（デフォルトの動作）。0 より大きく、下層ノードの実行インスタンスの数より小さい値を設定すると、Exchange Node の数は設定値と同じになります。

分散型クエリ実行計画では、通常、上層ノードには一つまたは複数の Exchange Node があり、異なる BE 上の下層ノードの実行インスタンスからのデータを受け取ります。通常、Exchange Node の数は下層ノードの実行インスタンスの数と同じです。

一部の集約クエリのシナリオでは、下層がスキャンする必要のあるデータ量が多いが、集約後のデータ量が少ない場合、この変数を小さい値に変更してみると、この種のクエリのリソース消費を減らすことができます。例えば、DUPLICATE KEY 詳細モデルでの集約クエリのシナリオです。

### parallel_fragment_exec_instance_num

スキャンノードに対して、各 BE ノード上で実行するインスタンスの数を設定します。デフォルトは 1 です。

クエリ計画は通常、一連の scan range、つまりスキャンする必要のあるデータ範囲を生成します。これらのデータは複数の BE ノードに分散しています。一つの BE ノードには一つまたは複数の scan range があります。デフォルトでは、各 BE ノードの一連の scan range は一つの実行インスタンスによって処理されます。マシンリソースが豊富な場合は、この変数を増やして、複数の実行インスタンスが同時に一連の Scan Range を処理することで、クエリの効率を向上させることができます。

また、scan インスタンスの数は、上層の他の実行ノード、例えば集約ノード、join ノードの数を決定します。したがって、クエリ計画の実行の並行度を全体的に増やすことになります。このパラメータを変更すると、大規模なクエリの効率が向上する可能性がありますが、大きな値はより多くのマシンリソース、例えば CPU、メモリ、ディスク I/O を消費します。

### partial_update_mode (3.1 以降)

部分更新のモードを制御します。取りうる値は以下の通りです：

* `auto`（デフォルト値）は、システムが更新ステートメントと関連する列を分析して、部分更新を実行する際のモードを自動的に決定することを意味します。
* `column`は、列モードで部分更新を実行することを指定し、少数の列に関連し、多数の行がある部分列更新のシナリオに適しています。

詳細については、[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md#列模式の部分更新自-31)を参照してください。

### performance_schema

用于兼容 8.0.16 及以上版本的 MySQL JDBC。无实际作用。

### pipeline_dop

Pipeline インスタンスの並列数。インスタンスの並列数を設定することでクエリの並行度を調整できます。デフォルト値は 0 で、システムが各 pipeline の並列度を自動調整します。0 より大きい値を設定することもでき、通常は BE ノードの CPU 物理コア数の半分に設定します。バージョン 3.0 からは、クエリの並行度に基づいて `pipeline_dop` を自動調整する機能がサポートされています。

### pipeline_profile_level

profile のレベルを制御するために使用します。profile には通常、5つのレベルがあります：Fragment、FragmentInstance、Pipeline、PipelineDriver、Operator。レベルによって、profile の詳細度が異なります：

* 0：このレベルでは、StarRocks は profile をマージし、いくつかのコア指標のみを表示します。
* 1：デフォルト値。このレベルでは、StarRocks は profile を簡略化し、同じ pipeline の指標をマージしてレベルを減らします。
* 2：このレベルでは、StarRocks は profile のすべてのレベルを保持し、簡略化しません。この設定では profile のサイズが非常に大きくなり、特に SQL が複雑な場合は推奨されません。

### prefer_compute_node

一部の実行計画を CN ノードで実行するようにスケジュールします。デフォルトは false です。この変数はバージョン 2.4 からサポートされています。

### query_cache_size (global)

MySQL クライアントとの互換性のために使用します。実際の効果はありません。

### query_cache_type

JDBC コネクションプール C3P0 との互換性のために使用します。実際の効果はありません。

### query_cache_entry_max_bytes (2.5 以降)

Cache 項目のバイト数上限で、Passthrough モードをトリガーする閾値です。範囲：0 ~ 9223372036854775807。デフォルト値：4194304。Tablet が生成する計算結果のバイト数または行数が `query_cache_entry_max_bytes` または `query_cache_entry_max_rows` で指定された閾値を超えると、クエリは Passthrough モードで実行されます。`query_cache_entry_max_bytes` または `query_cache_entry_max_rows` の値が 0 の場合、Tablet が結果を生成しなくても Passthrough モードが使用されます。

### query_cache_entry_max_rows (2.5 以降)

Cache 項目の行数上限で、`query_cache_entry_max_bytes` の説明を参照してください。デフォルト値：409600。

### query_cache_agg_cardinality_limit (2.5 以降)

GROUP BY 集約の高基数の上限です。GROUP BY 集約の出力がこの行数を超えると予測される場合、cache は使用されません。デフォルト値：5000000。

### query_mem_limit

各 BE ノード上のクエリのメモリ制限を設定するために使用します。デフォルト値は `0` で、制限なしを意味します。`B`、`K`、`KB`、`M`、`MB`、`G`、`GB`、`T`、`TB`、`P`、`PB` などの単位がサポートされています。

### query_queue_concurrency_limit (global)

単一の BE ノードでの並行クエリの上限です。`0` より大きい値に設定された場合のみ有効になります。デフォルト値：`0`。

### query_queue_cpu_used_permille_limit (global)

単一の BE ノードでの CPU 使用率の千分率上限です（つまり CPU 使用率 ### 1000）。`0` より大きい値に設定された場合のみ有効になります。デフォルト値：`0`。範囲：[0, 1000]

### query_queue_max_queued_queries (global)

キュー内のクエリ数の上限です。この閾値に達した場合、新しいクエリは実行を拒否されます。`0` より大きい値に設定された場合のみ有効になります。デフォルト値：`1024`。

### query_queue_mem_used_pct_limit (global)

単一の BE ノードでのメモリ使用率のパーセンテージ上限です。`0` より大きい値に設定された場合のみ有効になります。デフォルト値：`0`。範囲：[0, 1]

### query_queue_pending_timeout_second (global)

キュー内の単一クエリの最大タイムアウト時間です。この閾値に達した場合、クエリは実行を拒否されます。デフォルト値：`300`。単位：秒。

### query_timeout

クエリのタイムアウト時間を秒単位で設定するために使用します。この変数は現在の接続内のすべてのクエリ文と INSERT 文に適用されます。デフォルトは 300 秒、つまり 5 分です。範囲：1 ~ 259200。

### range_pruner_max_predicate (3.0 以降)

Range パーティションのプルーニング時に使用できる IN 述語の最大数を設定します。デフォルト値：100。この値を超えると、すべての tablet がスキャンされ、クエリのパフォーマンスが低下します。

### resource_group

現在は使用されていません。

### rewrite_count_distinct_to_bitmap_hll

Bitmap と HLL 型の count distinct クエリを bitmap_union_count と hll_union_agg に書き換えるかどうか。

### runtime_filter_on_exchange_node

GRF が Exchange オペレータを越えて成功裏にプッシュダウンされた後、Exchange Node 上に GRF を配置するかどうか。GRF が Exchange オペレータを越えてプッシュダウンされ、最終的に Exchange オペレータより下層のオペレータに配置される場合、Exchange オペレータ自体は GRF を配置しません。これにより、GRF を使用してデータをフィルタリングする際の計算時間の増加を避けることができます。しかし、GRF の配信は try-best モードであり、クエリ実行時に Exchange 下層のオペレータが GRF の受信に失敗し、Exchange 自体に GRF が配置されていない場合、データはフィルタリングされず、パフォーマンスが低下する可能性があります。

このオプションをオン（`true` に設定）にすると、GRF が Exchange オペレータを越えてプッシュダウンされても、Exchange オペレータ上に GRF を配置し、有効にします。デフォルト値は `false` です。

### runtime_join_filter_push_down_limit

Bloomfilter 型の Local RF の Hash Table の行数のしきい値を生成します。このしきい値を超えると、Local RF は生成されません。この変数は大きすぎる Local RF の生成を避けるためにあります。整数値で、行数を表します。デフォルト値：1024000。

### runtime_profile_report_interval

Runtime Profile の報告間隔です。この変数は v3.1.0 からサポートされています。

単位：秒、デフォルト値：`10`。

### spill_mode (3.0 以降)

中間結果をディスクに書き出す実行方法です。デフォルト値：`auto`。有効な値は以下の通りです：

* `auto`：メモリ使用のしきい値に達すると、自動的に書き出しがトリガーされます。
* `force`：メモリ使用状況に関わらず、StarRocks は関連するすべてのオペレータの中間結果を強制的にディスクに書き出します。

この変数は `enable_spill` が `true` に設定されている場合のみ有効です。

### SQL_AUTO_IS_NULL

JDBC コネクションプール C3P0 との互換性のために使用します。実際の効果はありません。デフォルト値：false。

### sql_dialect (3.0 以降)

有効な SQL 文法を設定します。例えば、`set sql_dialect = 'trino';` コマンドを実行すると Trino 文法に切り替えられ、Trino 固有の SQL 文法や関数をクエリで使用できます。

> **注意**
>
> Trino 文法を使用する設定にすると、クエリはデフォルトで大文字と小文字を区別しません。そのため、StarRocks でデータベースやテーブルを作成する際は、必ず小文字のデータベース名やテーブル名を使用する必要があります。そうでないとクエリが失敗します。

### sql_mode

特定の SQL 方言に適応するために SQL モードを指定するために使用します。

### sql_safe_updates

MySQL クライアントとの互換性のために使用します。実際の効果はありません。

### sql_select_limit

MySQL クライアントとの互換性のために使用します。実際の効果はありません。

### statistic_collect_parallel

BE で並行して実行できる統計情報収集タスクの数を調整するために使用します。デフォルト値は 1 ですが、この数値を増やすことで収集タスクの実行速度を速めることができます。

### storage_engine

使用するストレージエンジンを指定します。StarRocks がサポートするエンジンタイプには以下が含まれます：

* olap：StarRocks の独自エンジン。
* mysql：MySQL 外部テーブルを使用。
* broker：Broker プログラムを介して外部テーブルにアクセス。
* ELASTICSEARCH または es：Elasticsearch 外部テーブルを使用。
* HIVE：Hive 外部テーブルを使用。
* ICEBERG：Iceberg 外部テーブルを使用。バージョン 2.1 からサポートされています。
* HUDI：Hudi 外部テーブルを使用。バージョン 2.2 からサポートされています。
* jdbc：JDBC 外部テーブルを使用。バージョン 2.3 からサポートされています。

### streaming_preaggregation_mode

複数段階の集約において、group-by の第一段階の事前集約方法を設定します。第一段階のローカル事前集約の効果が良くない場合は、事前集約をオフにしてストリーミング方式に切り替え、データをシンプルにシリアライズして送信することができます。取りうる値は以下の通りです：

* `auto`：最初にローカル事前集約を試み、効果が良ければローカル事前集約を行い、そうでなければストリーミングに切り替えます。デフォルト値で、このままでの使用を推奨します。
* `force_preaggregation`：探索せず、直接ローカル事前集約を行います。
* `force_streaming`：探索せず、直接ストリーミングを行います。

### system_time_zone (global)

現在のシステムタイムゾーンを表示します。変更はできません。

### time_zone

現在のセッションのタイムゾーンを設定するために使用します。タイムゾーンは一部の時間関数の結果に影響を与えます。

### tx_isolation

MySQL クライアントとの互換性のために使用します。実際の効果はありません。

### use_compute_nodes

CNノードの使用数の上限を設定するために使用されます。この設定は `prefer_compute_node=true` の場合にのみ有効になります。

-1 はすべてのCNノードを使用することを意味し、0 はCNノードを使用しないことを意味します。デフォルト値は -1 です。この変数はバージョン2.4からサポートされています。

### use_v2_rollup

クエリがSegment v2のストレージフォーマットのRollupインデックスを使用してデータを取得することを制御するために使用されます。この変数はSegment v2をオンラインにする際の検証に使用されます。それ以外の場合の使用は推奨されません。

### vectorized_engine_enable (バージョン2.4から廃止)

クエリ実行にベクトル化エンジンを使用するかどうかを制御するために使用されます。値がtrueの場合はベクトル化エンジンを使用し、そうでない場合は非ベクトル化エンジンを使用します。デフォルト値はtrueです。バージョン2.4からはデフォルトでオンになっているため、この変数は廃止されました。

### version (グローバル)

MySQLサーバーのバージョン。

### version_comment (グローバル)

StarRocksのバージョンを表示するために使用され、変更可能です。

### wait_timeout

アイドル状態の接続のタイムアウト時間を秒単位で設定するために使用されます。アイドル状態の接続がこの時間内にStarRocksとの交流がない場合、StarRocksはその接続を自動的に切断します。デフォルトは8時間です。
