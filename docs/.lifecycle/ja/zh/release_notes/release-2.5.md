---
displayed_sidebar: Chinese
---
# StarRocks バージョン 2.5

## 2.5.17

リリース日：2023年12月19日

### 新機能

- 監視メトリクス `max_tablet_rowset_num`（Rowsetの最大数を設定するためのもの）が追加されました。これにより、Compactionに問題が発生する可能性があるかどうかを事前に検出し、適切な対応を行うことができます。また、"too many versions"のエラーメッセージが減少します。[#36539](https://github.com/StarRocks/starrocks/pull/36539)
- 以下のBitmap関数が追加されました：[subdivide_bitmap](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/subdivide_bitmap/)。[#35817](https://github.com/StarRocks/starrocks/pull/35817)

### 機能の最適化

- [SHOW ROUTINE LOAD](https://docs.starrocks.io/zh/docs/sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD/)の結果に`OtherMsg`が追加され、最後の失敗したタスクの関連情報が表示されるようになりました。[#35806](https://github.com/StarRocks/starrocks/pull/35806)
- Trashファイルのデフォルトの有効期限が1日に調整されました（元は3日でした）。[#37113](https://github.com/StarRocks/starrocks/pull/37113)
- 主キーモデルテーブルのすべてのRowsetをCompactionする際の永続化インデックスの更新パフォーマンスが最適化され、I/O負荷が低減されました。[#36819](https://github.com/StarRocks/starrocks/pull/36819)
- 主キーモデルテーブルのCompaction Scoreの取得ロジックが最適化され、他のモデルのテーブルとの取得範囲が一貫して見えるようになりました。[#36534](https://github.com/StarRocks/starrocks/pull/36534)
- MySQL外部テーブルとJDBCカタログ外部テーブルのWHERE句にキーワードを含めることができるようになりました。[#35917](https://github.com/StarRocks/starrocks/pull/35917)
- Spark Loadにbitmap_from_binary関数が追加され、バイナリビットマップのインポートがサポートされました。[#36050](https://github.com/StarRocks/starrocks/pull/36050)
- bRPCのタイムアウト時間が1時間からセッション変数`query_timeout`で設定された時間に変更され、RPCのタイムアウトが長すぎてクエリが失敗することを防ぎます。[#36778](https://github.com/StarRocks/starrocks/pull/36778)

### 互換性の変更

#### パラメータの変更

- BEの設定項目`enable_stream_load_verbose_log`が追加されました。デフォルト値は`false`で、オンにするとログにStream LoadのHTTPリクエストとレスポンスの情報が記録され、問題が発生した場合のトラブルシューティングが容易になります。[#36113](https://github.com/StarRocks/starrocks/pull/36113)
- BEの設定項目`update_compaction_per_tablet_min_interval_seconds`が静的パラメータから動的パラメータに変更されました。[#36819](https://github.com/StarRocks/starrocks/pull/36819)

### 問題の修正

以下の問題が修正されました：

- ハッシュ結合時のクエリの失敗により、BEがクラッシュする可能性がありました。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- FEの設定項目`enable_collect_query_detail_info`を有効にすると、FEのパフォーマンスが著しく低下する場合がありました。[#35945](https://github.com/StarRocks/starrocks/pull/35945)
- 永続化インデックスをオープンした主キーモデルテーブルに大量のデータをインポートすると、エラーが発生する場合がありました。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- `./agentctl.sh stop be`を実行すると、starrocks_beプロセスが正常に終了しない場合がありました。[#35108](https://github.com/StarRocks/starrocks/pull/35108)
- ARRAY_DISTINCT関数がランダムにBEクラッシュする場合がありました。[#36377](https://github.com/StarRocks/starrocks/pull/36377)
- 特定の状況下で、マテリアライズドビューのリフレッシュ時にデッドロックが発生する場合がありました。[#35736](https://github.com/StarRocks/starrocks/pull/35736)
- 特定の特殊なシナリオで、動的パーティションの異常が発生すると、FEが再起動できなくなる場合がありました。[#36846](https://github.com/StarRocks/starrocks/pull/36846)

## 2.5.16

リリース日：2023年12月1日

### 問題の修正

以下の問題が修正されました：

- 特定のシナリオで、グローバルランタイムフィルターがBEのクラッシュを引き起こす可能性がありました。[#35776](https://github.com/StarRocks/starrocks/pull/35776)

## 2.5.15

リリース日：2023年11月29日

### 機能の最適化

- メトリクスのスローアクセスログが追加されました。[#33908](https://github.com/StarRocks/starrocks/pull/33908)
- ファイルが多い場合のSpark LoadによるParquet/Orcファイルの読み取りパフォーマンスが最適化されました。[#34787](https://github.com/StarRocks/starrocks/pull/34787)
- Bitmap関連の一部の操作のパフォーマンスが最適化されました。主に以下の点が含まれます：
  - ネストループ結合のパフォーマンスが最適化されました。[#340804](https://github.com/StarRocks/starrocks/pull/34804) [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor`関数のパフォーマンスが最適化されました。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - Copy on Write（COW）がサポートされ、パフォーマンスが最適化され、メモリ使用量が削減されました。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 互換性の変更

#### パラメータの変更

- FEの設定項目`enable_new_publish_mechanism`が静的パラメータに変更されました。変更後はFEを再起動する必要があります。[#35338](https://github.com/StarRocks/starrocks/pull/35338)

### 問題の修正

以下の問題が修正されました：

- Broker Loadジョブにフィルタ条件が含まれている場合、データのインポート中に一部の場合にBEがクラッシュすることがありました。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- レプリケーション操作のリプレイが失敗すると、FEがクラッシュする可能性がありました。[#32295](https://github.com/StarRocks/starrocks/pull/32295)
- `recover_with_empty_tablet`が`true`に設定されている場合、FEがクラッシュする可能性がありました。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- クエリ時に「get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction」というエラーが発生することがありました。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- ウィンドウ関数を含むクエリがBEのクラッシュを引き起こす可能性がありました。[#33671](https://github.com/StarRocks/starrocks/pull/33671)
- `show proc '/statistic'`を実行すると、一部の場合にフリーズすることがありました。[#34237](https://github.com/StarRocks/starrocks/pull/34237/files)
- 永続化インデックスをオープンした主キーモデルテーブルに大量のデータをインポートすると、エラーが発生することがありました。[#34566](https://github.com/StarRocks/starrocks/pull/34566)
- 2.4以前のバージョンからアップグレードすると、Compaction Scoreが非常に高くなる場合がありました。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- MariaDB ODBC Driverを使用して`INFORMATION_SCHEMA`の情報をクエリすると、`schemata`ビューの`CATALOG_NAME`列の値がすべて`null`になる場合がありました。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- Stream Loadのインポートジョブが**PREPARED**状態であり、同時にスキーマ変更が実行されている場合、データが失われる可能性がありました。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- HDFSのパスが2つ以上のスラッシュ(`/`)で終わる場合、HDFSのバックアップとリストアが失敗することがありました。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- クラスタでインポートタスクやクエリを実行すると、FEがフリーズすることがありました。[#34569](https://github.com/StarRocks/starrocks/pull/34569)

## 2.5.14

リリース日：2023年11月14日

### 機能の最適化

- `INFORMATION_SCHEMA.COLUMNS`テーブルがARRAY、MAP、STRUCT型のフィールドを表示するようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 互換性の変更

#### システム変数

- セッション変数`cbo_decimal_cast_string_strict`が追加されました。DECIMAL型をSTRING型に変換するための最適化制御を行います。値が`true`の場合、v2.5.x以降の処理ロジックが使用され、厳密な変換（スケールに基づく切り捨てと補完）が実行されます。値が`false`の場合、v2.5.x以前の処理ロジック（有効桁数に基づく）が保持されます。デフォルト値は`true`です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- セッション変数`cbo_eq_base_type`が追加されました。DECIMAL型とSTRING型のデータ比較時に強制的な型を指定します。デフォルトは`VARCHAR`で、`DECIMAL`を選択することもできます。[#34208](https://github.com/StarRocks/starrocks/pull/34208)

### 問題の修正

以下の問題が修正されました：

- 特定のシナリオで、ON条件にサブクエリが含まれるとエラーが発生する場合がありました：`java.lang.IllegalStateException: null`。[#30876](https://github.com/StarRocks/starrocks/pull/30876)
- INSERT INTO SELECT ... LIMITを実行した後、すぐにCOUNT(*)クエリを使用すると、異なるレプリカからの結果が一致しない場合がありました。[#24435](https://github.com/StarRocks/starrocks/pull/24435)
- cast()関数を使用してデータ型を変換する際、変換前後の型が同じ場合、一部の型ではBEがクラッシュする場合がありました。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- Broker Loadでデータをインポートする際、特定のパス形式では`msg:Fail to parse columnsFromPath, expected: [rec_dt]`というエラーが発生する場合がありました。[#32721](https://github.com/StarRocks/starrocks/issues/32721)
- 3.xバージョンにアップグレードする際、一部の列の型もアップグレードされている場合（例：DecimalからDecimal v3にアップグレード）、特定の特徴を持つテーブルはCompaction時にBEがクラッシュする場合がありました。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをインポートする際、高い並行性と制限されたHTTPおよびScanスレッド数の場合にフリーズする場合がありました。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlを呼び出すとBEがクラッシュする場合がありました。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- BITMAP型の列を持つ主キーモデルテーブルに列を追加すると、`Analyze columnDef error: No aggregate function specified for 'userid'`というエラーが発生しました。[#31763](https://github.com/StarRocks/starrocks/pull/31763)
- 永続化インデックスをオープンした主キーモデルテーブルに大量のデータをインポートすると、BEがクラッシュする場合がありました。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュを有効にした場合、クエリ結果が正しくありませんでした。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 主キーモデルを作成する際、ORDER BYの後のフィールドがNULLの場合、Compactionが実行されませんでした。[#29225](https://github.com/StarRocks/starrocks/pull/29225)
- 実行時間が長い複雑なJoinクエリで、"StarRocks planner use long time 10000 ms in logical phase"というエラーが発生する場合がありました。[#34177](https://github.com/StarRocks/starrocks/pull/34177)

## 2.5.13

リリース日：2023年9月28日

### 機能の最適化

- 窓関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMPがORDER BY句とWindow句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型のデータのクエリ結果が範囲外の場合、NULLではなくエラーが返されるようになりました。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- SQLコマンドに不正なコメントが含まれている場合、MySQLと同じ結果が返されます。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 削除されたTabletに対応するRowsetをクリーンアップし、BEの起動時に消費されるメモリを削減します。[#30625](https://github.com/StarRocks/starrocks/pull/30625)

### 問題の修正

以下の問題が修正されました：

- Spark ConnectorまたはFlink Connectorを使用してStarRocksデータを読み取る際に、「Set cancelled by MemoryScratchSinkOperator」というエラーが発生する問題を修正しました。[#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 聚合関数が含まれるORDER BY句で「java.lang.IllegalStateException: null」というエラーが発生する問題を修正しました。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- Inactiveなマテリアライズドビューがある場合、FEの再起動が失敗する問題を修正しました。[#30015](https://github.com/StarRocks/starrocks/pull/30015)
- 重複するパーティションが存在する場合、INSERT OVERWRITEによってメタデータが破損し、後続のFEの再起動が失敗する問題を修正しました。[#27545](https://github.com/StarRocks/starrocks/pull/27545)
- 存在しない列を主キーモデルテーブルで変更しようとすると、「java.lang.NullPointerException: null」というエラーが発生する問題を修正しました。[#30366](https://github.com/StarRocks/starrocks/pull/30366)
- パーティションを持つStarRocks外部テーブルにデータを書き込む際に「get TableMeta failed from TNetworkAddress」というエラーが発生する問題を修正しました。[#30124](https://github.com/StarRocks/starrocks/pull/30124)
- 特定のシナリオでCloudCanalによるデータのインポートがエラーになる問題を修正しました。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- Flink Connectorを使用して書き込みを行ったり、DELETE、INSERTなどの操作を実行する際に「current running txns on db xxx is 200, larger than limit 200」というエラーが発生する問題を修正しました。[#18393](https://github.com/StarRocks/starrocks/pull/18393)
- HAVING句に聚合関数を含むクエリで作成された非同期マテリアライズドビューのクエリの改変が正しく行われない問題を修正しました。[#29976](https://github.com/StarRocks/starrocks/pull/29976)

## 2.5.12

リリース日：2023年9月4日

### 機能の最適化

- Audit LogファイルにSQLのコメント情報が保持されるようになりました。[#29747](https://github.com/StarRocks/starrocks/pull/29747)
- Audit LogにINSERT INTO SELECTのCPUとメモリの統計情報が追加されました。[#29901](https://github.com/StarRocks/starrocks/pull/29901)

### 問題の修正

以下の問題が修正されました：

- Broker Loadを使用してデータをインポートする際に、一部のフィールドのNOT NULL制約がBEのクラッシュや「msg:mismatched row count」というエラーを引き起こす問題を修正しました。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- 上流のApache ORCのBugFix ORC-1304（[apache/orc#1299](https://github.com/apache/orc/pull/1299)）がマージされていないため、ORCファイルのクエリが失敗する問題を修正しました。[#29804](https://github.com/StarRocks/starrocks/pull/29804)
- 主キーモデルテーブルのリストア後、BEの再起動後にメタデータが破損し、メタデータが一貫しなくなる問題を修正しました。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 2.5.11

リリース日：2023年8月28日

### 機能の最適化

- すべての複合述語およびWHERE句の式に対して暗黙の型変換をサポートし、[セッション変数](https://docs.starrocks.io/zh-cn/latest/reference/System_variable) `enable_strict_type` を使用して暗黙の型変換を有効化または無効化できるようになりました（デフォルト値は `false`）。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- Iceberg Catalogの作成時に`hive.metastore.uri`が指定されていない場合のエラーメッセージがより正確になりました。[#16543](https://github.com/StarRocks/starrocks/issues/16543)
- 「xxx too many versions xxx」というエラーメッセージに対する処理方法の提案が追加されました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 動的パーティションに年の粒度をサポートしました。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

### 問題の修正

以下の問題が修正されました：

- 複数のレプリカを持つテーブルにデータをインポートする際、一部のパーティションにデータが存在しない場合、無駄なログが書き込まれる問題を修正しました。[#28824](https://github.com/StarRocks/starrocks/pull/28824)
- WHERE条件にBITMAPまたはHLLのフィールドタイプが含まれる場合、DELETEが失敗する問題を修正しました。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 非同期マテリアライズドビューのリフレッシュ戦略が手動リフレッシュであり、同期呼び出しのリフレッシュタスク（SYNC MODE）が実行された場合、手動リフレッシュ後に`information_schema.task_run`テーブルに複数のINSERT OVERWRITEレコードが存在する問題を修正しました。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- 特定のTabletが特定のERROR状態になった後にClone操作がトリガされると、ディスク使用率が上昇する問題を修正しました。[#28488](https://github.com/StarRocks/starrocks/pull/28488)
- Join Reorderを有効にした場合、クエリの列が定数の場合、クエリ結果が正しくならない問題を修正しました。[#29239](https://github.com/StarRocks/starrocks/pull/29239)
- ホットデータとコールドデータの移行時にタスクが過剰に発行され、BEがOOMになる問題を修正しました。[#29055](https://github.com/StarRocks/starrocks/pull/29055)
- `/apache_hdfs_broker/lib/log4j-1.2.17.jar`にセキュリティの脆弱性が存在する問題を修正しました。[#28866](https://github.com/StarRocks/starrocks/pull/28866)
- Hive Catalogのクエリで、WHERE句にパーティション列が使用され、OR条件が含まれる場合、クエリ結果が正しくならない問題を修正しました。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- クエリ実行時に「java.util.ConcurrentModificationException: null」というエラーが偶発的に発生する問題を修正しました。[#29296](https://github.com/StarRocks/starrocks/pull/29296)
- 非同期マテリアライズドビューの基になるテーブルが削除されると、FEの再起動時にエラーが発生する問題を修正しました。[#29318](https://github.com/StarRocks/starrocks/pull/29318)
- クロスデータベースの非同期マテリアライズドビューの基になるテーブルがデータを書き込む際に、FEリーダーがデッドロックになる場合がある問題を修正しました。[#29432](https://github.com/StarRocks/starrocks/pull/29432)

## 2.5.10

リリース日：2023年8月7日

### 新機能

- [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)の集計関数がサポートされました。
- [窓関数](../sql-reference/sql-functions/Window_function.md)COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMPがサポートされました。

### 機能の最適化

- TabletCheckerのスケジューリングロジックが最適化され、一時的に修復できないTabletの重複スケジューリングを回避するようになりました。[#27648](https://github.com/StarRocks/starrocks/pull/27648)
- Schema ChangeとRoutine loadが同時に実行される場合、Schema Changeが先に完了した場合にRoutine Loadの失敗時のエラーメッセージが改善されました。[#28425](https://github.com/StarRocks/starrocks/pull/28425)
- 外部テーブルの作成時にNot Null列を定義することを禁止しました（既にNot Null列が定義されている場合、アップグレード後にエラーが発生し、テーブルを再作成する必要があります）。2.3.0以降はCatalogを使用することをお勧めします。外部テーブルの使用は避けてください。[#25485](https://github.com/StarRocks/starrocks/pull/25441)
- Broker Loadのリトライ中にエラーが発生した場合のエラーメッセージが追加され、問題のトラブルシューティングが容易になりました。[#21982](https://github.com/StarRocks/starrocks/pull/21982)
- 主キーモデルのインポート時に、UPSERTとDELETEを含む大量のデータの書き込みもサポートされるようになりました。[#17264](https://github.com/StarRocks/starrocks/pull/17264)
- マテリアライズドビューの改変機能が最適化されました。[#27934](https://github.com/StarRocks/starrocks/pull/27934) [#25542](https://github.com/StarRocks/starrocks/pull/25542) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#27557](https://github.com/StarRocks/starrocks/pull/27557) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#26957](https://github.com/StarRocks/starrocks/pull/26957) [#27728](https://github.com/StarRocks/starrocks/pull/27728) [#27900](https://github.com/StarRocks/starrocks/pull/27900)

### 問題の修正

以下の問題が修正されました：

- cast関数を使用して文字列を配列に変換する際、入力に定数が含まれている場合に正しい結果が返らない問題を修正しました。[#19793](https://github.com/StarRocks/starrocks/pull/19793)
- ORDER BYとLIMITが含まれる場合、SHOW TABLETの結果が正しくない問題を修正しました。[#23375](https://github.com/StarRocks/starrocks/pull/23375)
- マテリアライズドビューのOuter joinとAnti joinの改変が正しく行われない問題を修正しました。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- FEのテーブルレベルのスキャン統計情報が誤っているため、テーブルのクエリとインポートのメトリクス情報が正しくない問題を修正しました。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- HMSでイベントリスナーを設定してHiveメタデータを自動的に増分更新する場合、FEログに「An exception occurred when using the current long link to access metastore. msg: Failed to get next notification based on last event id: 707602」というエラーが表示される問題を修正しました。[#21056](https://github.com/StarRocks/starrocks/pull/21056)
- パーティションテーブルでソートキー列を変更した後、クエリ結果が安定しない問題を修正しました。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- DATE、DATETIME、DECIMALをバケット列として使用した場合、Spark Loadによるデータのインポートが誤ったバケットにインポートされる問題を修正しました。[#27005](https://github.com/StarRocks/starrocks/pull/27005)
- 特定の場合にregex_replace関数がBEのクラッシュを引き起こす問題を修正しました。[#27117](https://github.com/StarRocks/starrocks/pull/27117)
- sub_bitmap関数の引数がBITMAP型でない場合にBEのクラッシュが発生する問題を修正しました。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- Join Reorderを有効にした場合、特定のクエリでunknown errorが発生する問題を修正しました。[#27472](https://github.com/StarRocks/starrocks/pull/27472)
- 主キーモデルの一部の列を更新する際に平均行サイズの推定が正しくないため、過剰なメモリ使用量が発生する問題を修正しました。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 低基数最適化が有効になっている場合、特定の場合にINSERTのインポートで「[42000][1064] Dict Decode failed, Dict can't take cover all key :0」というエラーが発生する問題を修正しました。[#26463](https://github.com/StarRocks/starrocks/pull/26463)
- Broker Loadを使用してHDFSからデータをインポートする際、ジョブの認証方式（`hadoop.security.authentication`）が`simple`に設定されている場合にジョブが失敗する問題を修正しました。[#27774](https://github.com/StarRocks/starrocks/pull/27774)
- マテリアライズドビューのリフレッシュモードを変更すると、メタデータが一貫しなくなる問題を修正しました。[#28082](https://github.com/StarRocks/starrocks/pull/28082) [#28097](https://github.com/StarRocks/starrocks/pull/28097)
- SHOW CREATE CATALOG、SHOW RESOURCESで特定の情報を表示する際に、PASSWORDが非表示にならない問題を修正しました。[#28059](https://github.com/StarRocks/starrocks/pull/28059)
- LabelCleanerスレッドがハングし、FEのメモリリークが発生する問題を修正しました。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 2.5.9

```
发布日：2023年7月19日

### 新機能

- クエリとマテリアライズドビューの結合タイプが異なる場合でも、クエリの書き換えをサポートします。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 機能の最適化

- 現在のクラスタがターゲットクラスタと同じ場合、StarRocks外部テーブルの作成を禁止します。[#25441](https://github.com/StarRocks/starrocks/pull/25441)
- クエリのフィールドがマテリアライズドビューの出力列に含まれていないが、述語条件には含まれている場合でも、マテリアライズドビューを使用してクエリを書き換えることができます。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- `Information_schema.tables_config` テーブルに `table_id` フィールドを追加しました。`table_id` フィールドを使用して、`Information_schema` データベースの `tables_config` テーブルと `be_tablets` テーブルを関連付けて、タブレットの所属データベースとテーブルをクエリできます。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### 問題の修正

以下の問題を修正しました：

- 明細モデルテーブルのCount Distinctの結果が異常でした。[#24222](https://github.com/StarRocks/starrocks/pull/24222)
- Join列がBINARY型であり、サイズが大きい場合、BEがクラッシュする問題を修正しました。[#25084](https://github.com/StarRocks/starrocks/pull/25084)
- データの挿入時に、テーブルのCHARの定義長を超える長さのデータを挿入すると、挿入が応答しない問題を修正しました。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- Coalesce関数のクエリ結果が正しくありませんでした。[#26250](https://github.com/StarRocks/starrocks/pull/26250)
- リストア後、同じタブレットのBEとFEのバージョンが一致しない問題を修正しました。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- リカバリ時にテーブルの自動パーティション作成が失敗する問題を修正しました。[#26813](https://github.com/StarRocks/starrocks/pull/26813)

## 2.5.8

リリース日：2023年6月30日

### 機能の最適化

- パーティションのないテーブルにパーティションを追加する際のエラーメッセージを最適化しました。[#25266](https://github.com/StarRocks/starrocks/pull/25266)
- テーブルの[データ分布](../table_design/Data_distribution.md#确定分桶数量)を最適化しました。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- テーブル作成時のコメントのデフォルト値を最適化しました。[#24803](https://github.com/StarRocks/starrocks/pull/24803)
- 非同期マテリアライズドビューの手動リフレッシュ戦略を最適化しました。REFRESH MATERIALIZED VIEW WITH SYNC MODEを使用してマテリアライズドビューのリフレッシュタスクを同期的に呼び出すことができます。[#25910](https://github.com/StarRocks/starrocks/pull/25910)

### 問題の修正

以下の問題を修正しました：

- 非同期マテリアライズドビューの作成時にUnionが含まれていると、Countの結果が正確でない問題がありました。[#24460](https://github.com/StarRocks/starrocks/issues/24460)
- rootパスワードの強制リセット時に「Unknown error」というエラーが表示される問題を修正しました。[#25492](https://github.com/StarRocks/starrocks/pull/25492)
- クラスタが3つのAlive BEを満たしていない場合、INSERT OVERWRITEのエラーメッセージが正確でない問題を修正しました。[#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 2.5.7

リリース日：2023年6月14日

### 新機能

- 基底テーブルが削除されて無効になったマテリアライズドビューをアクティブにするために、`ALTER MATERIALIZED VIEW <mv_name> ACTIVE` を使用することができるようになりました。詳細については、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を参照してください。[#24001](https://github.com/StarRocks/starrocks/pull/24001)

- テーブルの作成およびパーティションの追加時に、適切なバケット数を自動的に設定する機能が追加されました。詳細については、[データ分布の決定](../table_design/Data_distribution.md#确定分桶数量)を参照してください。[#10614](https://github.com/StarRocks/starrocks/pull/10614)

### 機能の最適化

- 外部テーブルのスキャンノードのI/O並行数を最適化し、メモリ使用量を減らし、メモリ使用比率を制限して外部テーブルのインポートの安定性を向上させました。[#23617](https://github.com/StarRocks/starrocks/pull/23617) [#23624](https://github.com/StarRocks/starrocks/pull/23624) [#23626](https://github.com/StarRocks/starrocks/pull/23626)
- Broker Loadのエラーメッセージを最適化し、エラーファイルの名前のヒントと再試行情報を追加しました。[#18038](https://github.com/StarRocks/starrocks/pull/18038) [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- CREATE TABLEのタイムアウトエラーメッセージを最適化し、パラメータの調整提案を追加しました。[#24510](https://github.com/StarRocks/starrocks/pull/24510)
- テーブルの状態がNormalでないためALTER TABLEが失敗する場合のエラーメッセージを最適化しました。[#24381](https://github.com/StarRocks/starrocks/pull/24381)
- テーブル作成時に中国語のスペースを無視するようにしました。[#23885](https://github.com/StarRocks/starrocks/pull/23885)
- Brokerのアクセスタイムアウト時間を最適化し、Broker Loadのインポート失敗率を低下させました。[#22699](https://github.com/StarRocks/starrocks/pull/22699)
- 主キーモデルのSHOW TABLETの`VersionCount`フィールドに、保留中のRowsetsの状態も含まれるようにしました。[#23847](https://github.com/StarRocks/starrocks/pull/23847)
- 永続インデックスの戦略を最適化しました。[#22140](https://github.com/StarRocks/starrocks/pull/22140)

### 問題の修正

以下の問題を修正しました：

- StarRocksにParquet形式のデータをインポートする際、日付型の変換オーバーフローによりデータが正しくない問題を修正しました。[#22356](https://github.com/StarRocks/starrocks/pull/22356)
- 動的パーティションを無効にした後、バケット情報が失われる問題を修正しました。[#22595](https://github.com/StarRocks/starrocks/pull/22595)
- サポートされていないプロパティを指定してパーティションテーブルを作成すると、ヌルポインタエラーが発生する問題を修正しました。[#21374](https://github.com/StarRocks/starrocks/issues/21374)
- Information Schemaのテーブル権限フィルタが機能しない問題を修正しました。[#23804](https://github.com/StarRocks/starrocks/pull/23804)
- SHOW TABLE STATUSの結果が完全に表示されない問題を修正しました。[#24279](https://github.com/StarRocks/starrocks/issues/24279)
- スキーマ変更とデータインポートが同時に行われる場合、スキーマ変更が時々スタックする問題を修正しました。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- RocksDB WALフラッシュの待機により、brpcワーカーが同期待ちになり、他のbthreadの処理に切り替えることができず、主キーモデルの高頻度インポートでブレークポイントが発生する問題を修正しました。[#22489](https://github.com/StarRocks/starrocks/pull/22489)
- テーブル作成時に、不正なデータ型のTIME列を作成できる問題を修正しました。[#23474](https://github.com/StarRocks/starrocks/pull/23474)
- マテリアライズドビューのUnionクエリの書き換えに失敗する問題を修正しました。[#22922](https://github.com/StarRocks/starrocks/pull/22922)

## 2.5.6

リリース日：2023年5月19日

### 機能の最適化

- `thrift_server_max_worker_threads`が小さすぎるため、INSERT INTO SELECTがタイムアウトする場合のエラーメッセージを最適化しました。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- CTASで作成されたテーブルは通常のテーブルと同じで、デフォルトで3つのレプリカがあります。[#22854](https://github.com/StarRocks/starrocks/pull/22854)

### 問題の修正

- Truncate操作はパーティション名の大文字と小文字を区別するため、Truncate Partitionが失敗する問題を修正しました。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- マテリアライズドビューの作成時に一時パーティションの作成に失敗し、BEがオフラインになる問題を修正しました。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- FEパラメータの動的変更で、空の配列を設定することができない問題を修正しました。[#22225](https://github.com/StarRocks/starrocks/pull/22225)
- `partition_refresh_number`プロパティが設定されているマテリアライズドビューが完全にリフレッシュされない場合がある問題を修正しました。[#21619](https://github.com/StarRocks/starrocks/pull/21619)
- SHOW CREATE TABLEが認証情報を誤ってメモリに表示する問題を修正しました。[#21311](https://github.com/StarRocks/starrocks/pull/21311)
- 外部テーブルのクエリで、一部のORCファイルでは述語が無効になる問題を修正しました。[#21901](https://github.com/StarRocks/starrocks/pull/21901)
- フィルタ条件が列名の大文字と小文字を正しく処理できない問題を修正しました。[#22626](https://github.com/StarRocks/starrocks/pull/22626)
- 遅延マテリアライズドビューにより、複雑なデータ型（STRUCTまたはMAP）のクエリがエラーになる問題を修正しました。[#22862](https://github.com/StarRocks/starrocks/pull/22862)
- バックアップの復元中に発生する主キーモデルテーブルの問題を修正しました。[#23384](https://github.com/StarRocks/starrocks/pull/23384)

## 2.5.5

リリース日：2023年4月28日

### 新機能

主キーモデルテーブルのタブレットの状態を監視する機能が追加されました。以下の機能が含まれます：

- FEに`err_state_metric`モニタリング項目が追加されました。
- `SHOW PROC '/statistic/'`の結果に、エラーステート（err_state）のタブレット数を統計するための`ErrorStateTabletNum`列が追加されました。
- `SHOW PROC '/statistic/<db_id>/'`の結果に、エラーステートのタブレットIDを表示するための`ErrorStateTablets`列が追加されました。[#19517](https://github.com/StarRocks/starrocks/pull/19517)

詳細については、[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)を参照してください。

### 機能の最適化

- 複数のBEを追加する際のディスクバランスの速度を最適化しました。[#19418](https://github.com/StarRocks/starrocks/pull/19418)
- `storage_medium`の推論メカニズムを最適化しました。BEがSSDとHDDを同時にストレージメディアとして使用する場合、`storage_cooldown_time`の設定に基づいてデフォルトのストレージタイプを決定します。`storage_cooldown_time`が設定されている場合、StarRocksは`storage_medium`を`SSD`に設定します。設定されていない場合は、`storage_medium`を`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- Unique KeyテーブルのValue列の統計情報の収集を禁止することで、Unique Keyテーブルのパフォーマンスを最適化しました。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### 問題の修正

以下の問題を修正しました：

- Colocationテーブルでは、コマンドを使用してレプリカのステータスをbadに手動で設定することができます：`ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`、BEの数がレプリカの数以下の場合、そのレプリカは修復できない問題を修正しました。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- BEを起動した後、プロセスは存在しますが、ポートが起動しない問題を修正しました。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- サブクエリでウィンドウ関数を使用した場合、集計結果が正確でない問題を修正しました。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 初回のマテリアライズドビューのリフレッシュ時に、`auto_refresh_partitions_limit`の制限が効かず、すべてのパーティションがリフレッシュされる問題を修正しました。[#19759](https://github.com/StarRocks/starrocks/issues/19759)
- HiveテーブルのCSV形式のクエリで、ARRAY配列に複雑なデータ型（MAPおよびSTRUCT）がネストされているために発生する問題を修正しました。[#20233](https://github.com/StarRocks/starrocks/pull/20233)
- Sparkコネクタを使用したクエリのタイムアウト問題を修正しました。[#20264](https://github.com/StarRocks/starrocks/pull/20264)
- 两副本的テーブルのうち、1つの副本に問題が発生した場合、自動修復は行われません。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- マテリアライズドビューのクエリの書き換えに失敗し、クエリが失敗します。[#19549](https://github.com/StarRocks/starrocks/issues/19549)
- DBロックによるメトリクスインターフェースのタイムアウト。[#20790](https://github.com/StarRocks/starrocks/pull/20790)
- ブロードキャストジョインのクエリ結果が間違っています。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- テーブル作成時にサポートされていないデータ型を使用すると、ヌルポインタが返されます。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- クエリキャッシュを有効にした状態でwindow_funnel関数を使用すると問題が発生します。[#21474](https://github.com/StarRocks/starrocks/issues/21474)
- CTEの最適化クエリの書き換えにより、最適化プランの選択に時間がかかります。[#16515](https://github.com/StarRocks/starrocks/pull/16515)

## 2.5.4

リリース日：2023年4月4日

### 機能の最適化

- クエリプランニングフェーズでのマテリアライズドビューのクエリの書き換えのパフォーマンスを最適化し、プランニング時間を約70%削減しました。[#19579](https://github.com/StarRocks/starrocks/pull/19579)
- 型の推論を最適化し、クエリ `SELECT sum(CASE WHEN XXX)FROM xxx;` に定数 `0` が含まれている場合、例えば `SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;` のような場合、自動的にプリアグリゲーションが有効になり、クエリの高速化が行われます。[#19474](https://github.com/StarRocks/starrocks/pull/19474)
- `SHOW CREATE VIEW` を使用してマテリアライズドビューの作成文を表示することができるようになりました。[#19999](https://github.com/StarRocks/starrocks/pull/19999)
- BEノード間の単一のbRPCリクエストで2GBを超えるデータパケットの転送がサポートされるようになりました。[#20283](https://github.com/StarRocks/starrocks/pull/20283) [#20230](https://github.com/StarRocks/starrocks/pull/20230)
- External Catalogを使用して、[SHOW CREATE CATALOG](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を介してCatalogの作成情報を表示できるようになりました。

### 問題の修正

以下の問題が修正されました：

- マテリアライズドビューのクエリの書き換え後、低基数グローバル辞書の最適化が効果を発揮しませんでした。[#19615](https://github.com/StarRocks/starrocks/pull/19615)
- マテリアライズドビューのクエリの書き換えができず、クエリが失敗します。[#19774](https://github.com/StarRocks/starrocks/pull/19774)
- 主キーモデルまたは更新モデルのテーブルでマテリアライズドビューを作成すると、マテリアライズドビューのクエリの書き換えができません。[#19600](https://github.com/StarRocks/starrocks/pull/19600)
- マテリアライズドビューの列名は大文字小文字を区別し、テーブル作成時の`PROPERTIES`で列名の大文字小文字が間違っている場合、テーブル作成は成功しますが、エラーメッセージが返されず、そのテーブルを基にしたマテリアライズドビューのクエリの書き換えができません。[#19780](https://github.com/StarRocks/starrocks/pull/19780)
- マテリアライズドビューのクエリの書き換え後、実行計画にパーティション列に基づく無効な述語が生成され、クエリのパフォーマンスに影響があります。[#19784](https://github.com/StarRocks/starrocks/pull/19784)
- 新しく作成したパーティションにデータをインポートした後、マテリアライズドビューのクエリの書き換えができない場合があります。[#20323](https://github.com/StarRocks/starrocks/pull/20323)
- マテリアライズドビューの作成時に `"storage_medium" = "SSD"` を設定すると、マテリアライズドビューのリフレッシュが失敗します。[#19539](https://github.com/StarRocks/starrocks/pull/19539) [#19626](https://github.com/StarRocks/starrocks/pull/19626)
- 主キーモデルのテーブルは並列Compactionされる可能性があります。[#19692](https://github.com/StarRocks/starrocks/pull/19692)
- 大量のDELETE操作の後、Compactionが適切に行われません。[#19623](https://github.com/StarRocks/starrocks/pull/19623)
- ステートメントの式に複数の低基数列が含まれている場合、式の書き換えが失敗し、低基数グローバル辞書の最適化が効果を発揮しません。[#20161](https://github.com/StarRocks/starrocks/pull/20161)

## 2.5.3

リリース日：2023年3月10日

### 機能の最適化

- マテリアライズドビューのクエリの書き換えを最適化しました：
  - Outer JoinとCross Joinのクエリの書き換えをサポートしました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - マテリアライズドビューデータのスキャンロジックを最適化し、クエリのさらなる高速化を実現しました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 単一テーブルの集計クエリの書き換え能力を強化しました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - View Deltaシナリオの書き換え能力を強化しました。つまり、クエリの関連テーブルがマテリアライズドビューの関連テーブルのサブセットである場合の書き換え能力を強化しました。[#18800](https://github.com/StarRocks/starrocks/pull/18800)
- フィルタ条件やソートキーとしてRankウィンドウ関数を使用する場合のパフォーマンスとメモリ使用量を最適化しました。[#17553](https://github.com/StarRocks/starrocks/issues/17553)

### 問題の修正

以下の問題が修正されました：

- ARRAY型の空リテラルによる問題。[#18563](https://github.com/StarRocks/starrocks/pull/18563)
- 複雑なクエリシナリオでは、低基数辞書が誤って使用される可能性がありました。このようなエラーを回避するために、低基数辞書の最適化前にチェックを追加しました。[#17318](https://github.com/StarRocks/starrocks/pull/17318)
- 単一のBE環境でのLocal Shuffleにより、GROUP BYに重複した結果が含まれる問題。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- **非パーティション**マテリアライズドビューの作成時に**パーティション**関連のパラメータが誤って使用される問題。非パーティションマテリアライズドビューを作成する場合、パーティション関連のパラメータは自動的に無効になるようにマテリアライズドビューの作成チェックが追加されました。[#18741](https://github.com/StarRocks/starrocks/pull/18741)
- ParquetのRepetition Columnの解析問題。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 列のnullable情報の取得エラー。CTASを使用して主キーモデルテーブルを作成する場合、主キー列のみをnon-nullableに設定し、非主キー列をnullableに設定します。[#16431](https://github.com/StarRocks/starrocks/pull/16431)
- 主キーモデルテーブルのデータ削除による問題。[#18768](https://github.com/StarRocks/starrocks/pull/18768)

## 2.5.2

リリース日：2023年2月21日

### 新機能

- AWS S3およびAWS Glueへのアクセス時に、インスタンスプロファイルとアサムロールを使用した認証と認可がサポートされました。[#15958](https://github.com/StarRocks/starrocks/pull/15958)
- 3つのbit関数が追加されました：[bit_shift_left](../sql-reference/sql-functions/bit-functions/bit_shift_left.md)、[bit_shift_right](../sql-reference/sql-functions/bit-functions/bit_shift_right.md)、[bit_shift_right_logical](../sql-reference/sql-functions/bit-functions/bit_shift_right_logical.md)。[#14151](https://github.com/StarRocks/starrocks/pull/14151)

### 機能の最適化

- クエリに含まれる大量の集計クエリの場合、一時的なメモリ使用量を大幅に削減するため、一部のメモリ解放ロジックを最適化しました。[#16913](https://github.com/StarRocks/starrocks/pull/16913)
- ソートのメモリ使用量を最適化し、ウィンドウ関数を含む一部のクエリやソートクエリのメモリ消費量を50%以上削減することができます。[#16937](https://github.com/StarRocks/starrocks/pull/16937) [#17362](https://github.com/StarRocks/starrocks/pull/17362) [#17408](https://github.com/StarRocks/starrocks/pull/17408)

### 問題の修正

以下の問題が修正されました：

- MAPまたはARRAYデータ型を含むApache Hive外部テーブルのリフレッシュができない問題。[#17548](https://github.com/StarRocks/starrocks/pull/17548)
- Supersetがマテリアライズドビューの列の型を認識できない問題。[#17686](https://github.com/StarRocks/starrocks/pull/17686)
- BIとの連携時にSET GLOBAL/SESSION TRANSACTIONを解析できないために接続性の問題が発生する問題。[#17295](https://github.com/StarRocks/starrocks/pull/17295)
- コロケートグループ内のダイナミックパーティションテーブルのバケット数を変更できず、エラーメッセージが返される問題。[#17418](https://github.com/StarRocks/starrocks/pull/17418/)
- プリペアフェーズでの失敗による潜在的な問題を修正しました。[#17323](https://github.com/StarRocks/starrocks/pull/17323)

### 動作の変更

- FEパラメータ`enable_experimental_mv`のデフォルト値を`true`に変更し、非同期マテリアライズドビュー機能をデフォルトで有効にしました。
- CHARACTERキーワードが追加されました。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.1

リリース日：2023年2月5日

### 機能の最適化

- 外部テーブルのマテリアライズドビューのクエリ書き換えをサポートしました。[#11116](https://github.com/StarRocks/starrocks/issues/11116)[#15791](https://github.com/StarRocks/starrocks/issues/15791)
- CBO自動フルコレクションが、クラスタのパフォーマンスの揺れを防ぐために、ユーザーが収集時間帯を設定できるようになりました。[#14996](https://github.com/StarRocks/starrocks/pull/14996)
- INSERT INTO SELECT時にThriftサーバーのリクエストが過度にビジーになることを防ぐために、Thriftサーバーキューが追加されました。[#14571](https://github.com/StarRocks/starrocks/pull/14571)
- テーブル作成時に`storage_medium`プロパティを明示的に指定しなかった場合、システムはBEノードのディスクタイプに基づいてテーブルのストレージタイプを自動的に推論および設定します。[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)のパラメータの説明を参照してください。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### 問題の修正

以下の問題が修正されました：

- SET PASSWORDによるNullPointerExceptionの問題。[#15247](https://github.com/StarRocks/starrocks/pull/15247)
- KEYが空の場合にJSONデータを解析できない問題。[#16852](https://github.com/StarRocks/starrocks/pull/16852)
- 不正なデータ型をARRAY型に変換できる問題。[#16866](https://github.com/StarRocks/starrocks/pull/16866)
- 異常なシナリオでNested Loop Joinが中断できない問題。[#16875](https://github.com/StarRocks/starrocks/pull/16875)

### 動作の変更

- FEパラメータ`default_storage_medium`をキャンセルし、テーブルのストレージメディアはシステムが自動的に推論するように変更しました。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.5.0

リリース日：2023年1月22日

### 新機能

- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)と[Hudi外部テーブル](../data_source/External_table.md#deprecated-hudi-外部テーブル)でMerge On Readテーブルのクエリをサポートしました。[#6780](https://github.com/StarRocks/starrocks/pull/6780)
- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、Hudiカタログ、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)でSTRUCTおよびMAP型のデータをクエリできるようになりました。[#10677](https://github.com/StarRocks/starrocks/issues/10677)
```
- [Data Cache](../data_source/data_cache.md)機能を提供し、外部のHDFSやオブジェクトストレージ上のホットデータのクエリを効率的に最適化します。[#11597](https://github.com/StarRocks/starrocks/pull/11579)
- [Delta Lake catalog](../data_source/catalog/deltalake_catalog.md)をサポートし、データのインポートや外部テーブルの作成なしでDelta Lakeデータをクエリできます。[#11972](https://github.com/StarRocks/starrocks/issues/11972)
- Hive catalog、Hudi catalog、Iceberg catalogはAWS Glueと互換性があります。[#12249](https://github.com/StarRocks/starrocks/issues/12249)
- [ファイル外部テーブル](../data_source/file_external_table.md)を使用して、HDFSやオブジェクトストレージ上のParquetおよびORCファイルをクエリできます。[#13064](https://github.com/StarRocks/starrocks/pull/13064)
- Hive、Hudi、またはIceberg catalogを使用してマテリアライズドビューを作成し、マテリアライズドビューを基にしたマテリアライズドビューを作成できます。詳細なドキュメントは[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#11873](https://github.com/StarRocks/starrocks/pull/11873)
- プライマリキーモデルテーブルでの条件付き更新をサポートします。詳細なドキュメントは[データの変更による条件付き更新](../loading/Load_to_Primary_Key_tables.md#条件更新)を参照してください。[#12159](https://github.com/StarRocks/starrocks/pull/12159)
- [クエリキャッシュ](../using_starrocks/query_cache.md)をサポートし、クエリの中間計算結果を保存することで、シンプルな高並行クエリのQPSを向上させ、平均レイテンシを低下させます。[#9194](https://github.com/StarRocks/starrocks/pull/9194)
- Broker Loadジョブに優先度を指定する機能をサポートします。詳細なドキュメントは[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。[#11029](https://github.com/StarRocks/starrocks/pull/11029)
- StarRocksネイティブテーブルにデータのインポートのレプリカ数を手動で設定できる機能をサポートします。詳細なドキュメントは[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11253](https://github.com/StarRocks/starrocks/pull/11253)
- クエリキュー機能をサポートします。詳細なドキュメントは[クエリキュー](../administration/query_queues.md)を参照してください。[#12594](https://github.com/StarRocks/starrocks/pull/12594)
- リソースグループを使用してインポート計算をリソースで隔離し、インポートタスクがクラスタリソースを間接的に消費することを制御できる機能をサポートします。詳細なドキュメントは[リソースの隔離](../administration/resource_group.md)を参照してください。[#12606](https://github.com/StarRocks/starrocks/pull/12606)
- StarRocksネイティブテーブルにデータの圧縮アルゴリズム（LZ4、Zstd、Snappy、Zlib）を手動で設定する機能をサポートします。詳細なドキュメントは[データの圧縮](../table_design/data_compression.md)を参照してください。[#10097](https://github.com/StarRocks/starrocks/pull/10097) [#12020](https://github.com/StarRocks/starrocks/pull/12020)
- [ユーザー定義変数](../reference/user_defined_variables.md)（User-Defined Variables）をサポートします。[#10011](https://github.com/StarRocks/starrocks/pull/10011)
- [Lambda式](../sql-reference/sql-functions/Lambda_expression.md)および高階関数（[array_map](../sql-reference/sql-functions/array-functions/array_map.md)、[array_sum](../sql-reference/sql-functions/array-functions/array_sum.md)、[array_sortby](../sql-reference/sql-functions/array-functions/array_sortby.md)）をサポートします。[#9461](https://github.com/StarRocks/starrocks/pull/9461) [#9806](https://github.com/StarRocks/starrocks/pull/9806) [#10323](https://github.com/StarRocks/starrocks/pull/10323) [#14034](https://github.com/StarRocks/starrocks/pull/14034)
- [ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)でQUALIFYを使用してクエリ結果をフィルタリングすることができます。[#13239](https://github.com/StarRocks/starrocks/pull/13239)
- テーブル作成時に、uuidまたはuuid_numeric関数の結果を列のデフォルト値として指定できるようになりました。詳細なドキュメントは[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11155](https://github.com/StarRocks/starrocks/pull/11155)
- 次の関数が追加されました：[map_size](../sql-reference/sql-functions/map-functions/map_size.md)、[map_keys](../sql-reference/sql-functions/map-functions/map_keys.md)、[map_values](../sql-reference/sql-functions/map-functions/map_values.md)、[max_by](../sql-reference/sql-functions/aggregate-functions/max_by.md)、[sub_bitmap](../sql-reference/sql-functions/bitmap-functions/sub_bitmap.md)、[bitmap_to_base64](../sql-reference/sql-functions/bitmap-functions/bitmap_to_base64.md)、[host_name](../sql-reference/sql-functions/utility-functions/host_name.md)、[date_slice](../sql-reference/sql-functions/date-time-functions/date_slice.md)。[#11299](https://github.com/StarRocks/starrocks/pull/11299) [#11323](https://github.com/StarRocks/starrocks/pull/11323) [#12243](https://github.com/StarRocks/starrocks/pull/12243) [#11776](https://github.com/StarRocks/starrocks/pull/11776) [#12634](https://github.com/StarRocks/starrocks/pull/12634) [#14225](https://github.com/StarRocks/starrocks/pull/14225)

### 機能の最適化

- [Hive catalog](../data_source/catalog/hive_catalog.md)、[Hudi catalog](../data_source/catalog/hudi_catalog.md)、[Iceberg catalog](../data_source/catalog/iceberg_catalog.md)のメタデータアクセス速度が最適化されました。[#11349](https://github.com/StarRocks/starrocks/issues/11349)
- [Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-外部表)は、ARRAY型のデータをクエリできるようになりました。[#9693](https://github.com/StarRocks/starrocks/pull/9693)
- マテリアライズドビューの最適化
  - 非同期マテリアライズドビューは、SPJGタイプのマテリアライズドビュークエリの自動透明改変をサポートしています。詳細なドキュメントは[マテリアライズドビュー](../using_starrocks/Materialized_view.md#使用异步物化视图改写加速查询)を参照してください。[#13193](https://github.com/StarRocks/starrocks/issues/13193)
  - 非同期マテリアライズドビューは、複数の非同期リフレッシュメカニズムをサポートしています。詳細なドキュメントは[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。[#12712](https://github.com/StarRocks/starrocks/pull/12712) [#13171](https://github.com/StarRocks/starrocks/pull/13171) [#13229](https://github.com/StarRocks/starrocks/pull/13229) [#12926](https://github.com/StarRocks/starrocks/pull/12926)
  - マテリアライズドビューのリフレッシュ効率が最適化されました。[#13167](https://github.com/StarRocks/starrocks/issues/13167)
- インポートの最適化
  - 複数のレプリカのインポートパフォーマンスが最適化され、シングルリーダーレプリケーションモードがサポートされ、インポートパフォーマンスが2倍向上しました。このモードの詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)の`replicated_storage`パラメータを参照してください。[#10138](https://github.com/StarRocks/starrocks/pull/10138)
  - 単一のHDFSクラスタまたは単一のKerberosユーザーの場合、ブローカープロセスをデプロイせずにBroker LoadまたはSpark Loadを使用してデータをインポートできるようになりました。複数のHDFSクラスタまたは複数のKerberosユーザーを構成している場合は、引き続きBrokerプロセスを使用してインポートを実行する必要があります。詳細なドキュメントは[データのHDFSまたは外部クラウドストレージシステムへのインポート](../loading/BrokerLoad.md)および[Apache Spark™を使用したバルクインポート](../loading/SparkLoad.md)を参照してください。[#9049](https://github.com/starrocks/starrocks/pull/9049) [#9228](https://github.com/StarRocks/starrocks/pull/9228)
  - 大量のORC小ファイルシナリオでのBroker Loadのインポートパフォーマンスが最適化されました。[#11380](https://github.com/StarRocks/starrocks/pull/11380)
  - プライマリキーモデルテーブルへのデータインポート時のメモリ使用量が最適化されました。[#12068](https://github.com/StarRocks/starrocks/pull/12068)
- StarRocksの組み込み`information_schema`データベースおよびその中の`tables`テーブルと`columns`テーブルが最適化されました。`table_config`テーブルが追加されました。詳細なドキュメントは[Information Schema](../reference/overview-pages/information_schema.md)を参照してください。[#10033](https://github.com/StarRocks/starrocks/pull/10033)
- バックアップとリストアが最適化されました：
  - データベースレベルのバックアップとリストアがサポートされました。詳細なドキュメントは[バックアップとリストア](../administration/Backup_and_restore.md)を参照してください。[#11619](https://github.com/StarRocks/starrocks/issues/11619)
  - プライマリキーモデルテーブルのバックアップとリストアがサポートされました。詳細なドキュメントはバックアップとリストアを参照してください。[#11885](https://github.com/StarRocks/starrocks/pull/11885)
- 関数の最適化：
  - [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)にパラメータが追加され、時間範囲の開始点と終了点を計算できるようになりました。[#11216](https://github.com/StarRocks/starrocks/pull/11216)
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)は、重複したタイムスタンプの計算を防ぐために、厳密に増加するモードをサポートしています。[#10134](https://github.com/StarRocks/starrocks/pull/10134)
  - [unnest](../sql-reference/sql-functions/array-functions/unnest.md)は可変引数をサポートしています。[#12484](https://github.com/StarRocks/starrocks/pull/12484)
  - leadとlagウィンドウ関数は、HLLおよびBITMAP型のデータをクエリできるようになりました。詳細なドキュメントは[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。[#12108](https://github.com/StarRocks/starrocks/pull/12108)
  - [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、[array_sort](../sql-reference/sql-functions/array-functions/array_sort.md)、[array_concat](../sql-reference/sql-functions/array-functions/array_concat.md)、[array_slice](../sql-reference/sql-functions/array-functions/array_slice.md)、および[reverse](../sql-reference/sql-functions/string-functions/reverse.md)関数は、JSONデータをクエリできるようになりました。[#13155](https://github.com/StarRocks/starrocks/pull/13155)
  - `current_date`、`current_timestamp`、`current_time`、`localtimestamp`、`localtime`関数は、`()`を付けずに実行できるようになりました。例：`select current_date;`。[#14319](https://github.com/StarRocks/starrocks/pull/14319)
- FEログが最適化され、一部の冗長な情報が削除されました。[#15374](https://github.com/StarRocks/starrocks/pull/15374)

### 問題の修正

以下の問題が修正されました：

- append_trailing_char_if_absent関数の空値操作が誤っていました。[#13762](https://github.com/StarRocks/starrocks/pull/13762)
- 削除されたテーブルをRECOVERステートメントで復元した後、テーブルが存在しなくなりました。[#13921](https://github.com/StarRocks/starrocks/pull/13921)
- SHOW CREATE MATERIALIZED VIEWの結果にcatalogおよびdatabaseの情報が欠落していました。[#12833](https://github.com/StarRocks/starrocks/pull/12833)
- waiting_stable状態のスキーマ変更タスクをキャンセルできない問題が修正されました。[#12530](https://github.com/StarRocks/starrocks/pull/12530)
- `SHOW PROC '/statistic';`コマンドの結果がリーダーFEと非リーダーFEで異なる問題が修正されました。[#12491](https://github.com/StarRocks/starrocks/issues/12491)
- FEが生成する実行計画にパーティションIDが欠落しており、Hiveパーティションデータの取得に失敗していました。[#15486](https://github.com/StarRocks/starrocks/pull/15486)
- SHOW CREATE TABLEの結果でORDER BY句の位置が間違っていました。[#13809](https://github.com/StarRocks/starrocks/pull/13809)

### 動作の変更

- `AWS_EC2_METADATA_DISABLED`パラメータのデフォルト値が`False`に設定され、デフォルトでAmazon EC2のメタデータを取得してAWSリソースにアクセスするようになりました。
- セッション変数`is_report_success`が`enable_profile`に名前が変更され、SHOW VARIABLESステートメントで確認できるようになりました。
- 新增四个关键字：`CURRENT_DATE`、`CURRENT_TIME`、`LOCALTIME`、`LOCALTIMESTAMP`。[#14319](https://github.com/StarRocks/starrocks/pull/14319)
- 表名和库名的长度限制放宽至不超过 1023 个字符。[#14929](https://github.com/StarRocks/starrocks/pull/14929) [#15020](https://github.com/StarRocks/starrocks/pull/15020)
- BEの設定項目 `enable_event_based_compaction_framework` と `enable_size_tiered_compaction_strategy` はデフォルトで有効になっており、タブレットの数が多い場合や単一タブレットのデータ量が大きい場合に、コンパクションのコストを大幅に削減することができます。

### アップグレードに関する注意事項

- 2.0.x、2.1.x、2.2.x、2.3.x、または2.4.xからアップグレード可能です。通常、バージョンをロールバックすることはお勧めしませんが、ロールバックが必要な場合は2.4.xにのみロールバックすることをお勧めします。
