---
displayed_sidebar: Chinese
---

# BE 配置項目

一部の BE ノードの設定項目は動的パラメータであり、コマンドを通じてオンラインで変更できます。その他の設定項目は静的パラメータであり、**be.conf** ファイルを変更してから BE サービスを再起動し、変更を有効にする必要があります。

## BE 設定項目の確認

以下のコマンドで BE の設定項目を確認できます：

```shell
curl http://<BE_IP>:<BE_HTTP_PORT>/varz
```

## BE 動的パラメータの設定

`curl` コマンドを使用して、BE ノードの動的パラメータをオンラインで変更できます。

```shell
curl -XPOST http://be_host:http_port/api/update_config?configuration_item=value
```

以下は BE の動的パラメータリストです。

#### report_task_interval_seconds

- 意味：個々のタスクの報告間隔。テーブル作成、テーブル削除、インポート、schema change はすべてタスクとみなされます。
- 単位：秒
- デフォルト値：10

#### report_disk_state_interval_seconds

- 意味：ディスク状態の報告間隔。各ディスクの状態とそのデータ量などを報告します。
- 単位：秒
- デフォルト値：60

#### report_tablet_interval_seconds

- 意味：tablet の報告間隔。すべての tablet の最新バージョンを報告します。
- 単位：秒
- デフォルト値：60

#### report_workgroup_interval_seconds

- 意味：workgroup の報告間隔。すべての workgroup の最新バージョンを報告します。
- 単位：秒
- デフォルト値：5

#### max_download_speed_kbps

- 意味：単一の HTTP リクエストの最大ダウンロード速度。この値は BE 間でのデータレプリカの同期速度に影響します。
- 単位：KB/s
- デフォルト値：50000

#### download_low_speed_limit_kbps

- 意味：単一の HTTP リクエストのダウンロード速度の下限。`download_low_speed_time` 秒以内にダウンロード速度が `download_low_speed_limit_kbps` 未満であれば、リクエストは終了されます。
- 単位：KB/s
- デフォルト値：50

#### download_low_speed_time

- 意味：`download_low_speed_limit_kbps` を参照してください。
- 単位：秒
- デフォルト値：300

#### status_report_interval

- 意味：プロファイルの状態報告間隔。FE がクエリ統計情報を収集するために使用します。
- 単位：秒
- デフォルト値：5

#### scanner_thread_pool_thread_num

- 意味：ストレージエンジンがディスクを並行スキャンするスレッド数。スレッドプールで一元管理されます。
- デフォルト値：48

#### thrift_client_retry_interval_ms

- 意味：Thrift client のデフォルト再試行間隔。
- 単位：ミリ秒
- デフォルト値：100

#### scanner_thread_pool_queue_size

- 意味：ストレージエンジンがサポートするスキャンタスクの数。
- デフォルト値：102400

#### scanner_row_num

- 意味：各スキャンスレッドが一回の実行で返すことができる最大行数。
- デフォルト値：16384

#### max_scan_key_num

- 意味：クエリが最大で分割する scan key の数。
- デフォルト値：1024

#### max_pushdown_conditions_per_column

- 意味：単一の列で許可される最大のプッシュダウン述語の数。この数を超えると、述語はストレージ層にプッシュダウンされません。
- デフォルト値：1024

#### exchg_node_buffer_size_bytes  

- 意味：Exchange ノード内で、単一のクエリが受信側で持つことができるバッファの容量。<br />これはソフトリミットであり、送信データの速度が速すぎる場合、受信側はバックプレッシャーを発生させて送信速度を制限します。
- 単位：バイト
- デフォルト値：10485760

#### column_dictionary_key_ratio_threshold

- 意味：文字列型の値の比率がこの比率未満の場合、辞書圧縮アルゴリズムを使用します。
- 単位：%
- デフォルト値：0

#### memory_limitation_per_thread_for_schema_change

- 意味：単一の schema change タスクが使用できる最大メモリ。
- 単位：GB
- デフォルト値：2

#### update_cache_expire_sec

- 意味：Update Cache の有効期限。
- 単位：秒
- デフォルト値：360

#### file_descriptor_cache_clean_interval

