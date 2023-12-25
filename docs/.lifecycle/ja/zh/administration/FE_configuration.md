---
displayed_sidebar: Chinese
---

# FE 設定項目

FE パラメータは、動的パラメータと静的パラメータに分けられます。動的パラメータは、SQL コマンドを使用してオンラインで設定および調整することができ、便利です。**SQL コマンドで行った動的設定は、FE を再起動すると無効になります。設定を永続的に有効にするには、fe.conf ファイルも同時に変更することをお勧めします。**

静的パラメータは、FE 設定ファイル **fe.conf** で設定および調整する必要があります。**調整が完了したら、FE を再起動して変更を有効にする必要があります。**

パラメータが動的パラメータかどうかは、[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md) の結果の `IsMutable` 列で確認できます。`TRUE` は動的パラメータを示します。

静的パラメータと動的パラメータの両方を **fe.conf** ファイルで変更できます。

## FE 設定項目の表示

FE を起動した後、MySQL クライアントで ADMIN SHOW FRONTEND CONFIG コマンドを実行してパラメータの設定を表示することができます。特定のパラメータの設定を表示する場合は、次のコマンドを実行します：

```SQL
 ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"];
 ```

詳細なコマンドのフィールドの説明については、[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md) を参照してください。

> **注意**
>
> クラスタ管理関連のコマンドを実行するには、`cluster_admin` の役割を持つユーザーである必要があります。


## FE 動的パラメータの設定

[ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) コマンドを使用して、FE の動的パラメータをオンラインで変更できます。

