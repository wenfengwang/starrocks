---
displayed_sidebar: English
---

# StarRocks バージョン 2.5

## 2.5.17

リリース日: 2023年12月19日

### 新機能

- `max_tablet_rowset_num` という新しいメトリックを追加しました。これは行セットの最大許容数を設定するためのもので、コンパクションの問題を検出し、「バージョンが多すぎます」というエラーの発生を減らすのに役立ちます。[#36539](https://github.com/StarRocks/starrocks/pull/36539)
- [subdivide_bitmap](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/subdivide_bitmap/) 関数を追加しました。[#35817](https://github.com/StarRocks/starrocks/pull/35817)

### 改善点

- [SHOW ROUTINE LOAD](https://docs.starrocks.io/zh/docs/sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD/) ステートメントの結果に `OtherMsg` という新しいフィールドが追加され、最後に失敗したタスクに関する情報が表示されるようになりました。[#35806](https://github.com/StarRocks/starrocks/pull/35806)
- ごみ箱ファイルのデフォルト保持期間が元の3日間から1日間に変更されました。[#37113](https://github.com/StarRocks/starrocks/pull/37113)
- 主キーテーブルの全行セットに対するコンパクション時の永続的インデックス更新のパフォーマンスが最適化され、ディスク読み取りI/Oが削減されました。[#36819](https://github.com/StarRocks/starrocks/pull/36819)
- 主キーテーブルのコンパクションスコアを計算するロジックを最適化し、他の3種類のテーブルとより一貫した範囲内で主キーテーブルのコンパクションスコアを揃えました。[#36534](https://github.com/StarRocks/starrocks/pull/36534)
- MySQL外部テーブルおよびJDBCカタログ内の外部テーブルのクエリは、WHERE句にキーワードを含めることをサポートします。[#35917](https://github.com/StarRocks/starrocks/pull/35917)
- Spark Loadにbitmap_from_binary関数を追加し、バイナリデータのロードをサポートしました。[#36050](https://github.com/StarRocks/starrocks/pull/36050)
- bRPCの有効期限が1時間からセッション変数[`query_timeout`](https://docs.starrocks.io/zh/docs/3.2/reference/System_variable/#query_timeout)で指定された期間に短縮されました。これにより、RPCリクエストの有効期限によるクエリの失敗が防止されます。[#36778](https://github.com/StarRocks/starrocks/pull/36778)

### 互換性の変更

#### パラメータ

- 新しいBE設定項目 `enable_stream_load_verbose_log` が追加されました。デフォルト値は `false` です。このパラメータを `true` に設定すると、StarRocksはStream LoadジョブのHTTPリクエストとレスポンスを記録でき、トラブルシューティングが容易になります。[#36113](https://github.com/StarRocks/starrocks/pull/36113)
- BEの静的パラメータ `update_compaction_per_tablet_min_interval_seconds` が変更可能になりました。[#36819](https://github.com/StarRocks/starrocks/pull/36819)

### バグ修正

以下の問題を修正しました:

- ハッシュ結合中にクエリが失敗し、BEがクラッシュする問題。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- FEの設定項目 `enable_collect_query_detail_info` を `true` に設定するとFEのパフォーマンスが著しく低下する問題。[#35945](https://github.com/StarRocks/starrocks/pull/35945)
- 永続インデックスが有効な主キーテーブルに大量のデータをロードする際にエラーが発生する問題。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- `./agentctl.sh stop be` を使用してBEを停止する際にstarrocks_beプロセスが予期せず終了する問題。[#35108](https://github.com/StarRocks/starrocks/pull/35108)
- [array_distinct](https://docs.starrocks.io/docs/sql-reference/sql-functions/array-functions/array_distinct/) 関数が時々BEをクラッシュさせる問題。[#36377](https://github.com/StarRocks/starrocks/pull/36377)
- ユーザーがマテリアライズドビューをリフレッシュする際にデッドロックが発生する問題。[#35736](https://github.com/StarRocks/starrocks/pull/35736)
- 動的パーティショニングを使用する際にエラーが発生し、FEの起動に失敗する問題。[#36846](https://github.com/StarRocks/starrocks/pull/36846)

## 2.5.16

リリース日: 2023年12月1日

### バグ修正

以下の問題を修正しました:

- 特定のシナリオでグローバルランタイムフィルタがBEをクラッシュさせる問題。[#35776](https://github.com/StarRocks/starrocks/pull/35776)

## 2.5.15

リリース日: 2023年11月29日

### 改善点

- 遅いリクエストを追跡するための遅いリクエストログを追加しました。[#33908](https://github.com/StarRocks/starrocks/pull/33908)
- 多数のファイルが存在する場合にSpark Loadを使用してParquetファイルとORCファイルを読み取るパフォーマンスを最適化しました。[#34787](https://github.com/StarRocks/starrocks/pull/34787)
- 以下を含むいくつかのBitmap関連操作のパフォーマンスを最適化しました:
  - ネストされたループ結合を最適化しました。[#34804](https://github.com/StarRocks/starrocks/pull/34804) [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor` 関数を最適化しました。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - Copy on WriteをサポートしてBitmapのパフォーマンスを最適化し、メモリ消費を削減しました。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 互換性の変更

#### パラメータ

- FEの動的パラメータ `enable_new_publish_mechanism` が静的パラメータに変更されました。パラメータ設定を変更した後はFEを再起動する必要があります。[#35338](https://github.com/StarRocks/starrocks/pull/35338)

### バグ修正

- ブローカーロードジョブでフィルタリング条件が指定されている場合、特定の状況下でデータロード中にBEがクラッシュする問題。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- レプリカ操作のリプレイに失敗し、FEがクラッシュする問題。[#32295](https://github.com/StarRocks/starrocks/pull/32295)
- FEパラメータ `recover_with_empty_tablet` を `true` に設定するとFEがクラッシュする問題。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- クエリに対して「get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction」というエラーが返される問題。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- ウィンドウ関数を含むクエリがBEをクラッシュさせる問題。[#33671](https://github.com/StarRocks/starrocks/pull/33671)
- `show proc '/statistic'` を実行するとデッドロックが発生する問題。[#34237](https://github.com/StarRocks/starrocks/pull/34237/files)
- 永続インデックスが有効な主キーテーブルに大量のデータをロードする際にエラーが発生する問題。[#34566](https://github.com/StarRocks/starrocks/pull/34566)
- StarRocksをv2.4以前から新しいバージョンにアップグレードした後、コンパクションスコアが予期せず上昇する問題。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- データベースドライバMariaDB ODBCを使用して `INFORMATION_SCHEMA` をクエリすると、`schemata` ビューで返される `CATALOG_NAME` 列が `null` 値のみを保持する問題。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- ストリームロードジョブが **PREPARD** 状態である間にスキーマ変更が実行されると、ジョブによってロードされるソースデータの一部が失われる問題。[#34381](https://github.com/StarRocks/starrocks/pull/34381)

- HDFSストレージパスの末尾に2つ以上のスラッシュ(`/`)を含めると、HDFSからのデータのバックアップと復元が失敗することがあります。 [#34601](https://github.com/StarRocks/starrocks/pull/34601)
- ロードタスクまたはクエリを実行すると、FEがハングすることがあります。 [#34569](https://github.com/StarRocks/starrocks/pull/34569)

## 2.5.14

リリース日: 2023年11月14日

### 改善点

- `INFORMATION_SCHEMA`システムデータベースの`COLUMNS`テーブルが、ARRAY、MAP、およびSTRUCT列を表示できるようになりました。 [#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 互換性の変更

#### システム変数

- セッション変数`cbo_decimal_cast_string_strict`が追加され、CBOがDECIMAL型からSTRING型へのデータ変換をどのように制御するかを設定できます。この変数が`true`に設定されている場合、v2.5.x以降のバージョンで組み込まれたロジックが優先され、システムは厳密な変換を実施します（つまり、生成された文字列は切り捨てられ、スケールの長さに基づいて0が埋められます）。この変数が`false`に設定されている場合、v2.5.xより前のバージョンで組み込まれたロジックが優先され、システムはすべての有効な桁を処理して文字列を生成します。デフォルト値は`true`です。 [#34208](https://github.com/StarRocks/starrocks/pull/34208)
- セッション変数`cbo_eq_base_type`が追加され、DECIMAL型データとSTRING型データ間の比較で使用するデータ型を指定できます。デフォルト値は`VARCHAR`で、DECIMALも有効な値です。 [#34208](https://github.com/StarRocks/starrocks/pull/34208)

### バグ修正

以下の問題を修正しました：

- ON条件がサブクエリでネストされている場合に`java.lang.IllegalStateException: null`エラーが報告される問題。 [#30876](https://github.com/StarRocks/starrocks/pull/30876)
- `INSERT INTO SELECT ... LIMIT`が成功した直後に`COUNT(*)`を実行すると、レプリカ間で`COUNT(*)`の結果が一致しない問題。 [#24435](https://github.com/StarRocks/starrocks/pull/24435)
- `cast()`関数で指定されたターゲットデータ型が元のデータ型と同じである場合に、特定のデータ型でBEがクラッシュする問題。 [#31465](https://github.com/StarRocks/starrocks/pull/31465)
- Broker Loadを使用したデータロード時に特定のパス形式を使用するとエラーが報告される問題：`msg:Fail to parse columnsFromPath, expected: [rec_dt]`。 [#32721](https://github.com/StarRocks/starrocks/issues/32721)
- 3.xへのアップグレード中に、一部のカラムタイプがアップグレードされる（例：DecimalがDecimalV3にアップグレードされる）と、特定の特性を持つテーブルでコンパクションを実行する際にBEがクラッシュする問題。 [#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードする際、高い並行ロードジョブが存在し、HTTPおよびScanスレッドの数が上限に達した場合にロードジョブが予期せず中断される問題。 [#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlが呼び出された際にBEがクラッシュする問題。 [#31667](https://github.com/StarRocks/starrocks/pull/31667)
- プライマリキーテーブルにBITMAPカラムを追加する際に、`Analyze columnDef error: No aggregate function specified for 'userid'`エラーで失敗する問題。 [#31763](https://github.com/StarRocks/starrocks/pull/31763)
- 永続インデックスが有効なプライマリキーテーブルに長時間頻繁にデータをロードするとBEがクラッシュする問題。 [#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効な場合にクエリ結果が不正確になる問題。 [#32778](https://github.com/StarRocks/starrocks/pull/32778)
- プライマリキーテーブルを作成する際にnull許容ソートキーを指定するとコンパクションが失敗する問題。 [#29225](https://github.com/StarRocks/starrocks/pull/29225)
- 複雑なJoinクエリで「StarRocks planner use long time 10000 ms in logical phase」というエラーが時々発生する問題。 [#34177](https://github.com/StarRocks/starrocks/pull/34177)

## 2.5.13

リリース日: 2023年9月28日

### 改善点

- ウィンドウ関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPがORDER BY句とウィンドウ句をサポートするようになりました。 [#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型データに対するクエリ中に10進数のオーバーフローが発生した場合、NULLではなくエラーが返されるようになりました。 [#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドを実行した場合、MySQLと一致する結果が返されるようになりました。 [#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 削除されたタブレットに対応するRowsetsがクリーンアップされ、BE起動時のメモリ使用量が削減されました。 [#30625](https://github.com/StarRocks/starrocks/pull/30625)

### バグ修正

以下の問題を修正しました：

- ユーザーがSpark ConnectorまたはFlink Connectorを使用してStarRocksからデータを読み取る際に「Set cancelled by MemoryScratchSinkOperator」というエラーが発生する問題。 [#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 集約関数を含むORDER BY句を使用したクエリ中に`java.lang.IllegalStateException: null`エラーが発生する問題。 [#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 非アクティブなマテリアライズドビューが存在する場合にFEが再起動に失敗する問題。 [#30015](https://github.com/StarRocks/starrocks/pull/30015)
- 重複するパーティションに対してINSERT OVERWRITE操作を実行するとメタデータが破損し、FEの再起動が失敗する問題。 [#27545](https://github.com/StarRocks/starrocks/pull/27545)
- ユーザーが主キーテーブルに存在しない列を変更する際に`java.lang.NullPointerException: null`エラーが発生する問題。 [#30366](https://github.com/StarRocks/starrocks/pull/30366)
- ユーザーがパーティション化されたStarRocks外部テーブルにデータをロードする際に`get TableMeta failed from TNetworkAddress`エラーが発生する問題。 [#30124](https://github.com/StarRocks/starrocks/pull/30124)
- 特定のシナリオでユーザーがCloudCanal経由でデータをロードする際にエラーが発生する問題。 [#30799](https://github.com/StarRocks/starrocks/pull/30799)
- ユーザーがFlink Connectorを介してデータをロードするか、DELETEおよびINSERT操作を実行する際に`current running txns on db xxx is 200, larger than limit 200`エラーが発生する問題。 [#18393](https://github.com/StarRocks/starrocks/pull/18393)
- 集約関数を含むHAVING句を使用する非同期マテリアライズドビューがクエリを正しく書き換えられない問題。 [#29976](https://github.com/StarRocks/starrocks/pull/29976)

## 2.5.12

リリース日: 2023年9月4日

### 改善点

- SQLのコメントが監査ログに保持されるようになりました。 [#29747](https://github.com/StarRocks/starrocks/pull/29747)
- INSERT INTO SELECTのCPUとメモリの統計が監査ログに追加されました。 [#29901](https://github.com/StarRocks/starrocks/pull/29901)

### バグ修正

以下の問題を修正しました：

- Broker Loadを使用してデータをロードする際、一部のフィールドのNOT NULL属性が原因でBEがクラッシュしたり、「msg:mismatched row count」エラーが発生したりする問題。 [#29832](https://github.com/StarRocks/starrocks/pull/29832)

- Apache ORCのバグ修正ORC-1304 ([apache/orc#1299](https://github.com/apache/orc/pull/1299))がマージされていないため、ORC形式のファイルに対するクエリは失敗します。[#29804](https://github.com/StarRocks/starrocks/pull/29804)

- Primary Keyテーブルを復元すると、BEが再起動された後にメタデータの不整合が発生します。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 2.5.11

リリース日: 2023年8月28日

### 改善点

- すべての複合述語とWHERE句内のすべての式に対する暗黙の型変換をサポートします。セッション変数`enable_strict_type`を使用して暗黙の型変換を有効または無効にできます。デフォルト値は`false`です。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- ユーザーがIcebergカタログを作成する際に`hive.metastore.uri`を指定しない場合のプロンプトを最適化しました。エラープロンプトがより正確になりました。[#16543](https://github.com/StarRocks/starrocks/issues/16543)
- エラーメッセージ`xxx too many versions xxx`に追加のプロンプトを追加しました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 動的パーティショニングが`year`単位のパーティショニングをさらにサポートします。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

### バグ修正

以下の問題を修正しました:

- 複数のレプリカを持つテーブルにデータがロードされた際、テーブルの一部のパーティションが空である場合に多数の無効なログレコードが書き込まれます。[#28824](https://github.com/StarRocks/starrocks/issues/28824)
- WHERE条件のフィールドがBITMAPまたはHLLフィールドである場合、DELETE操作が失敗します。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 非同期マテリアライズドビューを同期呼び出し(SYNC MODE)で手動でリフレッシュすると、`information_schema.task_runs`テーブルに複数のINSERT OVERWRITEレコードが生成されます。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- ERROR状態のタブレットに対してCLONE操作がトリガされると、ディスク使用量が増加します。[#28488](https://github.com/StarRocks/starrocks/pull/28488)
- Join Reorderが有効になっている場合、クエリ対象の列が定数であるとクエリ結果が不正確になります。[#29239](https://github.com/StarRocks/starrocks/pull/29239)
- SSDとHDD間でタブレットを移行する際に、FEがBEに過剰な移行タスクを送信すると、BEがOOMに遭遇することがあります。[#29055](https://github.com/StarRocks/starrocks/pull/29055)
- `/apache_hdfs_broker/lib/log4j-1.2.17.jar`におけるセキュリティ脆弱性。[#28866](https://github.com/StarRocks/starrocks/pull/28866)
- Hive Catalogを通じてデータクエリを行う際、WHERE句でパーティションカラムとOR演算子を使用すると、クエリ結果が不正確になります。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- データクエリ中に時折`java.util.ConcurrentModificationException: null`エラーが発生します。[#29296](https://github.com/StarRocks/starrocks/pull/29296)
- 非同期マテリアライズドビューのベーステーブルが削除された場合、FEを再起動できなくなります。[#29318](https://github.com/StarRocks/starrocks/pull/29318)
- 複数のデータベースにまたがる非同期マテリアライズドビューが作成されている場合、このマテリアライズドビューのベーステーブルにデータを書き込む際にLeader FEがデッドロックに陥ることがあります。[#29432](https://github.com/StarRocks/starrocks/pull/29432)

## 2.5.10

リリース日: 2023年8月7日

### 新機能

- 集約関数[COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、および[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)をサポートします。
- [ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPをサポートします。

### 改善点

- TabletCheckerのスケジューリングロジックを最適化し、修復されていないタブレットの繰り返しスケジューリングを防ぎます。[#27648](https://github.com/StarRocks/starrocks/pull/27648)
- スキーマ変更とルーチンロードが同時に発生する場合、スキーマ変更が先に完了するとルーチンロードジョブが失敗する可能性があります。この状況で報告されるエラーメッセージが最適化されました。[#28425](https://github.com/StarRocks/starrocks/pull/28425)
- ユーザーが外部テーブルを作成する際にNOT NULLカラムを定義することは禁止されています（NOT NULLカラムが定義されている場合、アップグレード後にエラーが発生し、テーブルを再作成する必要があります）。v2.3.0以降、外部テーブルの代わりに外部カタログの使用が推奨されます。[#25485](https://github.com/StarRocks/starrocks/pull/25441)
- Broker Loadのリトライ時にエラーが発生した場合のエラーメッセージを追加しました。これにより、データロード中のトラブルシューティングとデバッグが容易になります。[#21982](https://github.com/StarRocks/starrocks/pull/21982)
- UPSERTとDELETE操作の両方が含まれるロードジョブにおいて、大規模なデータ書き込みをサポートします。[#17264](https://github.com/StarRocks/starrocks/pull/17264)
- マテリアライズドビューを使用したクエリの書き換えを最適化しました。[#27934](https://github.com/StarRocks/starrocks/pull/27934) [#25542](https://github.com/StarRocks/starrocks/pull/25542) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#27557](https://github.com/StarRocks/starrocks/pull/27557) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#26957](https://github.com/StarRocks/starrocks/pull/26957) [#27728](https://github.com/StarRocks/starrocks/pull/27728) [#27900](https://github.com/StarRocks/starrocks/pull/27900)

### バグ修正

以下の問題を修正しました:

- CASTを使用して文字列を配列に変換する場合、入力に定数が含まれていると結果が不正確になることがあります。[#19793](https://github.com/StarRocks/starrocks/pull/19793)
- SHOW TABLETがORDER BYとLIMITを含む場合、結果が不正確になります。[#23375](https://github.com/StarRocks/starrocks/pull/23375)
- マテリアライズドビューのOuter joinとAnti joinの書き換えエラー。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- FEのテーブルレベルのスキャン統計が不正確で、テーブルクエリとローディングのメトリクスが不正確になります。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- HMSでイベントリスナーが設定されており、Hiveメタデータを増分的に更新する場合、FEログに`An exception occurred when using the current long link to access metastore. msg: Failed to get next notification based on last event id: 707602`と報告されます。[#21056](https://github.com/StarRocks/starrocks/pull/21056)
- パーティションテーブルのソートキーを変更すると、クエリ結果が安定しないことがあります。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- Spark Loadを使用してロードされたデータが、バケット列がDATE、DATETIME、またはDECIMAL列である場合、誤ったバケットに分散される可能性があります。[#27005](https://github.com/StarRocks/starrocks/pull/27005)
- regex_replace関数を使用すると、一部のシナリオでBEがクラッシュする可能性があります。[#27117](https://github.com/StarRocks/starrocks/pull/27117)
- BE がクラッシュする場合があります。これは、`sub_bitmap` 関数の入力が BITMAP 値でない場合です。 [#27982](https://github.com/StarRocks/starrocks/pull/27982)
- Join Reorder 機能が有効な場合、「Unknown error」がクエリに対して返されます。 [#27472](https://github.com/StarRocks/starrocks/pull/27472)
- 平均行サイズの見積もりが不正確な場合、Primary Key の部分更新が過度に大きなメモリを占有することがあります。 [#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 低カーディナリティ最適化が有効な場合、一部の INSERT ジョブが `[42000][1064] Dict Decode failed, Dict can't take cover all key :0` というエラーを返すことがあります。[#26463](https://github.com/StarRocks/starrocks/pull/26463)
- ユーザーが Broker Load ジョブで `"hadoop.security.authentication" = "simple"` を指定して HDFS からデータをロードしようとした場合、ジョブが失敗することがあります。 [#27774](https://github.com/StarRocks/starrocks/pull/27774)
- マテリアライズドビューのリフレッシュモードを変更すると、リーダー FE とフォロワー FE 間でメタデータが一致しなくなることがあります。 [#28082](https://github.com/StarRocks/starrocks/pull/28082) [#28097](https://github.com/StarRocks/starrocks/pull/28097)
- `SHOW CREATE CATALOG` および `SHOW RESOURCES` を使用して特定の情報を照会する際、パスワードが非表示にならない問題があります。 [#28059](https://github.com/StarRocks/starrocks/pull/28059)
- LabelCleaner スレッドがブロックされることによる FE のメモリリークが発生することがあります。 [#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 2.5.9

リリース日: 2023年7月19日

### 新機能

- マテリアライズドビューとは異なるタイプの結合を含むクエリが書き換え可能になりました。 [#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改善点

- 現在の StarRocks クラスターが宛先クラスターである StarRocks 外部テーブルは作成できなくなりました。 [#25441](https://github.com/StarRocks/starrocks/pull/25441)
- クエリされたフィールドがマテリアライズドビューの出力列には含まれていないが、述語には含まれている場合、クエリの書き換えが可能になりました。 [#23028](https://github.com/StarRocks/starrocks/issues/23028)
- `Information_schema` データベースの `tables_config` テーブルに `table_id` という新しいフィールドが追加されました。`table_id` 列を使用して `tables_config` と `be_tablets` を結合し、タブレットが属するデータベースとテーブルの名前を照会できます。 [#24061](https://github.com/StarRocks/starrocks/pull/24061)

### バグ修正

以下の問題を修正しました：

- Duplicate Key テーブルにおける Count Distinct の結果が不正確です。 [#24222](https://github.com/StarRocks/starrocks/pull/24222)
- Join キーが大きな BINARY 列である場合、BE がクラッシュすることがあります。 [#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 挿入される STRUCT 内の CHAR データの長さが、STRUCT 列で定義された最大 CHAR 長を超える場合、INSERT 操作がハングアップすることがあります。 [#25942](https://github.com/StarRocks/starrocks/pull/25942)
- `coalesce()` 関数の結果が不正確です。 [#26250](https://github.com/StarRocks/starrocks/pull/26250)
- データ復元後、タブレットのバージョン番号が BE と FE で不一致になることがあります。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 復旧したテーブルに対してパーティションが自動的に作成されない問題があります。 [#26813](https://github.com/StarRocks/starrocks/pull/26813)

## 2.5.8

リリース日: 2023年6月30日

### 改善点

- パーティション化されていないテーブルにパーティションを追加する際のエラーメッセージが最適化されました。 [#25266](https://github.com/StarRocks/starrocks/pull/25266)
- テーブルの [auto tablet distribution policy](../table_design/Data_distribution.md#determine-the-number-of-buckets) が最適化されました。 [#24543](https://github.com/StarRocks/starrocks/pull/24543)
- `CREATE TABLE` ステートメントのデフォルトコメントが最適化されました。 [#24803](https://github.com/StarRocks/starrocks/pull/24803)
- 非同期マテリアライズドビューの手動リフレッシュが最適化され、`REFRESH MATERIALIZED VIEW WITH SYNC MODE` 構文を使用してマテリアライズドビューのリフレッシュタスクを同期的に実行できるようになりました。 [#25910](https://github.com/StarRocks/starrocks/pull/25910)

### バグ修正

以下の問題を修正しました：

- 非同期マテリアライズドビューの COUNT 結果が、マテリアライズドビューが Union 結果に基づいて構築された場合に不正確になることがあります。 [#24460](https://github.com/StarRocks/starrocks/issues/24460)
- ユーザーが root パスワードを強制リセットしようとすると「Unknown error」が報告されることがあります。 [#25492](https://github.com/StarRocks/starrocks/pull/25492)
- 3 つ未満のアクティブ BE を持つクラスターで `INSERT OVERWRITE` を実行すると、不正確なエラーメッセージが表示されることがあります。 [#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 2.5.7

リリース日: 2023年6月14日

### 新機能

- 非アクティブなマテリアライズドビューは、`ALTER MATERIALIZED VIEW <mv_name> ACTIVE` を使用して手動でアクティブ化できます。この SQL コマンドを使用して、ベーステーブルが削除された後に再作成されたマテリアライズドビューをアクティブ化できます。詳細は [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を参照してください。 [#24001](https://github.com/StarRocks/starrocks/pull/24001)
- StarRocks は、テーブルを作成する際やパーティションを追加する際に、適切なタブレット数を自動的に設定することができます。これにより、手動操作が不要になります。詳細は [タブレットの数を決定する](../table_design/Data_distribution.md#determine-the-number-of-buckets) を参照してください。 [#10614](https://github.com/StarRocks/starrocks/pull/10614)

### 改善点

- 外部テーブルクエリで使用される Scan ノードの I/O 並行性が最適化され、メモリ使用量が削減され、外部テーブルからのデータロードの安定性が向上しました。 [#23617](https://github.com/StarRocks/starrocks/pull/23617) [#23624](https://github.com/StarRocks/starrocks/pull/23624) [#23626](https://github.com/StarRocks/starrocks/pull/23626)
- Broker Load ジョブのエラーメッセージが最適化され、再試行情報とエラーが発生したファイルの名前が含まれるようになりました。 [#18038](https://github.com/StarRocks/starrocks/pull/18038) [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- `CREATE TABLE` がタイムアウトした際に返されるエラーメッセージが最適化され、パラメータ調整のヒントが追加されました。 [#24510](https://github.com/StarRocks/starrocks/pull/24510)
- テーブルのステータスが Normal でないために `ALTER TABLE` が失敗した際に返されるエラーメッセージが最適化されました。 [#24381](https://github.com/StarRocks/starrocks/pull/24381)
- `CREATE TABLE` ステートメントで全角スペースを無視するようになりました。 [#23885](https://github.com/StarRocks/starrocks/pull/23885)
- Broker アクセスのタイムアウトが最適化され、Broker Load ジョブの成功率が向上しました。 [#22699](https://github.com/StarRocks/starrocks/pull/22699)
- Primary Key テーブルの場合、`SHOW TABLET` で返される `VersionCount` フィールドに Pending 状態の Rowsets が含まれるようになりました。 [#23847](https://github.com/StarRocks/starrocks/pull/23847)
- Persistent Index ポリシーが最適化されました。 [#22140](https://github.com/StarRocks/starrocks/pull/22140)

### バグ修正

以下の問題を修正しました：

- ユーザーが Parquet データを StarRocks にロードする際、型変換で DATETIME 値がオーバーフローし、データエラーが発生する問題。 [#22356](https://github.com/StarRocks/starrocks/pull/22356)
- 動的パーティショニングを無効にした後、バケット情報が失われる問題。 [#22595](https://github.com/StarRocks/starrocks/pull/22595)
- CREATE TABLE ステートメントでサポートされていないプロパティを使用すると、nullポインター例外 (NPE) が発生する問題。 [#23859](https://github.com/StarRocks/starrocks/pull/23859)
- `information_schema` でのテーブル権限フィルタリングが無効になり、結果としてユーザーが権限を持たないテーブルを閲覧できる問題。 [#23804](https://github.com/StarRocks/starrocks/pull/23804)
- SHOW TABLE STATUS が返す情報が不完全な問題。 [#24279](https://github.com/StarRocks/starrocks/issues/24279)
- スキーマ変更中にデータロードが同時に行われると、スキーマ変更がハングすることがある問題。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- RocksDB WALのフラッシュがbrpcワーカーのbthreads処理をブロックし、プライマリキーテーブルへの高頻度データロードが中断される問題。 [#22489](https://github.com/StarRocks/starrocks/pull/22489)
- StarRocks でサポートされていない TIME 型のカラムが正常に作成されてしまう問題。 [#23474](https://github.com/StarRocks/starrocks/pull/23474)
- マテリアライズドビューの Union 書き換えが失敗する問題。 [#22922](https://github.com/StarRocks/starrocks/pull/22922)

## 2.5.6

リリース日: 2023年5月19日

### 改善点

- `thrift_server_max_worker_thread` の値が小さいために INSERT INTO ... SELECT がタイムアウトする場合のエラーメッセージを最適化しました。 [#21964](https://github.com/StarRocks/starrocks/pull/21964)
- CTAS を使用して作成されたテーブルはデフォルトで3つのレプリカを持ち、これは通常テーブルのデフォルトレプリカ数と一致します。 [#22854](https://github.com/StarRocks/starrocks/pull/22854)

### バグ修正

- TRUNCATE 操作がパーティション名の大文字と小文字を区別するため、パーティションの切り捨てに失敗する問題。 [#21809](https://github.com/StarRocks/starrocks/pull/21809)
- マテリアライズドビューの一時パーティションの作成に失敗し、BE のデコミッションが失敗する問題。 [#22745](https://github.com/StarRocks/starrocks/pull/22745)
- ARRAY 値が必要な動的 FE パラメータを空の配列に設定できない問題。 [#22225](https://github.com/StarRocks/starrocks/pull/22225)
- `partition_refresh_number` プロパティを指定したマテリアライズドビューが完全にリフレッシュされない問題。 [#21619](https://github.com/StarRocks/starrocks/pull/21619)
- SHOW CREATE TABLE がクラウド認証情報をマスクし、結果としてメモリ内の認証情報が不正確になる問題。 [#21311](https://github.com/StarRocks/starrocks/pull/21311)
- 外部テーブル経由でクエリされる一部の ORC ファイルに述部が効果を発揮しない問題。 [#21901](https://github.com/StarRocks/starrocks/pull/21901)
- min-max フィルタが列名の大文字と小文字を正しく処理できない問題。 [#22626](https://github.com/StarRocks/starrocks/pull/22626)
- 遅延マテリアライズが複合データ型 (STRUCT または MAP) のクエリにエラーを引き起こす問題。 [#22862](https://github.com/StarRocks/starrocks/pull/22862)
- プライマリキーテーブルの復元時に発生する問題。 [#23384](https://github.com/StarRocks/starrocks/pull/23384)

## 2.5.5

リリース日: 2023年4月28日

### 新機能

プライマリキーテーブルのタブレットステータスを監視するメトリックを追加しました：

- FE メトリック `err_state_metric` を追加しました。
- `SHOW PROC '/statistic/'` の出力に `ErrorStateTabletNum` 列を追加して、**err_state** タブレットの数を表示します。
- `SHOW PROC '/statistic/<db_id>/'` の出力に `ErrorStateTablets` 列を追加して、**err_state** タブレットの ID を表示します。

詳細については、[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md) を参照してください。

### 改善点

- 複数の BE を追加する際のディスクバランシング速度を最適化しました。 [#19418](https://github.com/StarRocks/starrocks/pull/19418)
- `storage_medium` の推論を最適化しました。BE が SSD と HDD をストレージデバイスとして使用している場合、`storage_cooldown_time` プロパティが指定されていると、StarRocks は `storage_medium` を `SSD` に設定します。それ以外の場合は `HDD` に設定します。 [#18649](https://github.com/StarRocks/starrocks/pull/18649)
- 値列からの統計収集を禁止することで、ユニークキーテーブルのパフォーマンスを最適化しました。 [#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

- コロケーションテーブルでは、`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");` のようなステートメントを使用してレプリカステータスを手動で `bad` に設定できます。BE の数がレプリカ数以下の場合、破損したレプリカは修復できない問題。 [#17876](https://github.com/StarRocks/starrocks/issues/17876)
- BE が起動した後、プロセスは存在するものの BE ポートが有効にならない問題。 [#19347](https://github.com/StarRocks/starrocks/pull/19347)
- サブクエリがウィンドウ関数でネストされている集約クエリに対して誤った結果が返される問題。 [#19725](https://github.com/StarRocks/starrocks/issues/19725)
- マテリアライズドビュー (MV) が初めてリフレッシュされたときに `auto_refresh_partitions_limit` が効果を発揮せず、結果としてすべてのパーティションがリフレッシュされる問題。 [#19759](https://github.com/StarRocks/starrocks/issues/19759)
- 配列データが MAP や STRUCT などの複雑なデータでネストされている CSV Hive 外部テーブルをクエリする際に発生するエラー。 [#20233](https://github.com/StarRocks/starrocks/pull/20233)
- Spark コネクタを使用したクエリがタイムアウトする問題。 [#20264](https://github.com/StarRocks/starrocks/pull/20264)
- 2 つのレプリカを持つテーブルの 1 つのレプリカが破損している場合、テーブルが回復できない問題。 [#20681](https://github.com/StarRocks/starrocks/pull/20681)
- マテリアライズドビュー (MV) クエリの書き換え失敗によるクエリ失敗の問題。 [#19549](https://github.com/StarRocks/starrocks/issues/19549)
- データベースロックによりメトリックインターフェースがタイムアウトする問題。 [#20790](https://github.com/StarRocks/starrocks/pull/20790)
- ブロードキャストジョインで誤った結果が返される問題。 [#20952](https://github.com/StarRocks/starrocks/issues/20952)
- CREATE TABLE でサポートされていないデータ型を使用した場合に NPE が返される問題。 [#20999](https://github.com/StarRocks/starrocks/issues/20999)
- クエリキャッシュ機能を使用した window_funnel() で問題が発生する問題。 [#21474](https://github.com/StarRocks/starrocks/issues/21474)
- CTE が書き換えられた後、最適化プランの選択に予想外に長い時間がかかる問題。 [#16515](https://github.com/StarRocks/starrocks/pull/16515)

## 2.5.4

リリース日: 2023年4月4日

### 改善点

- クエリ計画時にマテリアライズドビューのクエリを書き換えるパフォーマンスを最適化しました。クエリ計画にかかる時間が約70%削減されます。[#19579](https://github.com/StarRocks/starrocks/pull/19579)
- 型推論ロジックを最適化しました。例えば `SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;` のようなクエリに `0` が含まれている場合、クエリを高速化するためにプリアグリゲーションが自動的に有効になります。[#19474](https://github.com/StarRocks/starrocks/pull/19474)
- 実体化ビューの作成ステートメントを表示する `SHOW CREATE VIEW` の使用をサポートします。[#19999](https://github.com/StarRocks/starrocks/pull/19999)
- BEノード間で2GB以上のサイズのパケットを1つのbRPCリクエストで送信することをサポートします。[#20283](https://github.com/StarRocks/starrocks/pull/20283) [#20230](https://github.com/StarRocks/starrocks/pull/20230)
- [SHOW CREATE CATALOG](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して外部カタログの作成ステートメントを照会することをサポートします。

### バグ修正

以下のバグが修正されました：

- 実体化ビューに対するクエリが書き換えられた後、低カーディナリティ最適化用のグローバルディクショナリが機能しなくなります。[#19615](https://github.com/StarRocks/starrocks/pull/19615)
- 実体化ビューに対するクエリの書き換えに失敗すると、クエリが失敗します。[#19774](https://github.com/StarRocks/starrocks/pull/19774)
- 主キーまたはユニークキーのテーブルに基づいて作成された実体化ビューに対するクエリは書き換えられません。[#19600](https://github.com/StarRocks/starrocks/pull/19600)
- 実体化ビューの列名は大文字と小文字を区別します。しかし、テーブル作成時の `PROPERTIES` で列名が誤っていてもエラーメッセージなしにテーブルが作成され、さらにそのテーブルに基づいて作成された実体化ビューに対するクエリの書き換えが失敗します。[#19780](https://github.com/StarRocks/starrocks/pull/19780)
- 実体化ビューに対するクエリが書き換えられた後、クエリプランにパーティション列に基づく無効な述語が含まれることがあり、クエリのパフォーマンスに影響を与えます。[#19784](https://github.com/StarRocks/starrocks/pull/19784)
- 新しく作成されたパーティションにデータをロードした後、実体化ビューに対するクエリの書き換えが失敗することがあります。[#20323](https://github.com/StarRocks/starrocks/pull/20323)
- 実体化ビューの作成時に `"storage_medium" = "SSD"` を設定すると、実体化ビューのリフレッシュが失敗します。[#19539](https://github.com/StarRocks/starrocks/pull/19539) [#19626](https://github.com/StarRocks/starrocks/pull/19626)
- 主キーテーブルで同時コンパクションが発生することがあります。[#19692](https://github.com/StarRocks/starrocks/pull/19692)
- 大量のDELETE操作後にコンパクションが適時に行われないことがあります。[#19623](https://github.com/StarRocks/starrocks/pull/19623)
- ステートメントの式に低カーディナリティ列が複数含まれる場合、式が適切に書き換えられず、低カーディナリティ最適化用のグローバルディクショナリが機能しなくなることがあります。[#20161](https://github.com/StarRocks/starrocks/pull/20161)

## 2.5.3

リリース日: 2023年3月10日

### 改善点

- 実体化ビュー(MV)のクエリ書き換えを最適化しました。
  - Outer JoinおよびCross Joinを含むクエリの書き換えをサポートします。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - MVのデータスキャンロジックを最適化し、書き換えられたクエリの速度をさらに向上させました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 単一テーブル集約クエリの書き換え能力を強化しました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - クエリされたテーブルがMVのベーステーブルのサブセットであるView Deltaシナリオでの書き換え能力を強化しました。[#18800](https://github.com/StarRocks/starrocks/pull/18800)
- ウィンドウ関数RANK()をフィルターやソートキーとして使用する際のパフォーマンスとメモリ使用量を最適化しました。[#17553](https://github.com/StarRocks/starrocks/issues/17553)

### バグ修正

以下のバグが修正されました：

- ARRAYデータのnullリテラル `[]` が原因で発生したエラー。[#18563](https://github.com/StarRocks/starrocks/pull/18563)
- 複雑なクエリシナリオで低カーディナリティ最適化ディクショナリを誤って使用した場合。ディクショナリを適用する前にディクショナリマッピングチェックが追加されました。[#17318](https://github.com/StarRocks/starrocks/pull/17318)
- 単一BE環境でLocal ShuffleがGROUP BYによる重複結果を生成する問題。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- パーティション化されていないMVに対してパーティション関連のPROPERTIESを誤用すると、MVのリフレッシュが失敗する可能性があります。MVを作成する際にパーティションPROPERTIESチェックが行われるようになりました。[#18741](https://github.com/StarRocks/starrocks/pull/18741)
- Parquet Repetition列の解析エラー。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 取得された列のnullable情報が不正確です。解決策: CTASを使用して主キーテーブルを作成する場合、主キー列のみがnull非許容です。非主キー列はnullableです。[#16431](https://github.com/StarRocks/starrocks/pull/16431)
- 主キーテーブルからのデータ削除によって引き起こされる問題。[#18768](https://github.com/StarRocks/starrocks/pull/18768)

## 2.5.2

リリース日: 2023年2月21日

### 新機能

- AWS S3およびAWS Glueにアクセスするためのインスタンスプロファイルとアサンプトロールベースの認証方法をサポートします。[#15958](https://github.com/StarRocks/starrocks/pull/15958)
- 以下のビット関数をサポートします: `bit_shift_left`, `bit_shift_right`, `bit_shift_right_logical`。[#14151](https://github.com/StarRocks/starrocks/pull/14151)

### 改善点

- 多数の集約クエリが含まれるクエリのメモリ解放ロジックを最適化し、ピーク時のメモリ使用量を大幅に削減しました。[#16913](https://github.com/StarRocks/starrocks/pull/16913)
- ソート時のメモリ使用量を削減しました。ウィンドウ関数やソートを含むクエリでは、メモリ消費量が半減します。[#16937](https://github.com/StarRocks/starrocks/pull/16937) [#17362](https://github.com/StarRocks/starrocks/pull/17362) [#17408](https://github.com/StarRocks/starrocks/pull/17408)

### バグ修正

以下のバグが修正されました：

- MAPおよびARRAYデータを含むApache Hive外部テーブルがリフレッシュできない問題。[#17548](https://github.com/StarRocks/starrocks/pull/17548)
- Supersetが実体化ビューのカラムタイプを識別できない問題。[#17686](https://github.com/StarRocks/starrocks/pull/17686)
- `SET GLOBAL/SESSION TRANSACTION` が解析できず、BI接続が失敗する問題。[#17295](https://github.com/StarRocks/starrocks/pull/17295)
- Colocate Group内の動的パーティションテーブルのバケット数は変更できず、エラーメッセージが返されます。[#17418](https://github.com/StarRocks/starrocks/pull/17418/)
- Prepareステージの失敗によって引き起こされる潜在的な問題。[#17323](https://github.com/StarRocks/starrocks/pull/17323)

### 動作変更

- `enable_experimental_mv` のデフォルト値を `false` から `true` に変更しました。これは、非同期マテリアライズドビューがデフォルトで有効になることを意味します。
- 予約語リストにCHARACTERを追加しました。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.1

リリース日: 2023年2月5日

### 改善点

- 外部カタログに基づいて作成された非同期マテリアライズドビューは、クエリ書き換えをサポートします。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#15791](https://github.com/StarRocks/starrocks/issues/15791)
- 自動CBO統計収集の収集期間を指定することで、自動全収集によるクラスタパフォーマンスのジッターを防ぐことができます。[#14996](https://github.com/StarRocks/starrocks/pull/14996)
- Thriftサーバーキューを追加しました。INSERT INTO SELECT中に即時処理できないリクエストはThriftサーバーキューで保留され、リクエストが拒否されるのを防ぎます。[#14571](https://github.com/StarRocks/starrocks/pull/14571)
- FEパラメータ`default_storage_medium`を非推奨にしました。ユーザーがテーブルを作成する際に`storage_medium`が明示的に指定されていない場合、システムはBEディスクタイプに基づいてテーブルのストレージ媒体を自動的に推測します。詳細は`storage_medium`の[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_VIEW.md)の説明を参照してください。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### バグ修正

以下のバグが修正されました：

- SET PASSWORDによるNull Pointer Exception (NPE)。[#15247](https://github.com/StarRocks/starrocks/pull/15247)
- 空のキーを含むJSONデータが解析できない問題。[#16852](https://github.com/StarRocks/starrocks/pull/16852)
- 不正な型のデータがARRAYデータに正常に変換される問題。[#16866](https://github.com/StarRocks/starrocks/pull/16866)
- 例外発生時にNested Loop Joinが中断できない問題。[#16875](https://github.com/StarRocks/starrocks/pull/16875)

### 動作変更

- FEパラメータ`default_storage_medium`を非推奨にしました。テーブルのストレージ媒体はシステムによって自動的に推測されます。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.5.0

リリース日: 2023年1月22日

### 新機能

- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)および[Hudi外部テーブル](../data_source/External_table.md#deprecated-hudi-external-table)を使用したMerge On Readテーブルのクエリをサポートします。[#6780](https://github.com/StarRocks/starrocks/pull/6780)
- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、Hudiカタログ、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用したSTRUCTおよびMAPデータのクエリをサポートします。[#10677](https://github.com/StarRocks/starrocks/issues/10677)
- [データキャッシュ](../data_source/data_cache.md)を提供して、HDFSなどの外部ストレージシステムに保存されたホットデータのアクセス性能を向上させます。[#11597](https://github.com/StarRocks/starrocks/pull/11579)
- Delta Lakeからのデータに直接クエリを可能にする[Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)の作成をサポートします。[#11972](https://github.com/StarRocks/starrocks/issues/11972)
- Hive、Hudi、およびIcebergカタログはAWS Glueと互換性があります。[#12249](https://github.com/StarRocks/starrocks/issues/12249)
- HDFSおよびオブジェクトストアからのParquetおよびORCファイルに直接クエリを可能にする[ファイル外部テーブル](../data_source/file_external_table.md)の作成をサポートします。[#13064](https://github.com/StarRocks/starrocks/pull/13064)
- Hive、Hudi、Icebergカタログおよびマテリアライズドビューに基づくマテリアライズドビューの作成をサポートします。詳細は[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#11873](https://github.com/StarRocks/starrocks/pull/11873)
- 主キーテーブルを使用するテーブルの条件付き更新をサポートします。詳細は[データの変更をロードする](../loading/Load_to_Primary_Key_tables.md)を参照してください。[#12159](https://github.com/StarRocks/starrocks/pull/12159)
- クエリの中間計算結果を格納する[クエリキャッシュ](../using_starrocks/query_cache.md)をサポートし、高い同時実行性を持つ簡単なクエリのQPSを改善し、平均レイテンシーを短縮します。[#9194](https://github.com/StarRocks/starrocks/pull/9194)
- Broker Loadジョブの優先順位を指定することをサポートします。詳細は[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。[#11029](https://github.com/StarRocks/starrocks/pull/11029)
- StarRocksネイティブテーブルのデータロード用のレプリカ数を指定することをサポートします。詳細は[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11253](https://github.com/StarRocks/starrocks/pull/11253)
- [クエリキュー](../administration/query_queues.md)をサポートします。[#12594](https://github.com/StarRocks/starrocks/pull/12594)
- データロードによって占有される計算リソースを隔離し、データロードタスクのリソース消費を制限することをサポートします。詳細は[リソースグループ](../administration/resource_group.md)を参照してください。[#12606](https://github.com/StarRocks/starrocks/pull/12606)
- StarRocksネイティブテーブルに対するデータ圧縮アルゴリズム（LZ4、Zstd、Snappy、Zlib）の指定をサポートします。詳細は[データ圧縮](../table_design/data_compression.md)を参照してください。[#10097](https://github.com/StarRocks/starrocks/pull/10097) [#12020](https://github.com/StarRocks/starrocks/pull/12020)
- [ユーザー定義変数](../reference/user_defined_variables.md)をサポートします。[#10011](https://github.com/StarRocks/starrocks/pull/10011)
- [ラムダ式](../sql-reference/sql-functions/Lambda_expression.md)と以下の高階関数をサポートします：[array_map](../sql-reference/sql-functions/array-functions/array_map.md)、[array_sum](../sql-reference/sql-functions/array-functions/array_sum.md)、[array_sortby](../sql-reference/sql-functions/array-functions/array_sortby.md)。[#9461](https://github.com/StarRocks/starrocks/pull/9461) [#9806](https://github.com/StarRocks/starrocks/pull/9806) [#10323](https://github.com/StarRocks/starrocks/pull/10323) [#14034](https://github.com/StarRocks/starrocks/pull/14034)
- [ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)の結果をフィルタリングするQUALIFY句を提供します。[#13239](https://github.com/StarRocks/starrocks/pull/13239)
- uuid()関数とuuid_numeric()関数が返す結果を、テーブル作成時の列のデフォルト値として使用できるようにサポートします。詳細は[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11155](https://github.com/StarRocks/starrocks/pull/11155)
- 次の機能をサポートしています：[map_size](../sql-reference/sql-functions/map-functions/map_size.md)、[map_keys](../sql-reference/sql-functions/map-functions/map_keys.md)、[map_values](../sql-reference/sql-functions/map-functions/map_values.md)、[max_by](../sql-reference/sql-functions/aggregate-functions/max_by.md)、[sub_bitmap](../sql-reference/sql-functions/bitmap-functions/sub_bitmap.md)、[bitmap_to_base64](../sql-reference/sql-functions/bitmap-functions/bitmap_to_base64.md)、[host_name](../sql-reference/sql-functions/utility-functions/host_name.md)、および[date_slice](../sql-reference/sql-functions/date-time-functions/date_slice.md)。[#11299](https://github.com/StarRocks/starrocks/pull/11299) [#11323](https://github.com/StarRocks/starrocks/pull/11323) [#12243](https://github.com/StarRocks/starrocks/pull/12243) [#11776](https://github.com/StarRocks/starrocks/pull/11776) [#12634](https://github.com/StarRocks/starrocks/pull/12634) [#14225](https://github.com/StarRocks/starrocks/pull/14225)

### 改善点

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用して外部データをクエリする際のメタデータアクセスパフォーマンスを最適化しました。[#11349](https://github.com/StarRocks/starrocks/issues/11349)
- [Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を使用したARRAYデータのクエリをサポートします。[#9693](https://github.com/StarRocks/starrocks/pull/9693)
- マテリアライズドビューの以下の側面を最適化しました：
  - 非同期マテリアライズドビューは、SPJGタイプのマテリアライズドビューに基づく自動的かつ透過的なクエリリライトをサポートします。詳細は[マテリアライズドビュー](../using_starrocks/Materialized_view.md#rewrite-and-accelerate-queries-with-the-asynchronous-materialized-view)を参照してください。[#13193](https://github.com/StarRocks/starrocks/issues/13193)
  - 非同期マテリアライズドビューは、複数の非同期リフレッシュメカニズムをサポートします。詳細は[マテリアライズドビュー](../using_starrocks/Materialized_view.md#manually-refresh-an-asynchronous-materialized-view)を参照してください。[#12712](https://github.com/StarRocks/starrocks/pull/12712) [#13171](https://github.com/StarRocks/starrocks/pull/13171) [#13229](https://github.com/StarRocks/starrocks/pull/13229) [#12926](https://github.com/StarRocks/starrocks/pull/12926)
  - マテリアライズドビューの更新効率を向上しました。[#13167](https://github.com/StarRocks/starrocks/issues/13167)
- データロードの以下の側面を最適化しました：
  - 「シングルリーダーレプリケーション」モードをサポートすることで、マルチレプリカシナリオでのロードパフォーマンスを最適化しました。データロードにより、パフォーマンスが1倍向上します。"シングルリーダーレプリケーション"の詳細は[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)の`replicated_storage`を参照してください。[#10138](https://github.com/StarRocks/starrocks/pull/10138)
  - Broker LoadとSpark Loadは、1つのHDFSクラスターまたは1つのKerberosユーザーのみが設定されている場合、ブローカーに依存せずにデータロードが可能になりました。ただし、複数のHDFSクラスターや複数のKerberosユーザーがある場合は、ブローカーのデプロイが必要です。詳細は[HDFSまたはクラウドストレージからのデータロード](../loading/BrokerLoad.md)と[Apache Spark™を使用したバルクロード](../loading/SparkLoad.md)を参照してください。[#9049](https://github.com/starrocks/starrocks/pull/9049) [#9228](https://github.com/StarRocks/starrocks/pull/9228)
  - 多数の小さなORCファイルをロードする際のBroker Loadのパフォーマンスを最適化しました。[#11380](https://github.com/StarRocks/starrocks/pull/11380)
  - プライマリキーテーブルへのデータロード時のメモリ使用量を削減しました。
- `information_schema`データベースとその中の`tables`および`columns`テーブルを最適化しました。新しいテーブル`table_config`を追加しました。詳細は[Information Schema](../reference/overview-pages/information_schema.md)を参照してください。[#10033](https://github.com/StarRocks/starrocks/pull/10033)
- データバックアップとリストアを最適化しました：
  - データベース内の複数のテーブルからのデータのバックアップとリストアを一度にサポートします。詳細は[データのバックアップとリストア](../administration/Backup_and_restore.md)を参照してください。[#11619](https://github.com/StarRocks/starrocks/issues/11619)
  - プライマリキーテーブルからのデータのバックアップとリストアをサポートします。詳細はバックアップとリストアを参照してください。[#11885](https://github.com/StarRocks/starrocks/pull/11885)
- 以下の関数を最適化しました：
  - [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)関数に、時間間隔の開始または終了を返すかを決定するためのオプショナルパラメータを追加しました。[#11216](https://github.com/StarRocks/starrocks/pull/11216)
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)関数に新しいモード`INCREASE`を追加し、重複するタイムスタンプの計算を避けることができます。[#10134](https://github.com/StarRocks/starrocks/pull/10134)
  - [unnest](../sql-reference/sql-functions/array-functions/unnest.md)関数で複数の引数を指定できるようにサポートしました。[#12484](https://github.com/StarRocks/starrocks/pull/12484)
  - lead()関数とlag()関数は、HLLおよびBITMAPデータのクエリをサポートします。詳細は[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。[#12108](https://github.com/StarRocks/starrocks/pull/12108)
  - 以下のARRAY関数はJSONデータのクエリをサポートします：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、[array_sort](../sql-reference/sql-functions/array-functions/array_sort.md)、[array_concat](../sql-reference/sql-functions/array-functions/array_concat.md)、[array_slice](../sql-reference/sql-functions/array-functions/array_slice.md)、および[reverse](../sql-reference/sql-functions/array-functions/reverse.md)。[#13155](https://github.com/StarRocks/starrocks/pull/13155)
  - いくつかの関数の使用を最適化しました。`current_date`、`current_timestamp`、`current_time`、`localtimestamp`、および`localtime`関数は`()`を使用せずに実行できます。例えば、`select current_date;`を直接実行できます。[#14319](https://github.com/StarRocks/starrocks/pull/14319)
- FEログから不要な情報を削除しました。[#15374](https://github.com/StarRocks/starrocks/pull/15374)

### バグ修正

以下のバグが修正されました：

- append_trailing_char_if_absent()関数は、最初の引数が空の場合に誤った結果を返す可能性があります。[#13762](https://github.com/StarRocks/starrocks/pull/13762)
- RECOVERステートメントを使用してテーブルを復元した後、そのテーブルが存在しない問題。[#13921](https://github.com/StarRocks/starrocks/pull/13921)
- SHOW CREATE MATERIALIZED VIEWステートメントによって返される結果に、マテリアライズドビューの作成時にクエリステートメントで指定されたデータベースとカタログが含まれていない問題。[#12833](https://github.com/StarRocks/starrocks/pull/12833)
- `waiting_stable`状態のスキーマ変更ジョブをキャンセルできない問題。[#12530](https://github.com/StarRocks/starrocks/pull/12530)
- Leader FEと非Leader FEで`SHOW PROC '/statistic';`コマンドを実行すると異なる結果が返される問題。[#12491](https://github.com/StarRocks/starrocks/issues/12491)
- SHOW CREATE TABLEによって返される結果のORDER BY句の位置が正しくない問題。[#13809](https://github.com/StarRocks/starrocks/pull/13809)
- ユーザーがHiveカタログを使用してHiveデータをクエリする場合、FEによって生成された実行計画にパーティションIDが含まれていないと、BEはHiveパーティションデータのクエリに失敗します。[#15486](https://github.com/StarRocks/starrocks/pull/15486)。

### 動作変更

- パラメータ`AWS_EC2_METADATA_DISABLED`のデフォルト値を`False`に変更しました。これは、Amazon EC2のメタデータを取得してAWSリソースにアクセスすることを意味します。
- セッション変数`is_report_success`の名前を`enable_profile`に変更し、SHOW VARIABLESステートメントを使用してクエリできるようになりました。
- 予約語を4つ追加しました: `CURRENT_DATE`、`CURRENT_TIME`、`LOCALTIME`、`LOCALTIMESTAMP`。[#14319](https://github.com/StarRocks/starrocks/pull/14319)
- テーブル名とデータベース名の最大長は1023文字まで可能です。[#14929](https://github.com/StarRocks/starrocks/pull/14929) [#15020](https://github.com/StarRocks/starrocks/pull/15020)
- BEの設定項目`enable_event_based_compaction_framework`と`enable_size_tiered_compaction_strategy`はデフォルトで`true`に設定されており、タブレットの数が多い場合や単一のタブレットに大量のデータがある場合にコンパクションのオーバーヘッドを大幅に削減します。

### アップグレードノート

- クラスタは2.0.x、2.1.x、2.2.x、2.3.x、または2.4.xから2.5.0にアップグレード可能です。ただし、ロールバックが必要な場合は、2.4.xにのみロールバックすることを推奨します。