- 意味：ファイルディスクリプタキャッシュのクリーンアップ間隔。長期間使用されていないファイルディスクリプタをクリーンアップするために使用されます。
- 単位：秒
- デフォルト値：3600

#### disk_stat_monitor_interval

- 意味：ディスクの健康状態をチェックする間隔。
- 単位：秒
- デフォルト値：5

#### unused_rowset_monitor_interval

- 意味：期限切れの Rowset をクリーンアップする間隔。
- 単位：秒
- デフォルト値：30

#### max_percentage_of_error_disk  

- 意味：エラーディスクの割合が一定の比率に達した場合、BE は終了します。
- 単位：%
- デフォルト値：0

#### default_num_rows_per_column_file_block

- 意味：各 row block に格納できる最大行数。
- デフォルト値：1024

#### pending_data_expire_time_sec  

- 意味：ストレージエンジンが保持する未適用データの最大期間。
- 単位：秒
- デフォルト値：1800

#### inc_rowset_expired_sec

- 意味：インポートされたデータが有効になった後、ストレージエンジンが保持する期間。インクリメンタルクローンに使用されます。
- 単位：秒
- デフォルト値：1800

#### tablet_rowset_stale_sweep_time_sec

- 意味：無効な rowset をクリーンアップする間隔。
- 単位：秒
- デフォルト値：1800

#### snapshot_expire_time_sec

- 意味：スナップショットファイルをクリーンアップする間隔。デフォルトは 48 時間です。
- 単位：秒
- デフォルト値：172800

#### trash_file_expire_time_sec

- 意味：ごみ箱をクリーンアップする間隔。デフォルトは 24 時間です。バージョン v2.5.17、v3.0.9、および v3.1.6 から、デフォルト値は 259,200 から 86,400 に変更されました。
- 単位：秒
- デフォルト値：86,400

#### base_compaction_check_interval_seconds

- 意味：Base Compaction スレッドのポーリング間隔。
- 単位：秒
- デフォルト値：60

#### min_base_compaction_num_singleton_deltas

- 意味：BaseCompaction をトリガーする最小のセグメント数。
- デフォルト値：5

#### max_base_compaction_num_singleton_deltas

- 意味：一回の BaseCompaction でマージする最大のセグメント数。
- デフォルト値：100

#### base_compaction_interval_seconds_since_last_operation

- 意味：前回の BaseCompaction からの間隔。BaseCompaction をトリガーする条件の一つです。
- 単位：秒
- デフォルト値：86400

#### cumulative_compaction_check_interval_seconds

- 意味：CumulativeCompaction スレッドのポーリング間隔。
- 単位：秒
- デフォルト値：1

#### update_compaction_check_interval_seconds

- 意味：Primary key モデルの Update compaction のチェック間隔。
- 単位：秒
- デフォルト値：60

#### min_compaction_failure_interval_sec

- 意味：Tablet Compaction が失敗した後、再スケジュールされるまでの間隔。
- 単位：秒
- デフォルト値：120

#### periodic_counter_update_period_ms

- 意味：Counter 統計情報の更新間隔。
- 単位：ミリ秒
- デフォルト値：500

#### load_error_log_reserve_hours  

- 意味：インポートデータ情報の保持期間。
- 単位：時間
- デフォルト値：48

#### streaming_load_max_mb

- 意味：ストリーミングインポートの単一ファイルサイズの上限。バージョン 3.0 から、デフォルト値は 10240 から 102400 に変更されました。
- 単位：MB
- デフォルト値：102400

#### streaming_load_max_batch_size_mb  

- 意味：ストリーミングインポートの単一 JSON ファイルサイズの上限。
- 単位：MB
- デフォルト値：100

#### memory_maintenance_sleep_time_s

- 意味：ColumnPool GC タスクをトリガーする間隔。StarRocks は定期的に GC タスクを実行し、空いているメモリをオペレーティングシステムに返そうとします。
- 単位：秒
- デフォルト値：10

#### write_buffer_size

- 意味：MemTable がメモリ内で持つことができるバッファのサイズ。この限界を超えると flush がトリガーされます。
- 単位：バイト
- デフォルト値：104857600

#### tablet_stat_cache_update_interval_second