```sql
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

> **注意**
>
> 動的設定されたパラメータは、FE を再起動すると **fe.conf** ファイルの設定またはデフォルト値に戻ります。設定を永続的に有効にするには、設定後に **fe.conf** ファイルも同時に変更することをお勧めします。

このセクションでは、FE の動的パラメータを以下のように分類しています：

- [ログ](#log)
- [メタデータとクラスタ管理](#メタデータとクラスタ管理)
- [クエリエンジン](#query-engine)
- [インポートとエクスポート](#インポートとエクスポート)
- [ストレージ](#ストレージ)
- [その他の動的パラメータ](#その他の動的パラメータ)

### ログ

#### qe_slow_log_ms

- 含意：Slow query の判定時間。この閾値を超えるクエリの応答時間は、監査ログ `fe.audit.log` に slow query として記録されます。
- 単位：ミリ秒
- デフォルト値：5000

### メタデータとクラスタ管理

#### catalog_try_lock_timeout_ms

- 含意：グローバルロックの取得タイムアウト時間。
- 単位：ミリ秒
- デフォルト値：5000

#### edit_log_roll_num

- 含意：このパラメータは、ログファイルのサイズを制御し、指定された数のメタデータログを書き込むごとにログローリング操作を実行してこれらのログの新しいログファイルを生成します。新しいログファイルは BDBJE データベースに書き込まれます。
- デフォルト値：50000

#### ignore_unknown_log_id

- 含意：未知の logID を無視するかどうか。FE が低いバージョンにロールバックされる場合、低いバージョンの BE が認識できない logID が存在する可能性があります。<br />TRUE の場合、FE はこれらの logID を無視します。それ以外の場合、FE は終了します。
- デフォルト値：FALSE

#### ignore_materialized_view_error

- 含意：物理ビューエラーによるメタデータの異常を無視するかどうか。FE が物理ビューエラーによるメタデータの異常で起動できない場合、このパラメータを `true` に設定してエラーを無視することができます。
- デフォルト値：FALSE
- 導入バージョン：2.5.10

#### ignore_meta_check

- 含意：メタデータの遅延を無視するかどうか。true の場合、非メイン FE はメイン FE と自身の間のメタデータの遅延間隔を無視し、メタデータの遅延間隔が meta_delay_toleration_second を超えていても、非メイン FE は読み取りサービスを提供し続けます。<br />メイン FE を長時間停止しようとしているが、非メイン FE が読み取りサービスを提供し続ける場合には、このパラメータが役立ちます。
- デフォルト値：FALSE

#### meta_delay_toleration_second

- 含意：FE が所属する StarRocks クラスタで、リーダーではない FE が許容できるメタデータの遅延の最大時間。<br />非リーダー FE のメタデータとリーダー FE のメタデータの間の遅延時間がこのパラメータの値を超える場合、非リーダー FE はサービスを停止します。
- 単位：秒
- デフォルト値：300

#### drop_backend_after_decommission

- 含意：BE をオフラインにした後、その BE を削除するかどうか。true の場合、BE をオフラインにした後、その BE はすぐに削除されます。False の場合、オフラインにした後も BE は削除されません。
- デフォルト値：TRUE

#### enable_collect_query_detail_info

- 含意：クエリのプロファイル情報を収集するかどうか。true に設定すると、システムはクエリのプロファイルを収集します。false に設定すると、システムはクエリのプロファイルを収集しません。
- デフォルト値：FALSE

#### enable_background_refresh_connector_metadata

- 含意：Hive メタデータキャッシュの定期的なリフレッシュを有効にするかどうか。有効にすると、StarRocks は Hive クラスタのメタデータサービス（Hive Metastore または AWS Glue）をポーリングし、頻繁にアクセスされる Hive の外部データディレクトリのメタデータキャッシュをリフレッシュしてデータの更新を感知します。`true` は有効、`false` は無効を表します。
- デフォルト値：v3.0 は TRUE；v2.5 は FALSE

#### background_refresh_metadata_interval_millis

- 含意：2回の Hive メタデータキャッシュのリフレッシュ間隔。
- 単位：ミリ秒
- デフォルト値：600000
- 導入バージョン：2.5.5

#### background_refresh_metadata_time_secs_since_last_access_secs

- 含意：Hive メタデータキャッシュのリフレッシュタスクの有効期限。アクセス済みの Hive カタログに対して、この時間以上アクセスがない場合、そのカタログのメタデータキャッシュのリフレッシュを停止します。アクセスされていない Hive カタログに対しては、StarRocks はそのメタデータキャッシュをリフレッシュしません。
- デフォルト値：86400
- 単位：秒
- 導入バージョン：2.5.5

#### enable_statistics_collect_profile

- 含意：統計情報のクエリ時にプロファイルを生成するかどうか。この設定を `true` にすると、StarRocks はシステム統計クエリのためにプロファイルを生成します。
- デフォルト値：false
- 導入バージョン：3.1.5

### クエリエンジン

#### max_allowed_in_element_num_of_delete

- 含意：DELETE ステートメントの IN 句で許可される要素の最大数。
- デフォルト値：10000

#### enable_materialized_view

- 含意：物理ビューの作成を許可するかどうか。
- デフォルト値：TRUE

#### enable_decimal_v3

- 含意：Decimal V3 を有効にするかどうか。
- デフォルト値：TRUE

#### enable_sql_blacklist

- 含意：SQL クエリのブラックリストのチェックを有効にするかどうか。有効にすると、ブラックリストにあるクエリは実行できません。
- デフォルト値：FALSE

#### dynamic_partition_check_interval_seconds

- 含意：ダイナミックパーティションのチェック間隔。新しいデータが生成されると、パーティションが自動的に作成されます。
- 単位：秒
- デフォルト値：600

#### dynamic_partition_enable

- 含意：ダイナミックパーティション機能を有効にするかどうか。有効にすると、新しいデータに対して動的にパーティションを作成できます。また、StarRocks は自動的に期限切れのパーティションを削除してデータのタイムリネスを確保します。
- デフォルト値：TRUE

#### http_slow_request_threshold_ms

- 含意：HTTP リクエストの時間がこのパラメータで指定された時間を超える場合、そのリクエストを追跡するためのログが生成されます。
- 単位：ミリ秒
- デフォルト値：5000
- 導入バージョン：2.5.15、3.1.5

#### max_partitions_in_one_batch

- 含意：バッチで作成するパーティションの最大数。
- デフォルト値：4096

#### max_query_retry_time

- 含意：FE 上でのクエリのリトライの最大回数。
- デフォルト値：2

#### max_create_table_timeout_second

- 含意：テーブル作成の最大タイムアウト時間。
- 単位：秒
- デフォルト値：600

#### max_running_rollup_job_num_per_table

- 含意：テーブルごとの Rollup ジョブの最大並行数。
- デフォルト値：1

#### max_planner_scalar_rewrite_num

- 含意：最大の ScalarOperator のリライト回数。
- デフォルト値：100000

#### enable_statistic_collect

- 含意：統計情報の収集を有効にするかどうか。このスイッチはデフォルトでオンになっています。
- デフォルト値：TRUE

#### enable_collect_full_statistic

- 含意：自動的なフル統計情報の収集を有効にするかどうか。このスイッチはデフォルトでオンになっています。
- デフォルト値：TRUE

#### statistic_auto_collect_ratio

- 含意：自動統計情報のヘルススコアの閾値。統計情報のヘルススコアがこの閾値未満の場合、自動収集がトリガされます。
- デフォルト値：0.8

#### statistic_max_full_collect_data_size

- 含意：自動統計情報収集の最大パーティションサイズ。この値を超える場合、フル収集を中止し、テーブルのサンプリング収集に切り替えます。
- 単位：GB
- デフォルト値：100

#### statistic_collect_interval_sec

- 含意：自動定期収集タスクでデータの更新を検出する間隔。デフォルトでは 5 分間隔で検出します。
- 単位：秒
- デフォルト値：300

#### statistic_auto_analyze_start_time

- 含意：自動フル収集の開始時間を設定します。値の範囲は `00:00:00` 〜 `23:59:59` です。
- タイプ：STRING
- デフォルト値：00:00:00
- 導入バージョン：2.5.0

#### statistic_auto_analyze_end_time

- 含意：自動フル収集の終了時間を設定します。値の範囲は `00:00:00` 〜 `23:59:59` です。
- タイプ：STRING
- デフォルト値：23:59:59
- 導入バージョン：2.5.0

#### statistic_sample_collect_rows

- 含意：最小サンプリング行数。サンプリング収集（SAMPLE）の収集タイプが指定されている場合、このパラメータを設定する必要があります。<br />パラメータの値が実際のテーブルの行数を超える場合、デフォルトでフル収集が実行されます。
- デフォルト値：200000

#### histogram_buckets_size

- 含意：ヒストグラムのデフォルトのバケット数。
- デフォルト値：64

#### histogram_mcv_size

- 含意：ヒストグラムのデフォルトの最頻値の数。
- デフォルト値：100

#### histogram_sample_ratio

- 含意：ヒストグラムのデフォルトのサンプリング比率。
- デフォルト値：0.1

#### histogram_max_sample_row_count

- 含意：ヒストグラムの最大サンプリング行数。
- デフォルト値：10000000

#### statistics_manager_sleep_time_sec

- 含意：統計情報関連のメタデータスケジュール間隔。この間隔に基づいて、システムは次の操作を実行します：<ul><li>統計情報テーブルの作成</li><li>削除されたテーブルの統計情報の削除</li><li>期限切れの統計情報の履歴レコードの削除</li></ul>
- 単位：秒
- デフォルト値：60

#### statistic_update_interval_sec

- 含意：統計情報のメモリキャッシュの有効期限。
- 単位：秒
- デフォルト値：24 \* 60 \* 60

#### statistic_analyze_status_keep_second

- 含意：統計情報収集タスクの記録の保持時間。デフォルトでは 3 日間保持されます。
- 単位：秒
- デフォルト値：259200

#### statistic_collect_concurrency

- 含意：手動収集タスクの最大並行数。デフォルトでは、最大 3 つの手動収集タスクが同時に実行されます。それ以上のタスクは PENDING 状態で待機します。
- デフォルト値：3

#### statistic_auto_collect_small_table_rows

- 含意：自動収集中に外部データソース（Hive、Iceberg、Hudi）のテーブルが小さいかどうかを判断するための行数の閾値。
- デフォルト値：10000000
- 導入バージョン：v3.2

#### enable_local_replica_selection


- 含义：クエリの実行にローカルコピーを選択するかどうかを指定します。ローカルコピーを選択すると、データ転送のネットワーク遅延を減らすことができます。<br />trueに設定すると、オプティマイザは現在のFEと同じIPのBEノード上のタブレットコピーを優先的に選択します。falseに設定すると、ローカルまたは非ローカルのコピーを選択します。
- デフォルト値：FALSE

#### max_distribution_pruner_recursion_depth

- 含义：パーティションのプルーナーが許可する最大再帰深度です。再帰の深度を増やすと、より多くの要素をプルーニングできますが、CPUリソースの消費も増えます。
- デフォルト値：100

#### enable_udf

- 含义：UDFを有効にするかどうかを指定します。
- デフォルト値：FALSE

### インポートとエクスポート

#### max_broker_load_job_concurrency

- 含义：StarRocksクラスタで並行して実行できるブローカーロードジョブの最大数です。このパラメータはブローカーロードにのみ適用されます。値は `max_running_txn_num_per_db` より小さくする必要があります。2.5バージョン以降、このパラメータのデフォルト値は `10` から `5` に変更されました。パラメータの別名は `async_load_task_pool_size` です。
- デフォルト値：5

#### load_straggler_wait_second

- 含义：BEレプリカの最大許容インポート遅延時間を制御します。この時間を超えるとクローンが実行されます。
- 単位：秒
- デフォルト値：300

#### desired_max_waiting_jobs

- 含义：最大待機ジョブ数です。テーブルの作成、インポート、スキーマの変更など、すべてのジョブに適用されます。<br />FEのPENDING状態のジョブ数がこの値に達すると、FEは新しいインポートリクエストを拒否します。このパラメータの設定は非同期実行のインポートにのみ適用されます。2.5バージョン以降、このパラメータのデフォルト値は `100` から `1024` に変更されました。
- デフォルト値：1024

#### max_load_timeout_second

- 含义：インポートジョブの最大タイムアウト時間です。すべてのインポートに適用されます。
- 単位：秒
- デフォルト値：259200

#### min_load_timeout_second

- 含义：インポートジョブの最小タイムアウト時間です。すべてのインポートに適用されます。
- 単位：秒
- デフォルト値：1

#### max_running_txn_num_per_db

- 含义：StarRocksクラスタ内の各データベースで実行中のインポート関連トランザクションの最大数です。デフォルト値は `1000` です。3.1バージョン以降、デフォルト値は100から1000に変更されました。<br />データベース内で実行中のインポート関連トランザクションが最大数を超えると、後続のインポートは実行されません。同期的なインポートジョブリクエストの場合、ジョブは拒否されます。非同期のインポートジョブリクエストの場合、ジョブはキューで待機します。この値を大きくすることはお勧めしません。システムの負荷が増加します。
- デフォルト値：1000

#### load_parallel_instance_num

- 含义：1つのBEで実行できるジョブの最大並行インスタンス数です。3.1バージョン以降、非推奨となりました。
- デフォルト値：1

#### disable_load_job

- 含义：いかなるインポートタスクも無効にするかどうかを指定します。クラスタに問題が発生した場合のセーフティメカニズムです。
- デフォルト値：FALSE

#### history_job_keep_max_second

- 含义：保持できる最大の履歴ジョブの期間です。スキーマ変更などのジョブに適用されます。
- 単位：秒
- デフォルト値：604800

#### label_keep_max_num

- 含义：一定期間内に保持できるインポートジョブの最大数です。超えると、履歴インポートジョブの情報が削除されます。
- デフォルト値：1000

#### label_keep_max_second

- 含义：StarRocksシステムラベルで記録された、完了してFINISHEDまたはCANCELLED状態にあるインポートジョブの保持期間です。デフォルト値は3日です。<br />このパラメータの設定はすべてのモードのインポートジョブに適用されます。- 単位：秒。大きな値を設定すると、大量のメモリを消費します。
- デフォルト値：259200

#### max_routine_load_task_concurrent_num

- 含义：各Routine Loadジョブで同時に実行できるタスクの最大数です。
- デフォルト値：5

#### max_routine_load_task_num_per_be

- 含义：各BEで同時に実行できるRoutine Loadインポートタスクの最大数です。3.1.0以降、デフォルト値は5から16に変更され、BEの設定項目 `routine_load_thread_pool_size`（非推奨）は不要になりました。
- デフォルト値：16

#### max_routine_load_batch_size

- 含义：各Routine Loadタスクでインポートできる最大データ量です。
- 単位：バイト
- デフォルト値：4294967296

#### routine_load_task_consume_second

- 含义：クラスタ内の各Routine Loadインポートタスクがデータを消費する最大時間です。<br />v3.1.0以降、Routine Loadインポートジョブ [job_properties](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties) にパラメータ `task_consume_second` が追加され、単一のRoutine Loadインポートジョブ内のインポートタスクに適用されます。より柔軟になりました。
- 単位：秒
- デフォルト値：15

#### routine_load_task_timeout_second

- 含义：クラスタ内の各Routine Loadインポートタスクのタイムアウト時間です。- 単位：秒。<br />v3.1.0以降、Routine Loadインポートジョブ [job_properties](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties) にパラメータ `task_timeout_second` が追加され、単一のRoutine Loadインポートジョブ内のタスクに適用されます。より柔軟になりました。
- 単位：秒
- デフォルト値：60

#### routine_load_unstable_threshold_second
- 含义：Routine Loadインポートジョブのいずれかのインポートタスクの消費遅延時間です。つまり、消費中のメッセージのタイムスタンプと現在の時間の差がこの閾値を超え、かつデータソースに未消費のメッセージが存在する場合、インポートジョブはUNSTABLE状態になります。
- 単位：秒
- デフォルト値：3600

#### max_tolerable_backend_down_num

- 含义：許容される最大の故障BE数です。故障したBEノードの数がこの閾値を超えると、Routine Loadジョブは自動的に復旧されません。
- デフォルト値：0

#### period_of_auto_resume_min

- 含义：Routine Loadの自動復旧間隔です。
- 単位：分
- デフォルト値：5

#### spark_load_default_timeout_second

- 含义：Sparkインポートのタイムアウト時間です。
- 単位：秒
- デフォルト値：86400

#### spark_home_default_dir

- 含义：Sparkクライアントのルートディレクトリです。
- デフォルト値：`StarRocksFE.STARROCKS_HOME_DIR + "/lib/spark2x"`

#### stream_load_default_timeout_second

- 含义：Stream Loadのデフォルトのタイムアウト時間です。
- 単位：秒
- デフォルト値：600

#### max_stream_load_timeout_second

- 含义：Stream Loadの最大タイムアウト時間です。
- 単位：秒
- デフォルト値：259200

#### insert_load_default_timeout_second

- 含义：Insert Intoステートメントのタイムアウト時間です。
- 単位：秒
- デフォルト値：3600

#### broker_load_default_timeout_second

- 含义：Broker Loadのタイムアウト時間です。
- 単位：秒
- デフォルト値：14400

#### min_bytes_per_broker_scanner

- 含义：1つのBroker Loadタスクの最大並行インスタンス数です。
- 単位：バイト
- デフォルト値：67108864

#### max_broker_concurrency

- 含义：1つのBroker Loadタスクの最大並行インスタンス数です。3.1バージョン以降、StarRocksはこのパラメータをサポートしていません。
- デフォルト値：100

#### export_max_bytes_per_be_per_task

- 含义：1つのエクスポートタスクで1つのBEからエクスポートできる最大データ量です。
- 単位：バイト
- デフォルト値：268435456

#### export_running_job_num_limit

- 含义：エクスポートジョブの最大実行数です。
- デフォルト値：5

#### export_task_default_timeout_second

- 含义：エクスポートジョブのタイムアウト時間です。
- 単位：秒
- デフォルト値：7200

#### empty_load_as_error

- 含义：インポートデータが空の場合、エラーメッセージ `all partitions have no load data` を返すかどうかを指定します。<br />- **TRUE**：インポートデータが空の場合、インポートに失敗し、エラーメッセージ `all partitions have no load data` を返します。<br />- **FALSE**：インポートデータが空の場合、インポートに成功し、`OK` を返しますが、エラーメッセージは返しません。
- デフォルト値：TRUE

#### enable_sync_publish

- 含义：インポートトランザクションのパブリッシュフェーズでapplyタスクを同期的に実行するかどうかを指定します。主キーモデルテーブルにのみ適用されます。<br />- `TRUE`（デフォルト）：インポートトランザクションのパブリッシュフェーズでapplyタスクを同期的に実行し、applyタスクが完了するとインポートトランザクションのパブリッシュが成功し、インポートデータが実際に参照可能になります。したがって、一度に大量のデータをインポートする場合や、インポートの頻度が高い場合は、このパラメータをオンにするとクエリのパフォーマンスと安定性が向上しますが、インポートにかかる時間が増えます。<br />- `FALSE`：インポートトランザクションのパブリッシュフェーズでapplyタスクを非同期に実行し、インポートトランザクションのパブリッシュが成功した後すぐに返しますが、この時点ではインポートデータは実際には参照できません。そのため、並行するクエリはapplyタスクが完了するかタイムアウトするまで待機する必要があります。したがって、一度に大量のデータをインポートする場合や、インポートの頻度が高い場合は、このパラメータをオフにするとクエリのパフォーマンスと安定性に影響します。
- デフォルト値：TRUE
- 導入バージョン：v3.2.0

#### external_table_commit_timeout_ms

- 含义：StarRocks外部テーブルへの書き込みトランザクションのタイムアウト時間です。単位はミリ秒で、デフォルト値 `10000` はタイムアウト時間が10秒であることを示します。
- 単位：ミリ秒
- デフォルト値：10000

### ストレージ

#### default_replication_num

- 含义：パーティションのデフォルトのレプリケーション数を設定します。テーブル作成時に `replication_num` プロパティが指定されている場合、このプロパティが優先されます。テーブル作成時に `replication_num` が指定されていない場合、`default_replication_num` の設定が適用されます。このパラメータの値はクラスタ内のBEノード数を超えないようにすることをお勧めします。
- デフォルト値：3

#### enable_strict_storage_medium_check

- 含义：テーブル作成時にストレージメディアのタイプを厳密にチェックするかどうかを指定します。<br />trueの場合、テーブル作成時にBE上のストレージメディアを厳密にチェックします。たとえば、テーブル作成時に `storage_medium = HDD` と指定した場合、BE上でSSDのみが設定されている場合、テーブル作成は失敗します。<br />FALSEの場合、メディアの一致を無視してテーブル作成が成功します。
- デフォルト値：FALSE

#### enable_auto_tablet_distribution

- 含义：自動的にバケット数を設定する機能を有効にするかどうかを指定します。<ul><li>trueに設定すると、テーブル作成やパーティションの追加時にバケット数を指定する必要がなくなります。StarRocksが自動的にバケット数を決定します。バケット数の自動設定の戦略については、[バケット数の決定](../table_design/Data_distribution.md#バケット数の決定)を参照してください。</li><li>falseに設定すると、テーブル作成時にバケット数を手動で指定する必要があります。<br />パーティションの追加時にバケット数を指定しない場合、新しいパーティションのバケット数はテーブル作成時のバケット数を継承します。もちろん、新しいパーティションのバケット数を手動で指定することもできます。</li></ul>
- デフォルト値：TRUE
- 導入バージョン：2.5.6

#### storage_usage_soft_limit_percent

- 含义：BEのストレージディレクトリの使用率がこの値を超え、かつ残りの空き容量が `storage_usage_soft_limit_reserve_bytes` より小さい場合、そのパスに対してクローンを続行できません。
- デフォルト値：90

#### storage_usage_soft_limit_reserve_bytes

- 含义：デフォルトは200GBで、バイト単位です。BEのストレージディレクトリの残りの空き容量がこの値より小さく、かつ使用率が `storage_usage_soft_limit_percent` を超える場合、そのパスに対してクローンを続行できません。
- 単位：バイト
- デフォルト値：200 \* 1024 \* 1024 \* 1024

#### catalog_trash_expire_second


```
- 含义：表やデータベースを削除した後、メタデータがゴミ箱に保持される期間です。この期間を超えると、データは復元できません。
- 単位：秒
- デフォルト値：86400

