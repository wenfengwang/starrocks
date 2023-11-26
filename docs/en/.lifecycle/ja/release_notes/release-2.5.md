---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.5

## 2.5.14

リリース日: 2023年11月14日

### 改善点

- システムデータベース `INFORMATION_SCHEMA` の `COLUMNS` テーブルは、ARRAY、MAP、およびSTRUCTの列を表示できるようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

### バグ修正

以下の問題が修正されました:

- サブクエリでON条件がネストされている場合、エラー `java.lang.IllegalStateException: null` が報告されます。[#30876](https://github.com/StarRocks/starrocks/pull/30876)
- `INSERT INTO SELECT ... LIMIT` の直後にCOUNT(*)が実行されると、レプリカ間でCOUNT(*)の結果が一貫性がなくなる場合があります。[#24435](https://github.com/StarRocks/starrocks/pull/24435)
- CASTで指定されたターゲットデータ型が元のデータ型と同じ場合、特定のデータ型のデータでBEがクラッシュする場合があります。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- Broker Loadを介してデータをロードする際に特定のパス形式が使用されると、`msg:Fail to parse columnsFromPath, expected: [rec_dt]` というエラーが報告されます。[#32721](https://github.com/StarRocks/starrocks/issues/32721)
- 3.xへのアップグレード時に、一部の列の型もアップグレードされる場合（例: DecimalがDecimal v3にアップグレードされる場合）、特定の特性を持つテーブルでCompactionが実行されるとBEがクラッシュします。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードする際に、高並行のロードジョブが存在し、HTTPとScanスレッドの数が上限に達した場合、ロードジョブが予期せず中断されます。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlが呼び出されるとBEがクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- BITMAP列をプライマリキーテーブルに追加すると、次のエラーが発生して追加に失敗します: `Analyze columnDef error: No aggregate function specified for 'userid'`。[#31763](https://github.com/StarRocks/starrocks/pull/31763)
- 永続インデックスが有効なプライマリキーテーブルに長時間かつ頻繁にデータをロードすると、BEがクラッシュする場合があります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効な場合、クエリ結果が正しくありません。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- プライマリキーテーブルを作成する際にnullableなソートキーを指定すると、コンパクションが失敗します。[#29225](https://github.com/StarRocks/starrocks/pull/29225)
- 複雑なJoinクエリに対して "StarRocks planner use long time 10000 ms in logical phase" というエラーが発生することがあります。[#34177](https://github.com/StarRocks/starrocks/pull/34177)

## 2.5.13

リリース日: 2023年9月28日

### 改善点

- Window関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPは、ORDER BY句とWindow句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型のデータのクエリ中にDECIMALオーバーフローが発生した場合、NULLではなくエラーが返されます。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドを実行した場合、MySQLと同じ結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- BEの起動時のメモリ使用量を削減するため、削除されたタブレットに対応するRowsetをクリーンアップするようになりました。[#30625](https://github.com/StarRocks/starrocks/pull/30625)

### バグ修正

以下の問題が修正されました:

- Spark ConnectorまたはFlink Connectorを使用してStarRocksからデータを読み取る際に、エラー "Set cancelled by MemoryScratchSinkOperator" が発生します。[#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 集計関数が含まれるORDER BY句を持つクエリで、エラー "java.lang.IllegalStateException: null" が発生します。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 非アクティブなマテリアライズドビューが存在する場合、FEの再起動に失敗します。[#30015](https://github.com/StarRocks/starrocks/pull/30015)
- 重複したパーティションに対してINSERT OVERWRITE操作を実行すると、メタデータが破損し、FEの再起動に失敗します。[#27545](https://github.com/StarRocks/starrocks/pull/27545)
- プライマリキーテーブルに存在しない列を変更しようとすると、エラー "java.lang.NullPointerException: null" が発生します。[#30366](https://github.com/StarRocks/starrocks/pull/30366)
- パーティション化されたStarRocks外部テーブルにデータをロードする際に、エラー "get TableMeta failed from TNetworkAddress" が発生します。[#30124](https://github.com/StarRocks/starrocks/pull/30124)
- 特定のシナリオで、CloudCanalを介してデータをクエリする際にエラーが発生します。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- Flink Connectorを使用してデータをロードする際に、エラー "current running txns on db xxx is 200, larger than limit 200" が発生します。[#18393](https://github.com/StarRocks/starrocks/pull/18393)
- 集約関数を含むHAVING句を使用する非同期マテリアライズドビューは、クエリを正しく書き換えることができません。[#29976](https://github.com/StarRocks/starrocks/pull/29976)

## 2.5.12

リリース日: 2023年9月4日

### 改善点

- SQL内のコメントは監査ログに保持されるようになりました。[#29747](https://github.com/StarRocks/starrocks/pull/29747)
- INSERT INTO SELECTのCPUおよびメモリの統計情報が監査ログに追加されました。[#29901](https://github.com/StarRocks/starrocks/pull/29901)

### バグ修正

以下の問題が修正されました:

- Broker Loadを使用してデータをロードする際、一部のフィールドのNOT NULL属性がBEのクラッシュや "msg:mismatched row count" エラーを引き起こす場合があります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- ORC形式のファイルに対するクエリが失敗するため、Apache ORCのバグ修正ORC-1304 ([apache/orc#1299](https://github.com/apache/orc/pull/1299)) がマージされていません。[#29804](https://github.com/StarRocks/starrocks/pull/29804)
- プライマリキーテーブルのリストアにより、BEの再起動後にメタデータの不整合が発生します。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 2.5.11

リリース日: 2023年8月28日

### 改善点

- すべての複合述語とWHERE句のすべての式に対して、暗黙の型変換をサポートするようになりました。[セッション変数](https://docs.starrocks.io/en-us/3.1/reference/System_variable) `enable_strict_type` を使用して、暗黙の型変換を有効または無効にすることができます。デフォルト値は `false` です。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- Icebergカタログを作成する際に `hive.metastore.uri` を指定しない場合、より正確なエラーメッセージが表示されるようになりました。[#16543](https://github.com/StarRocks/starrocks/issues/16543)
- エラーメッセージ `xxx too many versions xxx` に追加のプロンプトを追加しました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 動的パーティショニングは、パーティショニング単位として `year` をさらにサポートするようになりました。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

### バグ修正

以下の問題が修正されました:

- マテリアライズドビューのクエリ結果が正確でない場合がある。[#28824](https://github.com/StarRocks/starrocks/issues/28824)
- ビットマップまたはHLLフィールドがWHERE条件のフィールドとして使用される場合、DELETE操作が失敗します。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 同期呼び出し（SYNC MODE）で非同期マテリアライズドビューを手動でリフレッシュすると、`information_schema.task_runs` テーブルに複数のINSERT OVERWRITEレコードが表示されます。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- エラー状態のタブレットに対してCLONE操作がトリガされると、ディスク使用量が増加します。[#28488](https://github.com/StarRocks/starrocks/pull/28488)
- Join Reorderが有効になっている場合、クエリ結果が定数の列をクエリする場合に正しくありません。[#29239](https://github.com/StarRocks/starrocks/pull/29239)
- SSDとHDDの間でタブレットの移行が行われる際、FEがBEに過剰な移行タスクを送信すると、BEがOOMの問題に遭遇します。[#29055](https://github.com/StarRocks/starrocks/pull/29055)
- `/apache_hdfs_broker/lib/log4j-1.2.17.jar` のセキュリティ脆弱性。[#28866](https://github.com/StarRocks/starrocks/pull/28866)
- Hiveカタログを介してデータをクエリする際に、パーティショニング列とOR演算子がWHERE句で使用されると、クエリ結果が正しくありません。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- データクエリ中にエラー "java.util.ConcurrentModificationException: null" が発生することがあります。[#29296](https://github.com/StarRocks/starrocks/pull/29296)
- ベーステーブルの非同期マテリアライズドビューが削除された場合、FEを再起動できません。[#29318](https://github.com/StarRocks/starrocks/pull/29318)
- データが非同期マテリアライズドビューのベーステーブルに書き込まれている間に、異なるデータベース間で作成された非同期マテリアライズドビューのベーステーブルにデッドロックが発生する場合があります。[#29432](https://github.com/StarRocks/starrocks/pull/29432)

## 2.5.10

リリース日: 2023年8月7日

### 新機能

- 集計関数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、および[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)をサポートしました。
- [window関数](../sql-reference/sql-functions/Window_function.md) COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPをサポートしました。

### 改善点

- 複数のBEが追加された場合のディスクのバランス速度を最適化しました。[# 19418](https://github.com/StarRocks/starrocks/pull/19418)
- `storage_medium` の推論を最適化しました。BEがSSDとHDDの両方をストレージデバイスとして使用する場合、`storage_cooldown_time` プロパティが指定されている場合、StarRocksは `storage_medium` を `SSD` に設定します。それ以外の場合、StarRocksは `storage_medium` を `HDD` に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- Unique Keyテーブルのパフォーマンスを最適化しました。値の列から統計情報を収集しないようにしました。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

以下の問題が修正されました:

- コロケーションテーブルの場合、ステートステータスを `bad` として指定することができます。例: `ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`。BEの数がレプリカの数以下の場合、破損したレプリカを修復することはできません。[# 17876](https://github.com/StarRocks/starrocks/issues/17876)
- BEが起動した後、プロセスは存在しますが、BEポートが有効になりません。[# 19347](https://github.com/StarRocks/starrocks/pull/19347)
- ウィンドウ関数がネストされたサブクエリを含む集計クエリの結果が正しくありません。[# 19725](https://github.com/StarRocks/starrocks/issues/19725)
- `auto_refresh_partitions_limit` が初回のマテリアライズドビュー（MV）のリフレッシュに影響を与えません。その結果、すべてのパーティションがリフレッシュされます。[# 19759](https://github.com/StarRocks/starrocks/issues/19759)
- 複雑なデータ（MAPやSTRUCTなど）でネストされた配列データを持つCSV Hive外部テーブルをクエリするとエラーが発生します。[# 20233](https://github.com/StarRocks/starrocks/pull/20233)
- Sparkコネクタを使用したクエリがタイムアウトします。[# 20264](https://github.com/StarRocks/starrocks/pull/20264)
- 2つのレプリカテーブルのうち1つのレプリカが破損した場合、テーブルは回復できません。[# 20681](https://github.com/StarRocks/starrocks/pull/20681)
- MVクエリのリライトの失敗によるクエリの失敗。[# 19549](https://github.com/StarRocks/starrocks/issues/19549)
- データベースロックによりメトリックインターフェースが期限切れになります。[# 20790](https://github.com/StarRocks/starrocks/pull/20790)
- ブロードキャストジョインの結果が間違って返されます。[# 20952](https://github.com/StarRocks/starrocks/issues/20952)
- CREATE TABLEでサポートされていないデータ型が使用された場合、NPEが返されます。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- クエリキャッシュ機能を使用したwindow_funnel()の問題。[# 21474](https://github.com/StarRocks/starrocks/issues/21474)
- CTEがリライトされた後、最適化計画の選択に予期しない長い時間がかかります。[# 16515](https://github.com/StarRocks/starrocks/pull/16515)

## 2.5.4

リリース日: 2023年4月4日

### 改善点

- クエリ計画中のマテリアライズドビューのクエリのリライトのパフォーマンスを最適化しました。クエリ計画にかかる時間が約70%削減されました。[#19579](https://github.com/StarRocks/starrocks/pull/19579)
- 型推論ロジックを最適化しました。`SELECT sum(CASE WHEN XXX);`のようなクエリには、`SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;`のように定数`0`が含まれている場合、プリアグリゲーションが自動的に有効になり、クエリの高速化が行われます。[#19474](https://github.com/StarRocks/starrocks/pull/19474)
- `SHOW CREATE VIEW`を使用してマテリアライズドビューの作成ステートメントを表示することができるようになりました。[#19999](https://github.com/StarRocks/starrocks/pull/19999)
- 単一のbRPCリクエストで2GB以上のパケットを送信することができるようになりました。BEノード間の単一のbRPCリクエストで2GB以上のパケットを送信することができるようになりました。[#20283](https://github.com/StarRocks/starrocks/pull/20283) [#20230](https://github.com/StarRocks/starrocks/pull/20230)
- [SHOW CREATE CATALOG](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して外部カタログの作成ステートメントをクエリすることができるようになりました。

### バグ修正

以下のバグが修正されました。

- マテリアライズドビューのクエリがリライトされた後、低カーディナリティ最適化のためのグローバル辞書が効果を発揮しません。[#19615](https://github.com/StarRocks/starrocks/pull/19615)
- マテリアライズドビューのクエリがリライトに失敗した場合、クエリが失敗します。[#19774](https://github.com/StarRocks/starrocks/pull/19774)
- プライマリキーまたはユニークキーテーブルを基にしたマテリアライズドビューが作成された場合、そのマテリアライズドビューのクエリはリライトできません。[#19600](https://github.com/StarRocks/starrocks/pull/19600)
- マテリアライズドビューの列名は大文字と小文字を区別します。しかし、テーブルを作成する際に、テーブル作成ステートメントの`PROPERTIES`に列名が間違っていてもエラーメッセージなしでテーブルが正常に作成され、さらにそのテーブルに作成されたマテリアライズドビューのクエリのリライトに失敗します。[#19780](https://github.com/StarRocks/starrocks/pull/19780)
- マテリアライズドビューのクエリがリライトされた後、クエリプランにはパーティション列ベースの無効な述語が含まれており、クエリのパフォーマンスに影響を与えます。[#19784](https://github.com/StarRocks/starrocks/pull/19784)
- 新しく作成されたパーティションにデータがロードされると、マテリアライズドビューのクエリのリライトに失敗する場合があります。[#20323](https://github.com/StarRocks/starrocks/pull/20323)
- マテリアライズドビューの作成時に`"storage_medium" = "SSD"`を設定すると、マテリアライズドビューのリフレッシュが失敗します。[#19539](https://github.com/StarRocks/starrocks/pull/19539) [#19626](https://github.com/StarRocks/starrocks/pull/19626)
- プライマリキーテーブルで同時コンパクションが発生する場合があります。[#19692](https://github.com/StarRocks/starrocks/pull/19692)
- 大量のDELETE操作の後にコンパクションがすぐに発生しません。[#19623](https://github.com/StarRocks/starrocks/pull/19623)
- ステートメントの式に複数の低カーディナリティ列が含まれている場合、式が正しくリライトされない場合があります。その結果、低カーディナリティ最適化のためのグローバル辞書が効果を発揮しません。[#20161](https://github.com/StarRocks/starrocks/pull/20161)

## 2.5.3

リリース日: 2023年3月10日

### 改善点

- マテリアライズドビュー（MVs）のクエリリライトを最適化しました。
  - アウタージョインとクロスジョインを含むクエリのリライトをサポートします。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - MVのデータスキャンロジックを最適化し、リライトされたクエリをさらに高速化します。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 単一テーブルの集計クエリのリライト機能を強化しました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - クエリされるテーブルがMVのベーステーブルのサブセットである場合のView Deltaシナリオでのリライト機能を強化しました。[#18800](https://github.com/StarRocks/starrocks/pull/18800)
- ウィンドウ関数RANK()をフィルタまたはソートキーとして使用する場合のパフォーマンスとメモリ使用量を最適化しました。[#17553](https://github.com/StarRocks/starrocks/issues/17553)

### バグ修正

以下のバグが修正されました。

- 配列データ内のnullリテラル`[]`によって引き起こされるエラーが修正されました。[#18563](https://github.com/StarRocks/starrocks/pull/18563)
- 複雑なクエリシナリオで低カーディナリティ最適化辞書の誤用が修正されました。辞書マッピングのチェックが辞書の適用前に追加されました。[#17318](https://github.com/StarRocks/starrocks/pull/17318)
- 単一のBE環境でのLocal Shuffleにより、GROUP BYが重複した結果を生成します。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- パーティション関連のPROPERTIESの誤用により、パーティションのないMVのリフレッシュが失敗する場合があります。ユーザーがMVを作成するときに、パーティションのPROPERTIESチェックが実行されるようになりました。[#18741](https://github.com/StarRocks/starrocks/pull/18741)
- ParquetのRepetition列の解析エラーが修正されました。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 取得した列のnull許容情報が正しくありません。解決策: CTASを使用してプライマリキーテーブルを作成する場合、プライマリキーカラムのみがnull許容ではなく、プライマリキー以外のカラムはnull許容です。[#16431](https://github.com/StarRocks/starrocks/pull/16431)
- プライマリキーテーブルからデータを削除することによって引き起こされるいくつかの問題が修正されました。[#18768](https://github.com/StarRocks/starrocks/pull/18768)

## 2.5.2

リリース日: 2023年2月21日

### 新機能

- インスタンスプロファイルとアサムドロールベースの認証方法を使用して、AWS S3とAWS Glueにアクセスすることができるようになりました。[#15958](https://github.com/StarRocks/starrocks/pull/15958)
- bit_shift_left、bit_shift_right、bit_shift_right_logicalのビット関数をサポートします。[#14151](https://github.com/StarRocks/starrocks/pull/14151)

### 改善点

- メモリ解放ロジックを最適化し、大量の集計クエリを含むクエリのピークメモリ使用量を大幅に削減しました。[#16913](https://github.com/StarRocks/starrocks/pull/16913)
- ソートのメモリ使用量を削減しました。ウィンドウ関数やソートを含むクエリのメモリ消費量が半分になります。[#16937](https://github.com/StarRocks/starrocks/pull/16937) [#17362](https://github.com/StarRocks/starrocks/pull/17362) [#17408](https://github.com/StarRocks/starrocks/pull/17408)

### バグ修正

以下のバグが修正されました。

- MAPとARRAYデータを含むApache Hive外部テーブルのリフレッシュができない問題が修正されました。[#17548](https://github.com/StarRocks/starrocks/pull/17548)
- Supersetがマテリアライズドビューの列の型を識別できない問題が修正されました。[#17686](https://github.com/StarRocks/starrocks/pull/17686)
- BI接続が失敗する問題が修正されました。SET GLOBAL/SESSION TRANSACTIONが解析できないためです。[#17295](https://github.com/StarRocks/starrocks/pull/17295)
- コロケートグループ内の動的パーティションテーブルのバケット数を変更できず、エラーメッセージが返される問題が修正されました。[#17418](https://github.com/StarRocks/starrocks/pull/17418/)
- Prepareステージの失敗によって引き起こされる潜在的な問題が修正されました。[#17323](https://github.com/StarRocks/starrocks/pull/17323)

### 動作の変更

- `AWS_EC2_METADATA_DISABLED`パラメータのデフォルト値を`False`に変更しました。これにより、Amazon EC2のメタデータを取得してAWSリソースにアクセスすることが可能になります。
- セッション変数`is_report_success`を`enable_profile`に名前変更しました。SHOW VARIABLESステートメントでクエリできるようになりました。
- 予約語に`CURRENT_DATE`、`CURRENT_TIME`、`LOCALTIME`、`LOCALTIMESTAMP`の4つの予約語が追加されました。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.1

リリース日: 2023年2月5日

### 改善点

- 外部カタログを基に作成された非同期マテリアライズドビューは、クエリのリライトをサポートします。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#15791](https://github.com/StarRocks/starrocks/issues/15791)
- 自動CBO統計情報の収集のためのコレクション期間を指定できるようになりました。これにより、自動フルコレクションによるクラスタのパフォーマンスのジッタが防止されます。[#14996](https://github.com/StarRocks/starrocks/pull/14996)
- Thriftサーバーキューを追加しました。INSERT INTO SELECT中に即座に処理できないリクエストは、Thriftサーバーキューに保留され、リクエストが拒否されることがありません。[#14571](https://github.com/StarRocks/starrocks/pull/14571)
- FEパラメータ`default_storage_medium`を非推奨にしました。テーブルを作成する際に`storage_medium`が明示的に指定されていない場合、システムはBEディスクのタイプに基づいてテーブルのストレージメディアを自動的に推論します。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_VIEW.md)の`storage_medium`の説明を参照してください。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### バグ修正

以下のバグが修正されました。

- SET PASSWORDによって発生するヌルポインタ例外（NPE）が修正されました。[#15247](https://github.com/StarRocks/starrocks/pull/15247)
- 空のキーを持つJSONデータを解析できない問題が修正されました。[#16852](https://github.com/StarRocks/starrocks/pull/16852)
- 無効な型のデータを正常にARRAYデータに変換できる問題が修正されました。[#16866](https://github.com/StarRocks/starrocks/pull/16866)
- 例外が発生した場合にNested Loop Joinを中断できない問題が修正されました。[#16875](https://github.com/StarRocks/starrocks/pull/16875)

### 動作の変更

- FEパラメータ`default_storage_medium`のデフォルト値を`false`から`true`に変更しました。これにより、非同期マテリアライズドビューがデフォルトで有効になります。
- リザーブドキーワードリストにCHARACTERを追加しました。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.0

リリース日: 2023年1月22日

### 新機能

- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)と[Hudi外部テーブル](../data_source/External_table.md#deprecated-hudi-external-table)を使用してMerge On Readテーブルをクエリすることができるようになりました。[#6780](https://github.com/StarRocks/starrocks/pull/6780)
- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、Hudiカタログ、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用してSTRUCTおよびMAPデータをクエリすることができるようになりました。[#10677](https://github.com/StarRocks/starrocks/issues/10677)
- [データキャッシュ](../data_source/data_cache.md)を提供し、HDFSなどの外部ストレージシステムに格納されたホットデータのアクセスパフォーマンスを向上させます。[#11597](https://github.com/StarRocks/starrocks/pull/11579)
- [Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)を作成することができるようになりました。これにより、Delta Lakeのデータを直接クエリできるようになります。[#11972](https://github.com/StarRocks/starrocks/issues/11972)
- Hive、Hudi、IcebergカタログはAWS Glueと互換性があります。[#12249](https://github.com/StarRocks/starrocks/issues/12249)
- [ファイル外部テーブル](../data_source/file_external_table.md)を作成することができるようになりました。これにより、HDFSやオブジェクトストアからのParquetやORCファイルを直接クエリできるようになります。[#13064](https://github.com/StarRocks/starrocks/pull/13064)
- Hive、Hudi、Icebergカタログおよびマテリアライズドビューを基にしたマテリアライズドビューを作成することができるようになりました。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#11873](https://github.com/StarRocks/starrocks/pull/11873)
- プライマリキーテーブルを使用するテーブルに対して条件付きの更新をサポートします。詳細については、[データのロードを介した変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。[#12159](https://github.com/StarRocks/starrocks/pull/12159)
- [クエリキャッシュ](../using_starrocks/query_cache.md)をサポートし、クエリの中間計算結果をキャッシュして高並行の単純なクエリのQPSを向上させ、平均レイテンシを削減します。[#9194](https://github.com/StarRocks/starrocks/pull/9194)
- ブローカーロードジョブの優先度を指定できるようになりました。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。[#11029](https://github.com/StarRocks/starrocks/pull/11029)
- StarRocksネイティブテーブルのデータロードのレプリカ数を指定できるようになりました。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11253](https://github.com/StarRocks/starrocks/pull/11253)
- [クエリキュー](../administration/query_queues.md)をサポートします。[#12594](https://github.com/StarRocks/starrocks/pull/12594)
- データロードが占有する計算リソースを分離し、データロードタスクのリソース消費を制限することができるようになりました。詳細については、[リソースグループ](../administration/resource_group.md)を参照してください。[#12606](https://github.com/StarRocks/starrocks/pull/12606)
- StarRocksネイティブテーブルのデータ圧縮アルゴリズムとして、LZ4、Zstd、Snappy、およびZlibを指定できるようになりました。詳細については、[データ圧縮](../table_design/data_compression.md)を参照してください。[#10097](https://github.com/StarRocks/starrocks/pull/10097) [#12020](https://github.com/StarRocks/starrocks/pull/12020)
- [ユーザー定義変数](../reference/user_defined_variables.md)をサポートします。[#10011](https://github.com/StarRocks/starrocks/pull/10011)
- [ラムダ式](../sql-reference/sql-functions/Lambda_expression.md)と次の高階関数をサポートします: [array_map](../sql-reference/sql-functions/array-functions/array_map.md)、[array_sum](../sql-reference/sql-functions/array-functions/array_sum.md)、および[array_sortby](../sql-reference/sql-functions/array-functions/array_sortby.md)。[#9461](https://github.com/StarRocks/starrocks/pull/9461) [#9806](https://github.com/StarRocks/starrocks/pull/9806) [#10323](https://github.com/StarRocks/starrocks/pull/10323) [#14034](https://github.com/StarRocks/starrocks/pull/14034)
- [QUALIFY句](../sql-reference/sql-functions/Window_function.md)を提供し、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)の結果をフィルタリングします。[#13239](https://github.com/StarRocks/starrocks/pull/13239)
- uuid()関数とuuid_numeric()関数が返す結果を、テーブルを作成する際のカラムのデフォルト値として使用できるようになりました。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11155](https://github.com/StarRocks/starrocks/pull/11155)
- [map_size](../sql-reference/sql-functions/map-functions/map_size.md)、[map_keys](../sql-reference/sql-functions/map-functions/map_keys.md)、[map_values](../sql-reference/sql-functions/map-functions/map_values.md)、[max_by](../sql-reference/sql-functions/aggregate-functions/max_by.md)、[sub_bitmap](../sql-reference/sql-functions/bitmap-functions/sub_bitmap.md)、[bitmap_to_base64](../sql-reference/sql-functions/bitmap-functions/bitmap_to_base64.md)、[host_name](../sql-reference/sql-functions/utility-functions/host_name.md)、および[date_slice](../sql-reference/sql-functions/date-time-functions/date_slice.md)の関数をサポートします。[#11299](https://github.com/StarRocks/starrocks/pull/11299) [#11323](https://github.com/StarRocks/starrocks/pull/11323) [#12243](https://github.com/StarRocks/starrocks/pull/12243) [#11776](https://github.com/StarRocks/starrocks/pull/11776) [#12634](https://github.com/StarRocks/starrocks/pull/12634) [#14225](https://github.com/StarRocks/starrocks/pull/14225)

### 改善点

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用して外部データをクエリする際のメタデータアクセスのパフォーマンスを最適化しました。[#11349](https://github.com/StarRocks/starrocks/issues/11349)
- [Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を使用してARRAYデータをクエリすることができるようになりました。[#9693](https://github.com/StarRocks/starrocks/pull/9693)
- マテリアライズドビューの以下の側面を最適化しました。
  - 非同期マテリアライズドビューは、SPJG型マテリアライズドビューに基づいて自動的かつ透過的なクエリのリライトをサポートします。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md#rewrite-and-accelerate-queries-with-the-asynchronous-materialized-view)を参照してください。[#13193](https://github.com/StarRocks/starrocks/issues/13193)
  - 非同期マテリアライズドビューは、複数の非同期リフレッシュメカニズムをサポートします。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md#manually-refresh-an-asynchronous-materialized-view)を参照してください。[#12712](https://github.com/StarRocks/starrocks/pull/12712) [#13171](https://github.com/StarRocks/starrocks/pull/13171) [#13229](https://github.com/StarRocks/starrocks/pull/13229) [#12926](https://github.com/StarRocks/starrocks/pull/12926)
  - マテリアライズドビューのリフレッシュの効率が向上しました。[#13167](https://github.com/StarRocks/starrocks/issues/13167)
- データロードの以下の側面を最適化しました。
  - "single leader replication"モードをサポートすることで、マルチレプリカシナリオでのロードパフォーマンスを最適化しました。データロードのパフォーマンスが1倍に向上します。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)の`replicated_storage`を参照してください。[#10138](https://github.com/StarRocks/starrocks/pull/10138)
  - ブローカーロードとSparkロードは、HDFSクラスタまたはKerberosユーザーが1つだけ構成されている場合、データロードにブローカーに依存する必要がありません。ただし、複数のHDFSクラスタや複数のKerberosユーザーを使用する場合は、引き続きブローカーをデプロイする必要があります。詳細については、[HDFSまたはクラウドストレージからデータをロードする](../loading/BrokerLoad.md)と[Apache Spark™を使用したバルクロード](../loading/SparkLoad.md)を参照してください。[#9049](https://github.com/starrocks/starrocks/pull/9049) [#9228](https://github.com/StarRocks/starrocks/pull/9228)
  - 大量の小さなORCファイルをロードする場合のBroker Loadのパフォーマンスを最適化しました。[#11380](https://github.com/StarRocks/starrocks/pull/11380)
  - データをプライマリキーテーブルにロードする際のメモリ使用量を削減しました。

- `information_schema`データベースと`tables`および`columns`テーブルのメタデータアクセスパフォーマンスを最適化しました。`table_config`という新しいテーブルが追加されました。詳細については、[Information Schema](../reference/information_schema/information_schema.md)を参照してください。[#10033](https://github.com/StarRocks/starrocks/pull/10033)
- データバックアップとリストアを最適化しました。
  - 一度に複数のテーブルのデータをバックアップおよびリストアすることができるようになりました。詳細については、[データのバックアップとリストア](../administration/Backup_and_restore.md)を参照してください。[#11619](https://github.com/StarRocks/starrocks/issues/11619)
  - プライマリキーテーブルのデータをバックアップおよびリストアすることができるようになりました。詳細については、バックアップとリストアを参照してください。[#11885](https://github.com/StarRocks/starrocks/pull/11885)
- 次の関数を最適化しました。
  - [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md)関数にオプションのパラメータを追加し、時間間隔の開始または終了を返すかどうかを指定できるようになりました。[#11216](https://github.com/StarRocks/starrocks/pull/11216)
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)関数の新しいモード`INCREASE`を追加し、重複するタイムスタンプの計算を回避します。[#10134](https://github.com/StarRocks/starrocks/pull/10134)
  - [unnest](../sql-reference/sql-functions/array-functions/unnest.md)関数で複数の引数を指定できるようになりました。[#12484](https://github.com/StarRocks/starrocks/pull/12484)
  - lead()関数とlag()関数は、HLLとBITMAPデータをクエリすることができるようになりました。詳細については、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。[#12108](https://github.com/StarRocks/starrocks/pull/12108)
  - 次のARRAY関数は、JSONデータをクエリすることができるようになりました: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)、[array_sort](../sql-reference/sql-functions/array-functions/array_sort.md)、[array_concat](../sql-reference/sql-functions/array-functions/array_concat.md)、[array_slice](../sql-reference/sql-functions/array-functions/array_slice.md)、および[reverse](../sql-reference/sql-functions/array-functions/reverse.md)。[#13155](https://github.com/StarRocks/starrocks/pull/13155)
  - 一部の関数の使用方法を最適化しました。`current_date`、`current_timestamp`、`current_time`、`localtimestamp`、および`localtime`関数は、`()`を使用せずに実行できます。たとえば、`select current_date;`と直接実行できます。[# 14319](https://github.com/StarRocks/starrocks/pull/14319)
- FEログからいくつかの冗長な情報を削除しました。[# 15374](https://github.com/StarRocks/starrocks/pull/15374)

### バグ修正

以下のバグが修正されました。

- append_trailing_char_if_absent()関数の最初の引数が空の場合に正しい結果が返らない問題が修正されました。[#13762](https://github.com/StarRocks/starrocks/pull/13762)
- RECOVERステートメントを使用してテーブルをリストアした後、テーブルが存在しない問題が修正されました。[#13921](https://github.com/StarRocks/starrocks/pull/13921)
- SHOW CREATE MATERIALIZED VIEWステートメントが返す結果に、マテリアライズドビューの作成時にクエリステートメントで指定されたデータベースとカタログが含まれていない問題が修正されました。[#12833](https://github.com/StarRocks/starrocks/pull/12833)
- `waiting_stable`状態のスキーマ変更ジョブをキャンセルできない問題が修正されました。[#12530](https://github.com/StarRocks/starrocks/pull/12530)
- リーダーFEと非リーダーFEで`SHOW PROC '/statistic';`コマンドを実行すると、異なる結果が返される問題が修正されました。[#12491](https://github.com/StarRocks/starrocks/issues/12491)
- SHOW CREATE TABLEの結果でORDER BY句の位置が正しくない問題が修正されました。[# 13809](https://github.com/StarRocks/starrocks/pull/13809)
- ユーザーがHiveデータをクエリするためにHiveカタログを使用する場合、FEによって生成された実行計画にパーティションIDが含まれていない場合、BEはHiveパーティションデータをクエリできません。[# 15486](https://github.com/StarRocks/starrocks/pull/15486)。

### 動作の変更

- `AWS_EC2_METADATA_DISABLED`パラメータのデフォルト値を`False`に変更しました。これにより、Amazon EC2のメタデータを取得してAWSリソースにアクセスすることが可能になります。
- セッション変数`is_report_success`を`enable_profile`に名前変更しました。SHOW VARIABLESステートメントでクエリできるようになりました。
- 予約語にCHARACTERを追加しました。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

### アップグレードノート

- クラスタを2.0.x、2.1.x、2.2.x、2.3.x、または2.4.xから2.5.0にアップグレードすることができます。ただし、ロールバックを実行する場合は、2.4.xにのみロールバックすることをお勧めします。