- 意味：Tablet Stat Cache の更新間隔。
- 単位：秒
- デフォルト値：300

#### result_buffer_cancelled_interval_time

- 意味：BufferControlBlock がデータを解放するまでの待機時間。
- 単位：秒
- デフォルト値：300

#### thrift_rpc_timeout_ms

- 意味：Thrift のタイムアウト時間。
- 単位：ミリ秒
- デフォルト値：5000

#### max_consumer_num_per_group

- 意味：Routine load で、各 consumer group 内の最大 consumer 数。
- デフォルト値：3

#### max_memory_sink_batch_count

- 意味：Scan cache の最大キャッシュバッチ数。
- デフォルト値：20

#### scan_context_gc_interval_min

- 意味：Scan context のクリーンアップ間隔。
- 単位：分
- デフォルト値：5

#### path_gc_check_step

- 意味：一度に連続して scan する最大のファイル数。
- デフォルト値：1000

#### path_gc_check_step_interval_ms


- 意味：連続してファイルをスキャンする間隔時間。
- 単位：ミリ秒
- デフォルト値：10

#### path_scan_interval_second

- 意味：GCスレッドが期限切れデータをクリーンアップする間隔時間。
- 単位：秒
- デフォルト値：86400

#### storage_flood_stage_usage_percent

- 意味：空間使用率がこの値を超え、かつ残り空間が `storage_flood_stage_left_capacity_bytes` 未満の場合、Load および Restore ジョブを拒否します。
- 単位：%
- デフォルト値：95

#### storage_flood_stage_left_capacity_bytes

- 意味：残り空間がこの値未満で、かつ空間使用率が `storage_flood_stage_usage_percent` を超える場合、Load および Restore ジョブを拒否します。デフォルトは100GBです。
- 単位：バイト
- デフォルト値：107374182400

#### tablet_meta_checkpoint_min_new_rowsets_num  

- 意味：前回のTabletMeta Checkpointから新たに作成されたrowsetの数。
- デフォルト値：10

#### tablet_meta_checkpoint_min_interval_secs

- 意味：TabletMeta Checkpointスレッドのポーリング間隔。
- 単位：秒
- デフォルト値：600

#### max_runnings_transactions_per_txn_map

- 意味：各パーティション内で同時に実行できるトランザクションの最大数。
- デフォルト値：100

#### tablet_max_pending_versions

- 意味：Primary Keyテーブルの各タブレットで許可される、コミット済み（committed）だが未適用（apply）の最大バージョン数。
- デフォルト値：1000

#### tablet_max_versions

- 意味：各タブレットで許可される最大バージョン数。この値を超えると新しい書き込みリクエストが失敗します。
- デフォルト値：1000

#### alter_tablet_worker_count

- 意味：schema changeを行うスレッド数。バージョン2.5以降、このパラメータは静的から動的に変更されました。
- デフォルト値：3

#### max_hdfs_file_handle

- 意味：開くことができる最大のHDFSファイルハンドル数。
- デフォルト値：1000

#### be_exit_after_disk_write_hang_second

- 意味：ディスクがハングした後にBEプロセスが終了するまでの待機時間。
- 単位：秒
- デフォルト値：60

#### min_cumulative_compaction_failure_interval_sec

- 意味：Cumulative Compactionが失敗した後の最小リトライ間隔。
- 単位：秒
- デフォルト値：30

#### size_tiered_level_num

- 意味：Size-tiered Compaction戦略のレベル数。各レベルは最大で1つのrowsetを保持するため、安定状態ではレベル数と同じ数のrowsetが存在します。|
- デフォルト値：7

#### size_tiered_level_multiple

- 意味：Size-tiered Compaction戦略において、隣接する2つのレベル間のデータ量の倍数。
- デフォルト値：5

#### size_tiered_min_level_size

- 意味：Size-tiered Compaction戦略における最小レベルのサイズ。この値未満のrowsetは直接compactionをトリガーします。
- 単位：バイト
- デフォルト値：131072

#### storage_page_cache_limit