#### alter_table_timeout_second

- 含义：Schema changeのタイムアウト時間です。
- 単位：秒
- デフォルト値：86400

#### enable_fast_schema_evolution

- 含义：クラスタ内のすべてのテーブルの高速スキーマ進化を有効にするかどうかを示します。値は`TRUE`または`FALSE`（デフォルト）です。有効にすると、列の追加や削除時のスキーマ変更の速度が向上し、リソースの使用量が減少します。
  > **注意**
  >
  > - StarRocksのストレージと計算の分離クラスタはこのパラメータをサポートしていません。
  > - 特定のテーブルにこの設定を適用する必要がある場合は、テーブル作成時に [`fast_schema_evolution`](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#setting-fast-schema-evolution) テーブルプロパティを設定できます。
- デフォルト値：FALSE
- 導入バージョン：3.2.0

#### recover_with_empty_tablet

- 含义：タブレットのレプリカが失われた/破損した場合、空のタブレットを使用するかどうかを示します。<br />これにより、タブレットのレプリカが失われた/破損した場合でも、クエリは実行されます（ただし、データが欠落しているため、結果は正しくありません）。デフォルトはfalseで、置換は行われず、クエリは失敗します。
- デフォルト値：FALSE

#### tablet_create_timeout_second

- 含义：タブレットの作成のタイムアウト時間です。バージョン3.1以降、デフォルト値は1から10に変更されました。
- 単位：秒
- デフォルト値：10

#### tablet_delete_timeout_second

- 含义：タブレットの削除のタイムアウト時間です。
- 単位：秒
- デフォルト値：2

#### check_consistency_default_timeout_second

- 含义：レプリカの整合性チェックのタイムアウト時間です。
- 単位：秒
- デフォルト値：600

#### tablet_sched_slot_num_per_path

- 含义：1つのBEストレージディレクトリで同時に実行できるタブレット関連のタスクの数です。パラメータの別名は `schedule_slot_num_per_path` です。バージョン2.5以降、このパラメータのデフォルト値は2.4の`4`から`8`に変更されました。
- デフォルト値：8

#### tablet_sched_max_scheduling_tablets

- 含义：同時にスケジュールできるタブレットの数です。スケジュール中のタブレットの数がこの値を超える場合、タブレットのバランス調整と修復チェックはスキップされます。
- デフォルト値：10000

#### tablet_sched_disable_balance

- 含义：タブレットのバランススケジュールを無効にするかどうかを示します。パラメータの別名は `disable_balance` です。
- デフォルト値：FALSE

#### tablet_sched_disable_colocate_balance

- 含义：Colocateテーブルのレプリカバランスを無効にするかどうかを示します。パラメータの別名は `disable_colocate_balance` です。
- デフォルト値：FALSE

#### tablet_sched_max_balancing_tablets

- 含义：バランス調整中のタブレットの最大数です。バランス調整中のタブレットの数がこの値を超える場合、タブレットの再バランスはスキップされます。パラメータの別名は `max_balancing_tablets` です。
- デフォルト値：500

#### tablet_sched_balance_load_disk_safe_threshold

- 含义：BEのディスク使用率が均等かどうかを判断するための閾値です。`tablet_sched_balancer_strategy` が `disk_and_tablet` に設定されている場合にのみ、このパラメータが有効になります。<br />すべてのBEのディスク使用率が50%未満の場合、ディスク使用が均等と見なされます。<br />disk_and_tablet戦略の場合、最大と最小のBEのディスク使用率の差が10%を超える場合、ディスク使用が均等でないと見なされ、タブレットの再バランスがトリガされます。パラメータの別名は `balance_load_disk_safe_threshold` です。
- デフォルト値：0.5

#### tablet_sched_balance_load_score_threshold

- 含义：BEの負荷が均等かどうかを判断するための閾値です。`tablet_sched_balancer_strategy` が `be_load_score` に設定されている場合にのみ、このパラメータが有効になります。<br />平均負荷から10%低い負荷のBEは低負荷状態、平均負荷から10%高い負荷のBEは高負荷状態です。パラメータの別名は `balance_load_score_threshold` です。
- デフォルト値：0.1

#### tablet_sched_repair_delay_factor_second

- 含义：FEでレプリカ修復を実行する間隔です。パラメータの別名は `tablet_repair_delay_factor_second` です。
- 単位：秒
- デフォルト値：60

#### tablet_sched_min_clone_task_timeout_sec

- 含义：タブレットのクローンの最小タイムアウト時間です。
- 単位：秒
- デフォルト値：3 \* 60

#### tablet_sched_max_clone_task_timeout_sec

- 含义：タブレットのクローンの最大タイムアウト時間です。パラメータの別名は `max_clone_task_timeout_sec` です。
- 単位：秒
- デフォルト値：2 \* 60 \* 60

#### tablet_sched_max_not_being_scheduled_interval_ms

- 含义：タブレットのスケジュールが行われない場合、そのタブレットのスケジュール優先度を高くし、できるだけ優先的にスケジュールする時間です。
- 単位：ミリ秒
- デフォルト値：15 \* 60 \* 100

### 存算分離関連の動的パラメータ

#### lake_compaction_score_selector_min_score

- 含义：Compaction操作をトリガするCompaction Scoreの閾値です。テーブルのパーティションのCompaction Scoreがこの値以上の場合、システムはそのパーティションに対してCompaction操作を実行します。
- デフォルト値：10.0
- 導入バージョン：v3.1.0

Compaction Scoreは、テーブルのパーティションにCompactionを実行する価値があるかどうかを示すスコアです。パーティション内のファイルの数と関連があります。ファイルの数が多いとクエリのパフォーマンスに影響を与えるため、システムは定期的にCompaction操作を実行して小さなファイルをマージし、ファイルの数を減らします。

#### lake_compaction_max_tasks

- 含义：同時に実行できるCompactionタスクの数です。
- デフォルト値：-1
- 導入バージョン：v3.1.0

システムは、パーティション内のタブレットの数に基づいてCompactionタスクの数を計算します。パーティションに10個のタブレットがある場合、そのパーティションに対して1回のCompactionを行うと、10個のCompactionタスクが作成されます。実行中のCompactionタスク数がこの閾値を超える場合、システムは新しいCompactionタスクを作成しません。値を`0`に設定するとCompactionが禁止され、`-1`に設定するとシステムは自動的に値を計算します。

#### lake_compaction_history_size

- 含义：リーダーFEノードのメモリに保持される最新の成功したCompactionタスクの履歴の数です。`SHOW PROC '/compactions'`コマンドを使用して最近の成功したCompactionタスクの履歴を表示できます。Compactionの履歴はFEプロセスのメモリに保存され、FEプロセスが再起動すると履歴が失われます。
- デフォルト値：12
- 導入バージョン：v3.1.0

#### lake_compaction_fail_history_size

- 含义：リーダーFEノードのメモリに保持される最新の失敗したCompactionタスクの履歴の数です。`SHOW PROC '/compactions'`コマンドを使用して最近の失敗したCompactionタスクの履歴を表示できます。Compactionの履歴はFEプロセスのメモリに保存され、FEプロセスが再起動すると履歴が失われます。
- デフォルト値：12
- 導入バージョン：v3.1.0

#### lake_publish_version_max_threads

- 含义：有効なバージョン（Publish Version）タスクの最大スレッド数です。
- デフォルト値：512
- 導入バージョン：v3.2.0

#### lake_autovacuum_parallel_partitions

- 含义：同時にゴミデータクリーニング（AutoVacuum）を実行できるテーブルパーティションの最大数です。AutoVacuumはCompactionの後に実行されるゴミファイルの回収です。
- デフォルト値：8
- 導入バージョン：v3.1.0

#### lake_autovacuum_partition_naptime_seconds

- 含义：同じテーブルパーティションのゴミデータクリーニングの最小間隔時間です。
- 単位：秒
- デフォルト値：180
- 導入バージョン：v3.1.0

#### lake_autovacuum_grace_period_minutes

- 含义：履歴データバージョンを保持する期間です。この期間内の履歴データバージョンは自動的にクリーンアップされません。データへのアクセス中にデータが削除されてクエリが失敗しないように、この値を最大クエリ時間より大きく設定する必要があります。
- 単位：分
- デフォルト値：5
- 導入バージョン：v3.1.0

#### lake_autovacuum_stale_partition_threshold

- 含义：一定期間（更新、削除、またはCompactionなど）に更新操作がない場合、そのパーティションの自動ゴミデータクリーニングは実行されなくなります。
- 単位：時間
- デフォルト値：12
- 導入バージョン：v3.1.0

#### lake_enable_ingest_slowdown

- 含义：インポートのスローダウン機能を有効にするかどうかを示します。インポートのスローダウン機能を有効にすると、Compaction Scoreが `lake_ingest_slowdown_threshold` を超えるテーブルパーティションのインポートタスクが制限されます。
- デフォルト値：false
- 導入バージョン：v3.2.0

#### lake_ingest_slowdown_threshold

- 含义：インポートのスローダウンをトリガするCompaction Scoreの閾値です。`lake_enable_ingest_slowdown` が `true` に設定されている場合にのみ、この設定が有効になります。
- デフォルト値：100
- 導入バージョン：v3.2.0

> **注意**
>
> `lake_ingest_slowdown_threshold` が `lake_compaction_score_selector_min_score` よりも小さい場合、実際の閾値は `lake_compaction_score_selector_min_score` になります。

#### lake_ingest_slowdown_ratio

- 含义：インポートのスローダウン比率です。
- デフォルト値：0.1
- 導入バージョン：v3.2.0

データのインポートタスクは、データの書き込みとデータのコミット（COMMIT）の2つの段階に分けることができます。インポートのスローダウンは、データのコミットを遅延させることで制限をかける仕組みです。遅延の割合は次の式で計算されます：`(compaction_score - lake_ingest_slowdown_threshold) * lake_ingest_slowdown_ratio`。例えば、データの書き込みに5分かかり、`lake_ingest_slowdown_ratio` が0.1で、Compaction Scoreが `lake_ingest_slowdown_threshold` よりも10多い場合、遅延する時間は `5 * 10 * 0.1 = 5` 分になります。つまり、書き込み段階の時間が5分から10分に増え、平均的なインポート速度が半分になります。

> **注意**
>
> - インポートタスクが複数のパーティションに同時に書き込む場合、遅延の時間はすべてのパーティションのCompaction Scoreの最大値を使用して計算されます。
> - 遅延の時間は最初のコミットの試行時に計算され、一度確定すると変更されません。遅延時間が経過すると、Compaction Scoreが `lake_compaction_score_upper_bound` を超えない限り、システムはデータのコミット（COMMIT）操作を実行します。
> - 遅延後のコミット時間がインポートタスクのタイムアウト時間を超える場合、インポートタスクは直ちに失敗します。

#### lake_compaction_score_upper_bound

- 含义：テーブルパーティションのCompaction Scoreの上限です。`lake_enable_ingest_slowdown` が `true` に設定されている場合にのみ、この設定が有効になります。テーブルパーティションのCompaction Scoreがこの上限に達するか超えると、そのパーティションに関連するすべてのインポートタスクは無期限にコミットが遅延され、Compaction Scoreがこの値以下になるかタスクがタイムアウトするまで実行されません。
- デフォルト値：0
- 導入バージョン：v3.2.0

### その他の動的パラメータ

#### plugin_enable

- 含义：プラグイン機能が有効かどうかを示します。リーダーFEでのみプラグインのインストール/アンインストールができます。
- デフォルト値：TRUE

#### max_small_file_number

- 含义：保存できる小さなファイルの最大数です。
- 默认值：100

#### max_small_file_size_bytes

- 含义：存储文件的大小上限。
- 单位：字节
- 默认值：1024 \* 1024

#### agent_task_resend_wait_time_ms

- 含义：代理任务重新发送前的等待时间。当代理任务的创建时间已设置，并且距离现在超过该值时，才能重新发送代理任务。<br />该参数防止过于频繁的代理任务发送。
- 单位：毫秒
- 默认值：5000

#### backup_job_default_timeout_ms

- 含义：备份作业的超时时间
- 单位：毫秒
- 默认值：86400*1000

#### enable_experimental_mv

- 含义：是否开启异步物化视图功能。`TRUE` 表示开启。从 2.5.2 版本开始，该功能默认开启。2.5.2 版本之前默认值为 `FALSE`。
- 默认值：TRUE

#### authentication_ldap_simple_bind_base_dn

- 含义：检索用户时使用的 Base DN，用于指定 LDAP 服务器检索用户鉴权信息的起始点。
- 默认值：空字符串

#### authentication_ldap_simple_bind_root_dn

- 含义：检索用户时使用的管理员账号的 DN。
- 默认值：空字符串

#### authentication_ldap_simple_bind_root_pwd

- 含义：检索用户时使用的管理员账号的密码。
- 默认值：空字符串

#### authentication_ldap_simple_server_host

- 含义：LDAP 服务器所在主机的主机名。
- 默认值：空字符串

#### authentication_ldap_simple_server_port

- 含义：LDAP 服务器的端口。
- 默认值：389

#### authentication_ldap_simple_user_search_attr

- 含义：LDAP 对象中标识用户的属性名称。
- 默认值：uid

#### max_upload_task_per_be

- 含义：单次备份操作下，系统向单个 BE 节点下发的最大上传任务数。设置为小于或等于 0 时表示不限制任务数。该参数自 v3.1.0 起新增。
- 默认值：0

#### max_download_task_per_be

- 含义：单次恢复操作下，系统向单个 BE 节点下发的最大下载任务数。设置为小于或等于 0 时表示不限制任务数。该参数自 v3.1.0 起新增。
- 默认值：0

#### allow_system_reserved_names

- 含义：是否允许用户创建以 `__op` 或 `__row` 开头命名的列。TRUE 表示启用此功能。请注意，在 StarRocks 中，这样的列名被保留用于特殊目的，创建这样的列可能导致未知行为，因此系统默认禁止使用这类名字。该参数自 v3.2.0 起新增。
- 默认值：FALSE

#### enable_backup_materialized_view

- 含义：在数据库的备份操作中，是否对数据库中的异步物化视图进行备份。如果设置为 `false`，将跳过对异步物化视图的备份。该参数自 v3.2.0 起新增。
- 默认值：TRUE

#### enable_colocate_mv_index

- 含义：在创建同步物化视图时，是否将同步物化视图的索引与基表加入到相同的 Colocate Group。如果设置为 `true`，TabletSink 将加速同步物化视图的写入性能。该参数自 v3.2.0 起新增。
- 默认值：TRUE

#### enable_mv_automatic_active_check

- 含义：是否允许系统自动检查和重新激活异步物化视图。启用此功能后，系统将会自动激活因基表（或视图）Schema Change 或重建而失效（Inactive）的物化视图。请注意，此功能不会激活由用户手动设置为 Inactive 的物化视图。此项功能支持从 v3.1.6 版本开始。
- 默认值：TRUE

## 配置 FE 静态参数

以下 FE 配置项为静态参数，不支持在线修改，您需要在 `fe.conf` 中修改并重启 FE。

本节对 FE 静态参数做了如下分类：

- [ログ](#log-fe-静态)
- [サーバー](#server-fe-静态)
- [メタデータとクラスタ管理](#元数据与集群管理fe-静态)
- [クエリ エンジン](#query-enginefe-静态)
- [インポートとエクスポート](#导入和导出fe-静态)
- [ストレージ](#存储fe-静态)
- [その他の動的パラメータ](#其他动态参数)

### ログ (FE 静态)

#### log_roll_size_mb

- 含义：ログファイルのサイズ。
- 単位：MB
- デフォルト値：1024（1 GB のログファイルサイズを示す）

#### sys_log_dir

- 含义：システムログファイルの保存ディレクトリ。
- デフォルト値：`StarRocksFE.STARROCKS_HOME_DIR + "/log"`

#### sys_log_level

- 含义：システムログのレベル。低から高に順に `INFO`、`WARN`、`ERROR`、`FATAL`。
- デフォルト値：INFO

#### sys_log_verbose_modules

- 含义：システムログの表示モジュール。パラメータの値が `org.apache.starrocks.catalog` の場合、Catalog モジュールのログのみ表示されます。
- デフォルト値：空文字列

#### sys_log_roll_interval

- 含义：システムログのローテーション間隔。`DAY` または `HOUR` の値を取ります。<br />`DAY` の場合、ログファイル名のサフィックスは `yyyyMMdd` になります。`HOUR` の場合、ログファイル名のサフィックスは `yyyyMMddHH` になります。
- デフォルト値：DAY

#### sys_log_delete_age

- 含义：システムログファイルの保持期間。デフォルト値 `7d` は、システムログファイルを 7 日間保持し、7 日を超えるログファイルは削除されます。
- デフォルト値：`7d`

#### sys_log_roll_num

- 含义：`sys_log_roll_interval` の時間範囲内で保持できるシステムログファイルの最大数。
- デフォルト値：10

#### audit_log_dir

- 含义：監査ログファイルの保存ディレクトリ。
- デフォルト値：`StarRocksFE.STARROCKS_HOME_DIR + "/log"`

#### audit_log_roll_num

- 含义：`audit_log_roll_interval` の時間範囲内で保持できる監査ログファイルの最大数。
- デフォルト値：90

#### audit_log_modules

- 含义：監査ログを出力するモジュール。デフォルトでは slow_query と query モジュールのログが出力されます。複数のモジュールを指定する場合は、モジュール名を英語のカンマとスペースで区切ってください。
- デフォルト値：slow_query, query

#### audit_log_roll_interval

- 含义：監査ログのローテーション間隔。`DAY` または `HOUR` の値を取ります。<br />`DAY` の場合、ログファイル名のサフィックスは `yyyyMMdd` になります。`HOUR` の場合、ログファイル名のサフィックスは `yyyyMMddHH` になります。
- デフォルト値：DAY

#### audit_log_delete_age

- 含义：監査ログファイルの保持期間。デフォルト値 `30d` は、監査ログファイルを 30 日間保持し、30 日を超えるログファイルは削除されます。
- デフォルト値：`30d`

#### dump_log_dir

- 含义：ダンプログファイルの保存ディレクトリ。
- デフォルト値：`StarRocksFE.STARROCKS_HOME_DIR + "/log"`

#### dump_log_modules

- 含义：ダンプログを出力するモジュール。デフォルトでは query モジュールのログが出力されます。複数のモジュールを指定する場合は、モジュール名を英語のカンマとスペースで区切ってください。
- デフォルト値：query

#### dump_log_roll_interval

- 含义：ダンプログのローテーション間隔。`DAY` または `HOUR` の値を取ります。<br />`DAY` の場合、ログファイル名のサフィックスは `yyyyMMdd` になります。`HOUR` の場合、ログファイル名のサフィックスは `yyyyMMddHH` になります。
- デフォルト値：DAY

#### dump_log_roll_num

- 含义：`dump_log_roll_interval` の時間範囲内で保持できるダンプログファイルの最大数。
- デフォルト値：10

#### dump_log_delete_age

- 含义：ダンプログファイルの保持期間。デフォルト値 `7d` は、ダンプログファイルを 7 日間保持し、7 日を超えるログファイルは削除されます。
- デフォルト値：`7d`

### サーバー (FE 静态)

#### frontend_address

- 含义：FE ノードの IP アドレス。
- デフォルト値：0.0.0.0

#### priority_networks

- 含义：複数の IP アドレスを持つサーバーに対して選択ポリシーを宣言します。 <br />注意：このリストに一致する IP アドレスは最大1つまでです。CIDR 表記法で指定する必要があります。一致する IP がない場合はランダムに選択されます。
- デフォルト値：空文字列

#### http_port

- 含义：FE ノード上の HTTP サーバーのポート番号。
- デフォルト値：8030

#### http_backlog_num

- 含义：HTTP サーバーがサポートする Backlog キューの長さ。
- デフォルト値：1024

#### cluster_name

- 含义：FE が所属する StarRocks クラスタの名前。ウェブページのタイトルとして表示されます。
- デフォルト値：StarRocks Cluster

#### rpc_port

- 含义：FE ノード上の Thrift サーバーのポート番号。
- デフォルト値：9020

#### thrift_backlog_num

- 含义：Thrift サーバーがサポートする Backlog キューの長さ。
- デフォルト値：1024

#### thrift_server_max_worker_threads

- 含义：Thrift サーバーがサポートする最大ワーカースレッド数。
- デフォルト値：4096

#### thrift_client_timeout_ms

- 含义：Thrift クライアントのアイドルタイムアウト時間。指定した時間内に新しいリクエストがない場合、接続が切断されます。
- 単位：ミリ秒
- デフォルト値：5000

#### thrift_server_queue_size

- 含义：Thrift サーバーのペンディングキューの長さ。現在の処理スレッド数が `thrift_server_max_worker_threads` の値を超える場合、余分なスレッドはペンディングキューに追加されます。
- デフォルト値：4096

#### brpc_idle_wait_max_time

- 含义：bRPC のアイドル待機時間。単位：ミリ秒。
- デフォルト値：10000

#### query_port

- 含义：FE ノード上の MySQL サーバーのポート番号。
- デフォルト値：9030

#### mysql_service_nio_enabled  

- 含义：MySQL サーバーの非同期 I/O オプションを有効にするかどうか。
- デフォルト値：TRUE

#### mysql_service_io_threads_num

- 含义：I/O イベントを処理するための最大スレッド数。
- デフォルト値：4

#### mysql_nio_backlog_num

- 含义：MySQL サーバーがサポートする Backlog キューの長さ。
- デフォルト値：1024

#### max_mysql_service_task_threads_num

- 含义：タスクを処理するための最大スレッド数。
- デフォルト値：4096

#### mysql_server_version

- 含义：MySQL サーバーのバージョン。このパラメータを変更すると、以下のシナリオで返されるバージョンが変更されます：1. `select version();` 2. ハンドシェイクパケットのバージョン 3. グローバル変数 `version` の値 (`show variables like 'version';`)
- デフォルト値：5.1.0

#### max_connection_scheduler_threads_num

- 含义：接続スケジューラがサポートする最大スレッド数。
- デフォルト値：4096

#### qe_max_connection

- 含义：すべてのユーザーによって発行された接続を含む、FE がサポートする最大接続数。
- デフォルト値：1024

#### check_java_version


- 含义: 检查已编译的 Java 版本与运行的 Java 版本是否兼容。<br />如果不兼容，则上报 Java 版本不匹配的异常信息，并终止启动。
- 默认值：TRUE

### 元数据与集群管理（FE 静态）

#### meta_dir

- 含义：元数据的保存目录。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/meta"`

#### heartbeat_mgr_threads_num

- 含义：Heartbeat Manager 中用于发送心跳任务的最大线程数。
- 默认值：8

#### heartbeat_mgr_blocking_queue_size

- 含义：Heartbeat Manager 中存储心跳任务的阻塞队列大小。
- 默认值：1024

#### metadata_failure_recovery

- 含义：是否强制重置 FE 的元数据。请谨慎使用该配置项。
- 默认值：FALSE

#### edit_log_port

- 含义：FE 所在 StarRocks 集群中各 Leader FE、Follower FE、Observer FE 之间通信用的端口。
- 默认值：9010

#### edit_log_type

- 含义：编辑日志的类型。取值只能为 `BDB`。
- 默认值：BDB

#### bdbje_heartbeat_timeout_second

- 含义：FE 所在 StarRocks 集群中 Leader FE 和 Follower FE 之间的 BDB JE 心跳超时时间。
- 单位：秒。
- 默认值：30

#### bdbje_lock_timeout_second

- 含义：BDB JE 操作的锁超时时间。
- 单位：秒。
- 默认值：1

#### max_bdbje_clock_delta_ms

- 含义：FE 所在 StarRocks 集群中 Leader FE 与非 Leader FE 之间能够容忍的最大时钟偏移。
- 单位：毫秒。
- 默认值：5000

#### txn_rollback_limit

- 含义：允许回滚的最大事务数。
- 默认值：100

#### bdbje_replica_ack_timeout_second  

- 含义：FE 所在 StarRocks 集群中，元数据从 Leader FE 写入到多个 Follower FE 时，Leader FE 等待足够多的 Follower FE 发送 ACK 消息的超时时间。当写入的元数据较多时，可能返回 ACK 的时间较长，进而导致等待超时。如果超时，会导致写元数据失败，FE 进程退出，此时可以适当地调大该参数取值。
- 单位：秒
- 默认值：10

#### master_sync_policy

- 含义：FE 所在 StarRocks 集群中，Leader FE 上的日志刷盘方式。该参数仅在当前 FE 为 Leader 时有效。取值范围：<ul><li>`SYNC`：事务提交时同步写日志并刷盘。</li><li> `NO_SYNC`：事务提交时不同步写日志。</li><li> `WRITE_NO_SYNC`：事务提交时同步写日志，但是不刷盘。</li></ul>如果您只部署了一个 Follower FE，建议将其设置为 `SYNC`。如果您部署了 3 个及以上 Follower FE，建议将其与下面的 `replica_sync_policy` 均设置为 `WRITE_NO_SYNC`。
- 默认值：SYNC

#### replica_sync_policy

- 含义：FE 所在 StarRocks 集群中，Follower FE 上的日志刷盘方式。取值范围：<ul><li>`SYNC`：事务提交时同步写日志并刷盘。</li><li> `NO_SYNC`：事务提交时不同步写日志。</li><li> `WRITE_NO_SYNC`：事务提交时同步写日志，但是不刷盘。</li></ul>
- 默认值：SYNC

#### replica_ack_policy

- 含义：判定日志是否有效的策略，默认是多数 Follower FE 返回确认消息，就认为生效。
- 默认值：SIMPLE_MAJORITY

#### cluster_id

- 含义：FE 所在 StarRocks 集群的 ID。具有相同集群 ID 的 FE 或 BE 属于同一个 StarRocks 集群。取值范围：正整数。- 默认值 `-1` 表示在 Leader FE 首次启动时随机生成一个。
- 默认值：-1

### クエリエンジン（FE 静态）

#### publish_version_interval_ms

- 含义：两个版本发布操作之间的时间间隔。
- 单位：ミリ秒
- 默认值：10

#### statistic_cache_columns

- 含义：缓存统计信息表的最大行数。
- 默认值：100000

### インポートとエクスポート（FE 静的）

#### load_checker_interval_second

- 含义：インポートジョブのポーリング間隔。
- 単位：秒。
- 默认值：5  

#### transaction_clean_interval_second

- 含义：終了したトランザクションのクリーンアップ間隔。クリーンアップ間隔を短くすることで、完了したトランザクションをタイムリーにクリーンアップできるようにします。
- 単位：秒。
- 默认值：30

#### label_clean_interval_second

- 含义：ジョブのラベルのクリーンアップ間隔。クリーンアップ間隔を短くすることで、過去のジョブのラベルをタイムリーにクリーンアップできるようにします。
- 単位：秒。
- 默认值：14400

#### spark_dpp_version

- 含义：Spark DPP のバージョン。
- 默认值：1.0.0

#### spark_resource_path

- 含义：Spark の依存パッケージのルートディレクトリ。
- 默认值：空の文字列

#### spark_launcher_log_dir

- 含义：Spark のログの保存ディレクトリ。
- 默认值：`sys_log_dir + "/spark_launcher_log"`

#### yarn_client_path

- 含义：Yarn クライアントのルートディレクトリ。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-client/hadoop/bin/yarn"`

#### yarn_config_dir

- 含义：Yarn の設定ファイルの保存ディレクトリ。
- 默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-config"`

#### export_checker_interval_second

- 含义：エクスポートジョブスケジューラのスケジュール間隔。
- 默认值：5

#### export_task_pool_size

- 含义：エクスポートタスクのスレッドプールのサイズ。
- 默认值：5

#### broker_client_timeout_ms

- 含义：Broker RPC のデフォルトのタイムアウト時間。
- 単位：ミリ秒
- 默认值：10s

### ストレージ（FE 静的）

#### tablet_sched_balancer_strategy

- 含义：Tablet のバランス戦略。パラメータの別名は `tablet_balancer_strategy`。取りうる値：`disk_and_tablet` および `be_load_score`。
- 默认值：`disk_and_tablet`

#### tablet_sched_storage_cooldown_second

- 含义：Table の作成時点から自動的に冷却されるまでの遅延時間。冷却とは、SSD メディアから HDD メディアへの移行を指します。<br />パラメータの別名は `storage_cooldown_second`。デフォルト値 `-1` は自動冷却を行わないことを示します。自動冷却機能を有効にするには、明示的にパラメータの値を 0 より大きく設定してください。
- 単位：秒。
- 默认值：-1

#### tablet_stat_update_interval_second

- 含义：FE が各 BE に対して Tablet の統計情報を要求する間隔。
- 単位：秒。
- 默认值：300

### StarRocks ストアアンドコンピューティング分離クラスタ

#### run_mode

- 含义：StarRocks クラスタの実行モード。有効な値：`shared_data` および `shared_nothing`（デフォルト）。`shared_data` は、ストアアンドコンピューティング分離モードで StarRocks を実行することを示します。`shared_nothing` は、ストアアンドコンピューティング一体モードで StarRocks を実行することを示します。<br />**注意**<br />StarRocks クラスタは、ストアアンドコンピューティング分離とストアアンドコンピューティング一体のモードを混在させることはサポートしていません。<br />クラスタのデプロイメントが完了した後に `run_mode` を変更しないでください。それを行うと、クラスタが再起動できなくなります。ストアアンドコンピューティング一体クラスタからストアアンドコンピューティング分離クラスタに変換することもサポートされていませんし、逆も同様です。
- 默认值：shared_nothing

#### cloud_native_meta_port

- 含义：クラウドネイティブメタデータサービスのリスニングポート。
- 默认值：6090

#### cloud_native_storage_type

- 含义：使用しているストレージのタイプ。ストアアンドコンピューティング分離モードでは、StarRocks は HDFS、Azure Blob（v3.1.1 以降）、および S3 互換のオブジェクトストレージ（AWS S3、Google GCP、Alibaba Cloud OSS、MinIO など）にデータを保存することができます。有効な値：`S3`（デフォルト）、`AZBLOB`、`HDFS`。`S3` を指定する場合は、`aws_s3` を接頭辞とする設定項目を追加する必要があります。`AZBLOB` を指定する場合は、`azure_blob` を接頭辞とする設定項目を追加する必要があります。`HDFS` を指定する場合は、`cloud_native_hdfs_url` のみを指定すればよいです。
- 默认值：S3

#### cloud_native_hdfs_url

- 含义：HDFS ストレージの URL。例：`hdfs://127.0.0.1:9000/user/xxx/starrocks/`。

#### aws_s3_path

- 含义：データを保存する S3 ストレージのパス。S3 バケットの名前とその下のサブパスで構成されます。例：`testbucket/subpath`。

#### aws_s3_region

- 含义：アクセスする S3 ストレージのリージョン。例：`us-west-2`。

#### aws_s3_endpoint

- 含义：S3 ストレージへの接続先のアドレス。例：`https://s3.us-west-2.amazonaws.com`。

#### aws_s3_use_aws_sdk_default_behavior

- 含义：AWS SDK のデフォルトの認証資格情報を使用するかどうか。有効な値：`true` および `false`（デフォルト）。
- 默认值：false

#### aws_s3_use_instance_profile

- 含义：インスタンスプロファイルまたはアサムドロールをセキュリティ資格情報として使用するかどうか。有効な値：`true` および `false`（デフォルト）。<ul><li>IAM ユーザー資格情報（アクセスキーとシークレットキー）を使用して S3 にアクセスする場合は、この項目を `false` に設定し、`aws_s3_access_key` と `aws_s3_secret_key` を指定する必要があります。</li><li>インスタンスプロファイルで S3 にアクセスする場合は、この項目を `true` に設定する必要があります。</li><li>アサムドロールで S3 にアクセスする場合は、この項目を `true` に設定し、`aws_s3_iam_role_arn` を指定する必要があります。</li><li>外部の AWS アカウントを使用してアサムドロール認証で S3 にアクセスする場合は、`aws_s3_external_id` を追加で指定する必要があります。</li></ul>
- 默认值：false

#### aws_s3_access_key

- 含义：S3 ストレージへのアクセスキー。

#### aws_s3_secret_key

- 含义：S3 ストレージへのシークレットキー。

#### aws_s3_iam_role_arn

- 含义：S3 ストレージへのアクセス権限を持つ IAM ロールの ARN。

#### aws_s3_external_id

- 含义：S3 ストレージへのアクセスのための外部 ID。

#### azure_blob_path

- 含义：データを保存する Azure Blob Storage のパス。ストレージアカウント内のコンテナ名とその下のサブパスで構成されます。例：`testcontainer/subpath`。

#### azure_blob_endpoint

- 含义：Azure Blob Storage のエンドポイント。例：`https://test.blob.core.windows.net`。

#### azure_blob_shared_key

- 含义：Azure Blob Storage へのアクセスキー。

#### azure_blob_sas_token

- 含义：Azure Blob Storage への共有アクセス署名（SAS）。
- 默认值：N/A

### その他の静的パラメータ

#### plugin_dir

- 含义：プラグインのインストールディレクトリ。
- 默认值：`STARROCKS_HOME_DIR/plugins`

#### small_file_dir

- 含义：小さなファイルのルートディレクトリ。
- デフォルト値：`StarRocksFE.STARROCKS_HOME_DIR + "/small_files"`

#### max_agent_task_threads_num

- 意味：エージェントタスクのスレッドプールで処理する最大スレッド数。
- デフォルト値：4096

#### auth_token

- 意味：内部認証用のクラスタートークン。空の場合、Leader FEが初回起動時にランダムに生成される。
- デフォルト値：空文字列

#### tmp_dir

- 意味：一時ファイルの保存ディレクトリ。例えば、バックアップやリストアプロセス中に生成される一時ファイル。<br />これらのプロセスが完了した後、生成された一時ファイルは削除される。
- デフォルト値：`StarRocksFE.STARROCKS_HOME_DIR + "/temp_dir"`

#### locale

- 意味：FEが使用する文字セット。
- デフォルト値：zh_CN.UTF-8

#### hive_meta_load_concurrency

- 意味：Hiveメタデータをサポートする最大並行スレッド数。
- デフォルト値：4

#### hive_meta_cache_refresh_interval_s

- 意味：Hive外部テーブルのメタデータキャッシュをリフレッシュする間隔。
- 単位：秒。
- デフォルト値：7200

#### hive_meta_cache_ttl_s

- 意味：Hive外部テーブルのメタデータキャッシュの有効期限。
- 単位：秒。
- デフォルト値：86400

#### hive_meta_store_timeout_s

- 意味：Hive Metastoreへの接続タイムアウト時間。
- 単位：秒。
- デフォルト値：10

#### es_state_sync_interval_second

- 意味：FEがElasticsearch Indexを取得し、StarRocks外部テーブルのメタデータを同期する間隔。
- 単位：秒。
- デフォルト値：10

#### enable_auth_check

- 意味：認証チェック機能を有効にするかどうか。選択肢：`TRUE` または `FALSE`。`TRUE` は機能を有効にすることを意味し、`FALSE` は機能を無効にすることを意味する。
- デフォルト値：TRUE

#### enable_metric_calculator

- 意味：定期的にメトリクスを収集する機能を有効にするかどうか。選択肢：`TRUE` または `FALSE`。`TRUE` は機能を有効にすることを意味し、`FALSE` は機能を無効にすることを意味する。
- デフォルト値：TRUE