- 意味：PageCacheの容量。STRING形式で容量サイズを指定できます。例：`20G`、`20480M`、`20971520K`、または`21474836480B`。また、PageCacheがシステムメモリの何パーセントを占めるかを指定することもできます。例：`20%`。このパラメータは`disable_storage_page_cache`が`false`の場合にのみ有効です。
- デフォルト値：20%

#### disable_storage_page_cache

- 意味：PageCacheを有効にするかどうか。PageCacheを有効にすると、StarRocksは最近スキャンしたデータをキャッシュし、クエリの反復性が高いシナリオではクエリの効率を大幅に向上させます。`true`は無効を意味します。バージョン2.4以降、このパラメータのデフォルト値は`true`から`false`に変更されました。バージョン3.1以降、このパラメータは静的から動的に変更されました。
- デフォルト値：FALSE

#### max_compaction_concurrency

- 意味：Compactionスレッドの最大同時実行数（BaseCompactionとCumulativeCompactionの最大合計）。このパラメータはCompactionが過剰なメモリを消費するのを防ぎます。-1は制限なしを意味します。0はcompactionを許可しないことを意味します。
- デフォルト値：-1

#### internal_service_async_thread_num

- 意味：単一のBEでKafkaとのやり取りを行うスレッドプールのサイズ。現在のRoutine LoadはFEとKafkaの間のやり取りをBEを通じて行いますが、実際の操作はBEごとに独立したスレッドプールで行われます。Routine Loadのタスクが多い場合、スレッドプールがビジーになる可能性があり、この設定を調整することができます。
- デフォルト値：10

#### update_compaction_ratio_threshold

- 意味：分散ストレージクラスター下での主キーモデルテーブルの単一Compactionでマージできる最大データ比率。単一のTabletが大きすぎる場合は、この設定値を適切に小さくすることをお勧めします。バージョンv3.1.5からサポートされています。
- デフォルト値：0.5

#### max_garbage_sweep_interval

- 意味：ディスクがゴミをクリーンアップする最大間隔。バージョン3.0以降、このパラメータは静的から動的に変更されました。
- 単位：秒
- デフォルト値：3600

#### min_garbage_sweep_interval

- 意味：ディスクがゴミをクリーンアップする最小間隔。バージョン3.0以降、このパラメータは静的から動的に変更されました。
- 単位：秒
- デフォルト値：180

## BE静的パラメータの設定

以下のBE設定項目は静的パラメータであり、オンラインでの変更はサポートされていません。**be.conf**ファイルで変更し、BEサービスを再起動する必要があります。

#### hdfs_client_enable_hedged_read

- 意味：Hedged Read機能を有効にするかどうかを指定します。
- デフォルト値：false
- 導入バージョン：3.0

#### hdfs_client_hedged_read_threadpool_size  

- 意味：HDFSクライアント側でHedged Readをサービスするために許可されるスレッドの最大数を指定します。このパラメータはバージョン3.0からサポートされており、HDFSクラスタの設定ファイル`hdfs-site.xml`の`dfs.client.hedged.read.threadpool.size`パラメータに対応しています。
- デフォルト値：128
- 導入バージョン：3.0

#### hdfs_client_hedged_read_threshold_millis

- 意味：Hedged Readリクエストを発行する前に待機するミリ秒数を指定します。例えば、このパラメータが`30`に設定されている場合、Readタスクが30ミリ秒以内に結果を返さない場合、HDFSクライアントは直ちにHedged Readを発行し、データブロックのレプリカからデータを読み取ります。このパラメータはバージョン3.0からサポートされており、HDFSクラスタの設定ファイル`hdfs-site.xml`の`dfs.client.hedged.read.threshold.millis`パラメータに対応しています。
- 単位：ミリ秒
- デフォルト値：2500
- 導入バージョン：3.0

#### be_port

- 意味：FEからのリクエストを受け取るためのBE上のthrift serverのポート。
- デフォルト値：9060

#### brpc_port

- 意味：bRPCのポートで、bRPCのネットワーク統計情報を確認できます。
- デフォルト値：8060

#### brpc_num_threads

- 意味：bRPCのbthreadsスレッドの数。-1はCPUコア数と同じを意味します。
- デフォルト値：-1

#### priority_networks

- 意味：CIDR形式でBEのIPアドレスを指定します（例：10.10.10.0/24）。複数のIPがあるマシンで、優先して使用するネットワークを指定する場合に適用されます。
- デフォルト値：空文字列

#### starlet_port

- 意味：分散ストレージクラスター内のCN（バージョン3.0のBE）の追加のAgentサービスポート。
- デフォルト値：9070

#### heartbeat_service_port

- 意味：心拍数サービスポート（thrift）で、FEからの心拍数を受け取ります。
- デフォルト値：9050

#### heartbeat_service_thread_count

- 意味：心拍数スレッドの数。
- デフォルト値：1

#### create_tablet_worker_count

- 意味：タブレットを作成するスレッドの数。
- デフォルト値：3

#### drop_tablet_worker_count

- 意味：タブレットを削除するスレッドの数。
- デフォルト値：3

#### push_worker_count_normal_priority

- 意味：NORMAL優先度のタスクを処理するインポートスレッドの数。
- デフォルト値：3

#### push_worker_count_high_priority

- 意味：HIGH優先度のタスクを処理するインポートスレッドの数。
- デフォルト値：3

#### transaction_publish_version_worker_count

- 意味：バージョンを有効にする最大スレッド数。このパラメータが0以下に設定されている場合、システムはCPUコア数の半分をデフォルトで使用し、インポートの並行性が高い場合にスレッドリソースが不足することを避けます。バージョン2.5以降、デフォルト値は`8`から`0`に変更されました。
- デフォルト値：0

#### clear_transaction_task_worker_count

- 意味：トランザクションをクリアするスレッドの数。
- デフォルト値：1

#### clone_worker_count

- 意味：クローンのスレッド数。
- デフォルト値：3

#### storage_medium_migrate_count

- 意味：メディア移行のスレッド数、SATAからSSDへの移行。
- デフォルト値：1

#### check_consistency_worker_count

- 意味：タブレットのチェックサムを計算します。
- デフォルト値：1

#### sys_log_dir

- 意味：ログを保存する場所。INFO、WARNING、ERROR、FATALなどのログを含みます。
- デフォルト値：`${STARROCKS_HOME}/log`

#### user_function_dir

- 意味：UDFプログラムを保存するパス。
- デフォルト値：`${STARROCKS_HOME}/lib/udf`

#### small_file_dir

- 意味：ファイルマネージャーがダウンロードしたファイルを保存するディレクトリ。
- デフォルト値：`${STARROCKS_HOME}/lib/small_file`

#### sys_log_level

- 意味：ログレベル。INFO < WARNING < ERROR < FATAL。
- デフォルト値：INFO

#### sys_log_roll_mode

- 意味：ログの分割サイズ。1Gごとにログを分割します。
- デフォルト値：SIZE-MB-1024

#### sys_log_roll_num

- 意味：保持するログの数。
- デフォルト値：10

#### sys_log_verbose_modules

- 意味：ログ出力のモジュール。BEのnamespaceの有効な値には、`starrocks`、`starrocks::debug`、`starrocks::fs`、`starrocks::io`、`starrocks::lake`、`starrocks::pipeline`、`starrocks::query_cache`、`starrocks::stream`、`starrocks::workgroup`が含まれます。
- デフォルト値：空文字列

#### sys_log_verbose_level

- 意味：ログ表示レベル。コード内のVLOGで始まるログ出力を制御するために使用されます。
- デフォルト値：10

#### log_buffer_level

- 意味：ログのフラッシュ戦略。デフォルトではメモリ内に保持されます。
- デフォルト値：空文字列

#### num_threads_per_core

- 意味：CPUコアごとに起動するスレッド数。
- デフォルト値：3

#### compress_rowbatches

- 意味：BE間のRPC通信でRowBatchを圧縮するかどうか。クエリ層間のデータ転送に使用されます。
- デフォルト値：TRUE

#### serialize_batch

- 意味：BE間のRPC通信でRowBatchをシリアライズするかどうか。クエリ層間のデータ転送に使用されます。
- デフォルト値：FALSE

#### storage_root_path

- 意味：データを保存するディレクトリとストレージメディアのタイプ。複数のディスクを設定する場合はセミコロン `;` で区切ります。<br />SSDディスクの場合はパスの後に `,medium:ssd` を、HDDディスクの場合は `,medium:hdd` を追加します。例：`/data1,medium:hdd;/data2,medium:ssd`。
- デフォルト値：`${STARROCKS_HOME}/storage`

#### max_length_for_bitmap_function

- 意味：bitmap関数の入力値の最大長。
- 単位：バイト。
- デフォルト値：1000000

#### max_length_for_to_base64

- 意味：to_base64()関数の入力値の最大長。
- 単位：バイト。
- デフォルト値：200000

#### max_tablet_num_per_shard

- 意味：各shardのタブレット数。タブレットを分割し、単一のディレクトリにタブレットのサブディレクトリが多すぎるのを防ぐために使用されます。
- デフォルト値：1024

#### file_descriptor_cache_capacity

- 意味：ファイルディスクリプタキャッシュの容量。
- デフォルト値：16384
  
#### min_file_descriptor_number

- 意味：BEプロセスのファイルディスクリプタのlimit要求の下限。
- デフォルト値：60000

#### index_stream_cache_capacity

- 意味：BloomFilter/Min/Maxなどの統計情報キャッシュの容量。
- デフォルト値：10737418240

#### base_compaction_num_threads_per_disk

- 意味：ディスクごとのBaseCompactionスレッドの数。
- デフォルト値：1

#### base_cumulative_delta_ratio

- 意味：BaseCompactionのトリガー条件の一つ：CumulativeファイルのサイズがBaseファイルの比率に達した場合。
- デフォルト値：0.3

#### compaction_trace_threshold

- 意味：単一のCompactionのtraceを出力する時間閾値。単一のcompaction時間がこの閾値を超えた場合、traceが出力されます。
- 単位：秒。
- デフォルト値：60

#### be_http_port

- 意味：HTTPサーバーのポート。
- デフォルト値：8040

#### webserver_num_workers

- 意味：HTTPサーバーのスレッド数。
- デフォルト値：48

#### load_data_reserve_hours

- 意味：小規模なインポートで生成されたファイルの保持時間。
- デフォルト値：4

#### number_tablet_writer_threads

- 意味：ストリーミングインポートのスレッド数。
- デフォルト値：16

#### streaming_load_rpc_max_alive_time_sec

- 意味：ストリーミングインポートRPCのタイムアウト時間。
- デフォルト値：1200

#### fragment_pool_thread_num_min

- 意味：最小のクエリスレッド数。デフォルトで64スレッドが起動します。
- デフォルト値：64

#### fragment_pool_thread_num_max

- 意味：最大のクエリスレッド数。
- デフォルト値：4096

#### fragment_pool_queue_size

- 意味：単一ノードで処理できるクエリリクエストの上限。
- デフォルト値：2048

#### enable_token_check

- 意味：トークンの検証を有効にします。
- デフォルト値：TRUE

#### enable_prefetch

- 意味：クエリの事前フェッチを有効にします。
- デフォルト値：TRUE

#### load_process_max_memory_limit_bytes

- 意味：単一ノード上のすべてのインポートスレッドが占有するメモリの上限、100GB。
- 単位：バイト。
- デフォルト値：107374182400

#### load_process_max_memory_limit_percent

- 意味：単一ノード上のすべてのインポートスレッドが占有するメモリの上限の割合。
- デフォルト値：30

#### sync_tablet_meta

- 意味：ストレージエンジンがsyncを有効にしてディスクに保持するかどうか。
- デフォルト値：FALSE

#### routine_load_thread_pool_size

- 意味：単一ノード上のRoutine Loadスレッドプールのサイズ。バージョン3.1.0から、このパラメータは廃止されました。単一ノード上のRoutine Loadスレッドプールのサイズは、FEの動的パラメータ`max_routine_load_task_num_per_be`によって完全に制御されます。
- デフォルト値：10

#### brpc_max_body_size

- 意味：bRPCの最大パケット容量。
- 単位：バイト。
- デフォルト値：2147483648

#### tablet_map_shard_size

- 意味：タブレットの分割数。
- デフォルト値：32

#### enable_bitmap_union_disk_format_with_set

- 意味：Bitmapの新しいストレージフォーマットで、bitmap_unionのパフォーマンスを最適化できます。
- デフォルト値：FALSE

#### mem_limit

- 意味：BEプロセスのメモリ上限。割合の上限（例："80%"）または物理的な上限（例："100GB"）として設定できます。
- デフォルト値：90%

#### flush_thread_num_per_store

- 意味：各StoreがMemTableをFlushするためのスレッド数。
- デフォルト値：2

#### datacache_enable

- 意味：Data Cacheを有効にするかどうか。<ul><li>`true`：有効にする。</li><li>`false`：有効にしない、デフォルト値。</li></ul> 有効にする場合は、このパラメータの値を`true`に設定します。
- デフォルト値：false

#### datacache_disk_path  

- 意味：ディスクパス。複数のパスを追加することができ、パス間はセミコロン(;)で区切ります。BEマシンにいくつかのディスクがある場合は、それに応じていくつかのパスを追加することをお勧めします。
- デフォルト値：N/A

#### datacache_meta_path  

- 意味：Blockのメタデータを保存するディレクトリ。カスタマイズ可能です。**`$STARROCKS_HOME`**のパスに作成することをお勧めします。
- デフォルト値：N/A

#### datacache_block_size

- 意味：単一のblockのサイズ。単位：バイト。デフォルト値は`1048576`、つまり1MBです。
- デフォルト値：1048576

#### datacache_mem_size

- 意味：メモリキャッシュのデータ量の上限。割合の上限（例：`10%`）または物理的な上限（例：`10G`、`21474836480`など）として設定できます。デフォルト値は`10%`です。このパラメータの値は最低でも10GBに設定することをお勧めします。
- 単位：N/A
- デフォルト値：10%

#### datacache_disk_size

- 意味：単一のディスクのデータキャッシュ量の上限。割合の上限（例：`80%`）または物理的な上限（例：`2T`、`500G`など）として設定できます。例：`datacache_disk_path`に2つのディスクを設定し、`datacache_disk_size`のパラメータ値を`21474836480`、つまり20GBに設定した場合、最大で40GBのディスクデータをキャッシュできます。デフォルト値は`0`で、これはメモリのみをキャッシュ媒体として使用し、ディスクは使用しないことを意味します。
- 単位：N/A
- デフォルト値：0

#### jdbc_connection_pool_size

- 意味：JDBC接続プールのサイズ。各BEノードは`jdbc_url`が同じ外部テーブルにアクセスする際に同じ接続プールを共有します。
- デフォルト値：8

#### jdbc_minimum_idle_connections  

- 意味：JDBC接続プール内の最小限のアイドル接続数。
- デフォルト値：1

#### jdbc_connection_idle_timeout_ms  

- 意味：JDBCアイドル接続のタイムアウト時間。JDBC接続プール内の接続がこの値を超えてアイドル状態になると、接続プールは`jdbc_minimum_idle_connections`の設定項目で指定された数を超えるアイドル接続を閉じます。
- 単位：ミリ秒。
- デフォルト値：600000

#### query_cache_capacity  


- 意味：Query Cache のサイズを指定します。単位：バイト。デフォルトは 512 MB。最小値は 4 MB 以上です。現在の BE のメモリ容量が希望する Query Cache のサイズを満たせない場合は、BE のメモリ容量を増やしてから、適切な Query Cache のサイズを設定してください。<br />各 BE は独自の Query Cache ストレージスペースを持っており、BE は自身のローカル Query Cache ストレージスペースのみを Populate または Probe します。
- 単位：バイト
- デフォルト値：536870912

#### enable_event_based_compaction_framework  

- 意味：Event-based Compaction Framework を有効にするかどうか。`true` は有効にすることを意味し、`false` は無効にすることを意味します。有効にすると、タブレットの数が多い場合や個々のタブレットのデータ量が大きい場合に、compaction のコストを大幅に削減できます。
- デフォルト値：TRUE

#### lake_service_max_concurrency

- 意味：ストレージと計算が分離されたクラスターでの、RPC リクエストの最大同時実行数です。この閾値に達すると、新しいリクエストは拒否されます。この値を 0 に設定すると、同時実行数に制限はありません。
- 単位：N/A
- デフォルト値：0
