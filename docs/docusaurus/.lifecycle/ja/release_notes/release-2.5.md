---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.5

## 2.5.16

リリース日: 2023年12月1日

### バグ修正

以下の問題が修正されました:

- 特定のシナリオでグローバルランタイムフィルターがBEをクラッシュさせる場合があります。[#35776](https://github.com/StarRocks/starrocks/pull/35776)

## 2.5.15

リリース日: 2023年11月29日

### 改善点

- 遅いリクエストを追跡するための遅いリクエストログが追加されました。[#33908](https://github.com/StarRocks/starrocks/pull/33908)
- 複数のファイルがある場合にSpark Loadを使用してParquetおよびORCファイルを読み込む際のパフォーマンスが最適化されました。[#34787](https://github.com/StarRocks/starrocks/pull/34787)
- ビットマップ関連のいくつかの操作のパフォーマンスが最適化されました:
  - ネストされたループ結合が最適化されました。[#340804](https://github.com/StarRocks/starrocks/pull/34804) [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor` 関数が最適化されました。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - Bitmapのパフォーマンスを最適化しメモリ消費を減らすためにCopy on Writeがサポートされました。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 互換性の変更

#### パラメータ

- FE 動的パラメータ `enable_new_publish_mechanism` が静的パラメータに変更されました。パラメータ設定を変更した後にFEを再起動する必要があります。[#35338](https://github.com/StarRocks/starrocks/pull/35338)

### バグ修正

- Broker Load ジョブでフィルタリング条件が指定されている場合、特定の状況でデータロード中にBEがクラッシュすることがあります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- レプリカ操作のリプレイの失敗がFEをクラッシュさせる可能性があります。[#32295](https://github.com/StarRocks/starrocks/pull/32295)
- FEパラメータ `recover_with_empty_tablet` を `true` に設定すると、FEがクラッシュする可能性があります。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- "get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction" エラーがクエリで返されます。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- ウィンドウ関数を含むクエリがBEをクラッシュさせる可能性があります。[#33671](https://github.com/StarRocks/starrocks/pull/33671)
- `show proc '/statistic'` を実行するとデッドロックが発生する可能性があります。[#34237](https://github.com/StarRocks/starrocks/pull/34237/files)
- 大量のデータが永続インデックスが有効なプライマリキーテーブルにロードされるとエラーがスローされることがあります。[#34566](https://github.com/StarRocks/starrocks/pull/34566)
- StarRocks が v2.4 より前のバージョンから後のバージョンにアップグレードされた後、コンパクションのスコアが予期しないほど上昇することがあります。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- `INFORMATION_SCHEMA` が MariaDB ODBC を使用してクエリされると、`schemata` ビューで返される `CATALOG_NAME` 列には `null` の値のみが保持されます。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- Stream Load ジョブが **PREPARD** 状態の間にスキーマ変更が実行されている場合、ジョブによってロードされるべきソースデータの一部が失われる可能性があります。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- HDFS ストレージパスの末尾に二つ以上のスラッシュ(`/`)を含めると、HDFSからデータのバックアップとリストアに失敗することがあります。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- ローディングタスクまたはクエリを実行すると、FEが停止する可能性があります。[#34569](https://github.com/StarRocks/starrocks/pull/34569)

## 2.5.14

リリース日: 2023年11月14日

### 改善点

- システムデータベース `INFORMATION_SCHEMA` の `COLUMNS` テーブルは、ARRAY、MAP、およびSTRUCT列を表示できます。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 互換性の変更

#### システム変数

- DECIMAL型からSTRING型へのCBOのデータ変換を制御するセッション変数 `cbo_decimal_cast_string_strict` が追加されました。この変数が `true` に設定されている場合、v2.5.xおよびそれ以降のバージョンに組み込まれたロジックが有効になり、システムは厳密な変換（つまり、システムは生成された文字列を切り捨て、スケールの長さに基づいて0を埋めます）を実装します。この変数が `false` に設定されている場合、v2.5.x以前のバージョンに組み込まれたロジックが有効になり、システムはすべての有効な桁を処理し、文字列を生成します。デフォルト値は `true` です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- DECIMAL型データとSTRING型データのデータ比較に使用するデータ型を指定するセッション変数 `cbo_eq_base_type` が追加されました。デフォルト値は `VARCHAR` で、DECIMAL も有効な値です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)

### バグ修正

以下の問題が修正されました:

- ON条件がサブクエリとネストされている場合、エラー `java.lang.IllegalStateException: null` が報告されます。[#30876](https://github.com/StarRocks/starrocks/pull/30876)
- `INSERT INTO SELECT ... LIMIT` を正常に実行した直後に COUNT(*) が実行されると、レプリカ間でCOUNT(*)の結果が一貫しない場合があります。[#24435](https://github.com/StarRocks/starrocks/pull/24435)
- cast() 関数で指定されたターゲットデータ型が元のデータ型と同じ場合、特定のデータ型に対してBEがクラッシュすることがあります。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- 特定のパス形式がBroker Load経由でデータロード中に使用されると、`msg:Fail to parse columnsFromPath, expected: [rec_dt]` エラーが発生します。[#32721](https://github.com/StarRocks/starrocks/issues/32721)
- 3.xへのアップグレード中、特定の特性を持つテーブルでCompactionが実行されると、一部の列の型もアップグレードされます（例: DecimalがDecimal v3にアップグレードされる場合）、その際にBEがクラッシュします。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードしている場合、高度に並行したロードジョブがあり、HTTPおよびScanスレッドの数が上限に達した場合、ロードジョブが予期せず中断されます。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurl が呼び出されるとBEがクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- プライマリキーテーブルにBITMAP列を追加しようとすると、次のエラーが発生します: `Analyze columnDef error: No aggregate function specified for 'userid'`。[#31763](https://github.com/StarRocks/starrocks/pull/31763)
- 永続インデックスが有効なプライマリキーテーブルに長時間かつ頻繁にデータをロードすると、BEがクラッシュすることがあります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効になっている場合、クエリ結果が正しくありません。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- プライマリキーテーブルを作成する際にnullableなソートキーを指定すると、コンパクションに失敗することがあります。[#29225](https://github.com/StarRocks/starrocks/pull/29225)
- 複雑なJoinクエリの場合、"StarRocks planner use long time 10000 ms in logical phase" エラーが発生することがあります。[#34177](https://github.com/StarRocks/starrocks/pull/34177)

## 2.5.13

リリース日: 2023年9月28日

### 改善点

- Window関数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMP は今やORDER BY句およびWindow句をサポートしています。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型データのクエリ中に小数のオーバーフローが発生した場合、NULLの代わりにエラーが返されます。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドを実行すると、MySQLと結果が一貫します。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 削除されたタブレットに対応するRowsetがクリーンアップされ、BE起動時のメモリ使用量が削減されます。[#30625](https://github.com/StarRocks/starrocks/pull/30625)
### バグの修正

以下の問題が修正されました：

- Spark ConnectorまたはFlink Connectorを使用してStarRocksからデータを読み取る際にエラー"Set cancelled by MemoryScratchSinkOperator"が発生する問題が修正されました。 [#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 集約関数を含むORDER BY句でクエリを実行中にエラー"java.lang.IllegalStateException: null"が発生する問題が修正されました。 [#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 非アクティブなマテリアライズド・ビューが存在する場合に、FEが再起動に失敗する問題が修正されました。 [#30015](https://github.com/StarRocks/starrocks/pull/30015)
- 重複するパーティションにINSERT OVERWRITE操作を行うと、メタデータが破損し、FEの再起動が失敗する問題が修正されました。 [#27545](https://github.com/StarRocks/starrocks/pull/27545)
- プライマリキーのテーブルに存在しない列をユーザーが変更しようとした際にエラー"java.lang.NullPointerException: null"が発生する問題が修正されました。 [#30366](https://github.com/StarRocks/starrocks/pull/30366)
- パーティショニングされたStarRocks外部テーブルにデータをロードする際にエラー"get TableMeta failed from TNetworkAddress"が発生する問題が修正されました。 [#30124](https://github.com/StarRocks/starrocks/pull/30124)
- 特定のシナリオで、CloudCanalを介してデータをロードする際にエラーが発生する問題が修正されました。 [#30799](https://github.com/StarRocks/starrocks/pull/30799)
- Flink Connectorを使用してデータをロードするかDELETEおよびINSERT操作を実行する際にエラー"current running txns on db xxx is 200, larger than limit 200"が発生する問題が修正されました。 [#18393](https://github.com/StarRocks/starrocks/pull/18393)
- 集約関数を含むHAVING句を使用する非同期マテリアライズド・ビューがクエリを正しく書き換えられない問題が修正されました。 [#29976](https://github.com/StarRocks/starrocks/pull/29976)

## 2.5.12

リリース日：2023年9月4日

### 改善点

- SQL内のコメントが監査ログに保持されるようになりました。 [#29747](https://github.com/StarRocks/starrocks/pull/29747)
- INSERT INTO SELECTのCPUおよびメモリの統計情報が監査ログに追加されました。 [#29901](https://github.com/StarRocks/starrocks/pull/29901)

### バグの修正

以下の問題が修正されました：

- ブローカーロードを使用してデータをロードする際、一部のフィールドのNOT NULL属性がBEのクラッシュや"msg:mismatched row count"エラーの原因となる問題が修正されました。 [#29832](https://github.com/StarRocks/starrocks/pull/29832)
- ORC形式のファイルへのクエリが失敗する問題が修正されました。これは、Apache ORCのバグ修正ORC-1304 ([apache/orc#1299](https://github.com/apache/orc/pull/1299))がマージされていないためです。 [#29804](https://github.com/StarRocks/starrocks/pull/29804)
- プライマリキーテーブルのリストアがBEの再起動後にメタデータの不整合を引き起こす問題が修正されました。 [#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 2.5.11

リリース日：2023年8月28日

### 改善点

- WHERE句での全ての複合述語および全ての式に対して暗黙的な変換をサポートするようになりました。 [セッション変数](https://docs.starrocks.io/en-us/3.1/reference/System_variable) `enable_strict_type`を使用して、暗黙的な変換を有効または無効にできます。デフォルト値は`false`です。 [#21870](https://github.com/StarRocks/starrocks/pull/21870)
- Icebergカタログを作成する際に`hive.metastore.uri`を指定しなかった場合、エラープロンプトがより正確になるように最適化されました。 [#16543](https://github.com/StarRocks/starrocks/issues/16543)
- エラーメッセージ`xxx too many versions xxx`に追加のプロンプトが追加されました。 [#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 動的パーティショニングが`year`をパーティショニング単位としてさらにサポートされるようになりました。 [#28386](https://github.com/StarRocks/starrocks/pull/28386)

### バグの修正

以下の問題が修正されました：
- 複数のレプリカを持つテーブルにデータをロードする際、一部のテーブルのパーティションが空の場合、大量の無効なログレコードが書き込まれる問題が修正されました。 [#28824](https://github.com/StarRocks/starrocks/issues/28824)
- WHERE条件でのフィールドがBITMAPまたはHLLフィールドである場合、DELETE操作が失敗する問題が修正されました。 [#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 同期呼び出し（SYNC MODE）を介して非同期マテリアライズド・ビューを手動でリフレッシュすると、`information_schema.task_runs`テーブルに複数のINSERT OVERWRITEレコードが生じる問題が修正されました。 [#28060](https://github.com/StarRocks/starrocks/pull/28060)
- エラー状態のタブレットにCLONE操作がトリガーされると、ディスク使用量が増加する問題が修正されました。 [#28488](https://github.com/StarRocks/starrocks/pull/28488)
- Join Reorderが有効になっている場合、クエリ結果が定数列をクエリする際に正しくない問題が修正されました。 [#29239](https://github.com/StarRocks/starrocks/pull/29239)
- SSDとHDD間のタブレット移行中に、FEがBEに過剰な移行タスクを送信すると、BEがOOMの問題に遭遇する問題が修正されました。 [#29055](https://github.com/StarRocks/starrocks/pull/29055)
- `/apache_hdfs_broker/lib/log4j-1.2.17.jar`のセキュリティ脆弱性が修正されました。 [#28866](https://github.com/StarRocks/starrocks/pull/28866)
- Hiveカタログを介してデータをクエリする際、WHERE句でパーティショニング列とOR演算子を使用すると、クエリ結果が正しくない問題が修正されました。 [#28876](https://github.com/StarRocks/starrocks/pull/28876)
- データクエリ中に時々エラー"java.util.ConcurrentModificationException: null"が発生する問題が修正されました。 [#29296](https://github.com/StarRocks/starrocks/pull/29296)
- 非同期マテリアライズド・ビューの基本テーブルが削除されると、FEを再起動できない問題が修正されました。 [#29318](https://github.com/StarRocks/starrocks/pull/29318)
- データを書き込んでいる際に、データベース間に作成された非同期マテリアライズド・ビューがリーダーFEでデッドロックが発生する問題が修正されました。 [#29432](https://github.com/StarRocks/starrocks/pull/29432)

## 2.5.10

リリース日：2023年8月7日

### 新機能

- 集約関数[COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、および[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)をサポートしました。
- 次の[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)をサポートしました：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMP。

### 改善点

- TabletCheckerのスケジューリングロジックが最適化され、修復されないタブレットが繰り返しスケジューリングされるのを防ぎました。 [#27648](https://github.com/StarRocks/starrocks/pull/27648)
- スキーマ変更とルーチンロードが同時に発生した場合、スキーマ変更が先に完了した場合、ルーチンロードジョブが失敗する可能性がある問題に対するエラーメッセージが最適化されました。 [#28425](https://github.com/StarRocks/starrocks/pull/28425)
- 外部テーブルを作成する際にNOT NULLの列を定義することが禁止されました（NOT NULLの列が定義されていると、アップグレード後にエラーが発生し、テーブルを再作成する必要があります）。外部カタログは、外部テーブルの代わりに、v2.3.0から推奨されます。 [#25485](https://github.com/StarRocks/starrocks/pull/25441)
- Broker Loadのリトライでエラーが発生した場合にエラーメッセージが追加され、データのロード中のトラブルシューティングとデバッグが容易になりました。 [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- UPSERTおよびDELETE操作を含むロードジョブが大規模なデータ書き込みをサポートしました。 [#17264](https://github.com/StarRocks/starrocks/pull/17264)
- マテリアライズド・ビューを使用したクエリの再構築が最適化されました。 [#27934](https://github.com/StarRocks/starrocks/pull/27934) [#25542](https://github.com/StarRocks/starrocks/pull/25542) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#27557](https://github.com/StarRocks/starrocks/pull/27557) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#26957](https://github.com/StarRocks/starrocks/pull/26957) [#27728](https://github.com/StarRocks/starrocks/pull/27728) [#27900](https://github.com/StarRocks/starrocks/pull/27900)

### バグの修正

以下の問題が修正されました：
- CASTを使用して文字列を配列に変換する場合、入力に定数が含まれていると結果が不正確になる可能性があります。[#19793](https://github.com/StarRocks/starrocks/pull/19793)
- ORDER BYおよびLIMITが含まれる場合、SHOW TABLETは不正確な結果を返します。[#23375](https://github.com/StarRocks/starrocks/pull/23375)
- Materialized viewsに対するOuter joinおよびAnti joinの再構築エラーがあります。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- FEにおける不正確なテーブルレベルのスキャン統計は、テーブルのクエリおよびロードに対して正確でないメトリクスを引き起こします。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 現在の長いリンクを使用してメタストアにアクセスした際に例外が発生します。msg: 最終イベントIDに基づいて次の通知を取得できませんでした: 707602。これは、HMSにイベントリスナーが構成されている場合にFEログで報告されます。[#21056](https://github.com/StarRocks/starrocks/pull/21056)
- ソートキーがパーティション化されたテーブルの場合、クエリ結果は安定しません。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- Spark Loadを使用してロードされたデータは、バケット列がDATE、DATETIME、またはDECIMAL列の場合、間違ったバケットに分散される可能性があります。[#27005](https://github.com/StarRocks/starrocks/pull/27005)
- regex_replace関数は、特定のシナリオでBEのクラッシュを引き起こす場合があります。[#27117](https://github.com/StarRocks/starrocks/pull/27117)
- sub_bitmap関数の入力がBITMAP値でない場合、BEがクラッシュします。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- Join Reorderが有効な場合、クエリの結果が"Unknown error"となります。[#27472](https://github.com/StarRocks/starrocks/pull/27472)
- 平均行サイズの不正確な推定により、Primary Keyの部分更新が過度に大きなメモリを占有します。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 低カーディナリティの最適化が有効になっている場合、一部のINSERTジョブは`[42000][1064] Dict Decode failed, Dict can't take cover all key :0`を返します。[#26463](https://github.com/StarRocks/starrocks/pull/26463)
- HDFSからデータをロードするために作成されたBroker Loadジョブで`"hadoop.security.authentication" = "simple"`を指定した場合、ジョブが失敗します。[#27774](https://github.com/StarRocks/starrocks/pull/27774)
- materialized viewsの更新モードを変更すると、リーダーFEとフォロワーFEの間で整合性のないメタデータが発生します。[#28082](https://github.com/StarRocks/starrocks/pull/28082) [#28097](https://github.com/StarRocks/starrocks/pull/28097)
- SHOW CREATE CATALOGおよびSHOW RESOURCESを使用して特定の情報をクエリすると、パスワードが隠されません。[#28059](https://github.com/StarRocks/starrocks/pull/28059)
- ブロックされたLabelCleanerスレッドによって引き起こされるFEメモリリークが修正されました。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 2.5.9

リリース日: 2023年7月19日

### 新機能

- materialized viewとは異なる種類のjoinを含むクエリは再構築できます。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改善

- 現在のStarRocksクラスターが宛先クラスターであるStarRocks外部テーブルは作成できません。[#25441](https://github.com/StarRocks/starrocks/pull/25441)
- materialized viewの出力列に含まれていないが、materialized viewの述語に含まれる場合、クエリは依然として再構築できます。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- データベース`Information_schema`のテーブル`tables_config`に新しいフィールド`table_id`が追加されました。`table_id`を使用して、`tables_config`を`be_tablets`と結合して、タブレットが属するデータベースとテーブルの名前をクエリできます。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### バグ修正

次の問題が修正されました:

- 重複キーのテーブルに対してCount Distinctの結果が不正確です。[#24222](https://github.com/StarRocks/starrocks/pull/24222)
- Joinキーが大きなBINARY列の場合、BEがクラッシュする可能性があります。[#25084](https://github.com/StarRocks/starrocks/pull/25084)
- 構造体に挿入するCHARデータの長さが、構造体列で定義された最大CHAR長を超える場合、INSERT操作がハングします。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- coalesce()の結果が不正確です。[#26250](https://github.com/StarRocks/starrocks/pull/26250)
- データが復元された後、BEとFEの間でタブレットのバージョン番号が一貫していません。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- パーティションは自動的に復元されたテーブルに作成されません。[#26813](https://github.com/StarRocks/starrocks/pull/26813)

## 2.5.8

リリース日: 2023年6月30日

### 改善

- 非パーティションテーブルにパーティションが追加された場合のエラーメッセージが最適化されました。[#25266](https://github.com/StarRocks/starrocks/pull/25266)
- テーブルの[auto tablet distribution policy](../table_design/Data_distribution.md#determine-the-number-of-buckets)が最適化されました。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- CREATE TABLE文のデフォルトコメントが最適化されました。[#24803](https://github.com/StarRocks/starrocks/pull/24803)
- 非同期materialized viewsの手動リフレッシュが最適化されました。REFRESH MATERIALIZED VIEW WITH SYNC MODE構文を使用して、materialized viewのリフレッシュタスクを同期的に呼び出すことができます。[#25910](https://github.com/StarRocks/starrocks/pull/25910)

### バグ修正

次の問題が修正されました:

- 非同期materialized viewのCOUNT結果がUnion結果に基づいて構築された場合、不正確になる可能性があります。[#24460](https://github.com/StarRocks/starrocks/issues/24460)
- ルートパスワードを強制的にリセットしようとしても"Unknown error"が報告されます。[#25492](https://github.com/StarRocks/starrocks/pull/25492)
- INSERT OVERWRITEが3つ未満の稼働中のBEを持つクラスターで実行された場合、不正確なエラーメッセージが表示されます。[#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 2.5.7

リリース日: 2023年6月14日

### 新機能

- `ALTER MATERIALIZED VIEW <mv_name> ACTIVE`を使用して非アクティブなmaterialized viewsを手動でアクティブ化できます。このSQLコマンドを使用して、基本テーブルが削除され、その後再作成されたmaterialized viewsをアクティブ化できます。詳細については[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を参照してください。[#24001](https://github.com/StarRocks/starrocks/pull/24001)
- StarRocksは、テーブルを作成したりパーティションを追加した際に、適切な数のタブレットを自動的に設定します。これにより、手動操作が不要になります。詳細については[Determine the number of tablets](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。[#10614](https://github.com/StarRocks/starrocks/pull/10614)

### 改善

- 外部テーブルクエリで使用されるScanノードのI/O並列処理が最適化され、メモリ使用量が減少し、安定性が向上しました。[#23617](https://github.com/StarRocks/starrocks/pull/23617) [#23624](https://github.com/StarRocks/starrocks/pull/23624) [#23626](https://github.com/StarRocks/starrocks/pull/23626)
- Broker Loadジョブのエラーメッセージが最適化されました。エラーメッセージには、リトライ情報とエラーが発生したファイルの名前が含まれています。[#18038](https://github.com/StarRocks/starrocks/pull/18038) [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- CREATE TABLEのタイムアウト時に返されるエラーメッセージが最適化され、パラメータの調整方法が追加されました。[#24510](https://github.com/StarRocks/starrocks/pull/24510)
- ALTER TABLEが失敗する理由としてテーブルの状態が正常でない場合のエラーメッセージが最適化されました。[#24381](https://github.com/StarRocks/starrocks/pull/24381)
- CREATE TABLE文で全角スペースが無視されるようになりました。[#23885](https://github.com/StarRocks/starrocks/pull/23885)
- ブローカーアクセスタイムアウトを最適化して、ブローカーのロードジョブの成功率を向上させました。[#22699](https://github.com/StarRocks/starrocks/pull/22699)
- プライマリキーテーブルの場合、SHOW TABLETによって返される`VersionCount`フィールドは、保留中の状態のローセットを含んでいます。[#23847](https://github.com/StarRocks/starrocks/pull/23847)
- 永続インデックスポリシーを最適化しました。[#22140](https://github.com/StarRocks/starrocks/pull/22140)

### バグ修正

以下の問題を修正しました：

- StarRocksにParquetデータをロードする際に、型変換中にDATETIME値がオーバーフローし、データエラーが発生する問題を修正しました。[#22356](https://github.com/StarRocks/starrocks/pull/22356)
- 動的パーティショニングを無効にした後に、バケツ情報が失われる問題を修正しました。[#22595](https://github.com/StarRocks/starrocks/pull/22595)
- CREATE TABLEステートメントでサポートされていないプロパティを使用すると、ヌルポインターエラー（NPE）が発生する問題を修正しました。[#23859](https://github.com/StarRocks/starrocks/pull/23859)
- `information_schema`でのテーブル権限フィルタリングが無効になる問題を修正しました。その結果、ユーザーは許可されていないテーブルを表示できるようになっていました。[#23804](https://github.com/StarRocks/starrocks/pull/23804)
- SHOW TABLE STATUSで返される情報が不完全な問題を修正しました。[#24279](https://github.com/StarRocks/starrocks/issues/24279)
- スキーマ変更中にデータロードと同時にスキーマ変更が行われると、スキーマ変更が時々停滞する問題を修正しました。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- RocksDB WALのフラッシュがbrpcワーカーのbスレッドの処理を妨げ、プライマリキーテーブルへの高頻度データロードが中断される問題を修正しました。[#22489](https://github.com/StarRocks/starrocks/pull/22489)
- StarRocksでサポートされていないTIME型の列が正常に作成されてしまう問題を修正しました。[#23474](https://github.com/StarRocks/starrocks/pull/23474)
- マテリアライズドビューのUnion再構成が失敗する問題を修正しました。[#22922](https://github.com/StarRocks/starrocks/pull/22922)

## 2.5.6

リリース日: 2023年5月19日

### 改善

- `INSERT INTO ... SELECT`の有効期限が`thrift_server_max_worker_thread`の値が小さいために切れた際に報告されるエラーメッセージを最適化しました。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- CTASを使用して作成されたテーブルについて、デフォルトで3つのレプリカがあり、これは通常のテーブルのデフォルトのレプリカ数と一致しています。[#22854](https://github.com/StarRocks/starrocks/pull/22854)

### バグ修正

- パーティションの切り捨てが分割名の大文字と小文字を区別するために失敗する問題を修正しました。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- MATERIALIZE認定BEの一時パーティション作成に失敗するため、BEの動かしが失敗する問題を修正しました。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- ARRAY値を必要とする動的FEパラメーターを空の配列に設定できない問題を修正しました。[#22225](https://github.com/StarRocks/starrocks/pull/22225)
- `partition_refresh_number`プロパティが指定されたマテリアライズドビューが完全にリフレッシュに失敗する可能性がある問題を修正しました。[#21619](https://github.com/StarRocks/starrocks/pull/21619)
- SHOW CREATE TABLEでクラウドの資格情報をマスクするため、メモリ内に誤った資格情報が残る問題を修正しました。[#21311](https://github.com/StarRocks/starrocks/pull/21311)
- 外部テーブルを経由してクエリされる一部のORCファイルに対して述語が有効にならない問題を修正しました。[#21901](https://github.com/StarRocks/starrocks/pull/21901)
- 列名に大文字と小文字が使用される場合、min-maxフィルターが適切に処理できない問題を修正しました。[#22626](https://github.com/StarRocks/starrocks/pull/22626)
- 遅延マテリアライゼーションにより、複雑なデータ型（STRUCTまたはMAP）のクエリでエラーが発生する問題を修正しました。[#22862](https://github.com/StarRocks/starrocks/pull/22862)
- プライマリキーテーブルの復元時に発生する問題を修正しました。[#23384](https://github.com/StarRocks/starrocks/pull/23384)

## 2.5.5

リリース日: 2023年4月28日

### 新機能

プライマリキーテーブルのタブレット状態をモニタリングするためのメトリックを追加しました：

- FEメトリック`err_state_metric`を追加しました。
- `/statistic/`を表示するための`SHOW PROC`の出力に`ErrorStateTabletNum`列を追加し、**err_state**タブレットの数を表示します。
- `/statistic/<db_id>/`を表示するための`SHOW PROC`の出力に`ErrorStateTablets`列を追加し、**err_state**タブレットのIDを表示します。

詳細については、[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)を参照してください。

### 改善

- 複数のBEが追加されるときのディスクバランスの速度を最適化しました。[#19418](https://github.com/StarRocks/starrocks/pull/19418)
- BEがSSDとHDDの両方をストレージデバイスとして使用する場合、`storage_cooldown_time`プロパティが指定されている場合、StarRocksは`storage_medium`を`SSD`に設定します。それ以外の場合、`storage_medium`を`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- Unique Keyテーブルのパフォーマンスを最適化し、値列の統計情報の収集を禁止しました。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

- Colocationテーブルでは、`ADMIN SET REPLICA STATUS PROPERTIES`などのステートメントを使用してレプリカの状態を手動で`bad`に指定することができます。BEの数がレプリカの数以下の場合、損傷したレプリカを修復できなくなります。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- BEが起動した後、そのプロセスは存在しているが、BEポートが有効にならない問題を修正しました。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- サブクエリがウィンドウ関数とネストされた場合に誤った結果が返される集計クエリの問題を修正しました。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- マテリアライズドビュー（MV）が初めてリフレッシュされる際に`auto_refresh_partitions_limit`が効果を発揮しないため、すべてのパーティションがリフレッシュされてしまう問題を修正しました。[#19759](https://github.com/StarRocks/starrocks/issues/19759)
- CSV Hive外部テーブルをクエリする際に、配列データがMAPやSTRUCTなどの複雑なデータと入れ子になっている場合にエラーが発生する問題を修正しました。[#20233](https://github.com/StarRocks/starrocks/pull/20233)
- Sparkコネクタを使用したクエリがタイムアウトする問題を修正しました。[#20264](https://github.com/StarRocks/starrocks/pull/20264)
- 2つのレプリカテーブルの1つのレプリカが損傷している場合、テーブルが復旧できない問題を修正しました。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- MVクエリの再構成が失敗するためのクエリの失敗問題を修正しました。[#19549](https://github.com/StarRocks/starrocks/issues/19549)
- データベースのロックによりメトリックインタフェースが有効期限切れになる問題を修正しました。[#20790](https://github.com/StarRocks/starrocks/pull/20790)
- Broadcast Joinのために誤った結果が返される問題を修正しました。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- CREATE TABLEでサポートされていないデータ型が使用された場合にNPEが返される問題を修正しました。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- クエリキャッシュ機能でwindow_funnel()を使用した場合に発生する問題を修正しました。[#21474](https://github.com/StarRocks/starrocks/issues/21474)
- CTEが再構成された後に最適化計画の選択に予期しない時間がかかる問題を修正しました。[#16515](https://github.com/StarRocks/starrocks/pull/16515)

## 2.5.4

リリース日: 2023年4月4日

### 改善

- クエリプランニング中のマテリアライズドビューのクエリの再構成のパフォーマンスを最適化し、クエリプランニングにかかる時間を約70%低減しました。[#19579](https://github.com/StarRocks/starrocks/pull/19579)
- 型推論ロジックを最適化しました。たとえば、`SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;`のような`SELECT sum(CASE WHEN XXX);`のようなクエリで、定数`0`が含まれている場合、クエリの加速のために自動的に事前集約が有効になります。[#19474](https://github.com/StarRocks/starrocks/pull/19474)
- `SHOW CREATE VIEW`を使用してマテリアライズドビューの作成ステートメントを表示することをサポートしました。[#19999](https://github.com/StarRocks/starrocks/pull/19999)
- BEノード間の単一のbRPCリクエストで、2 GB以上のパケットの送信をサポートしています。[#20283](https://github.com/StarRocks/starrocks/pull/20283) [#20230](https://github.com/StarRocks/starrocks/pull/20230)
- [SHOW CREATE CATALOG](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)を使用して、外部カタログの作成ステートメントをクエリすることをサポートしています。

### バグフィックス

以下のバグが修正されています：

- マテリアライズドビューのクエリの後、低基数最適化のグローバル辞書が効果を持たないことがあります。[#19615](https://github.com/StarRocks/starrocks/pull/19615)
- マテリアライズドビューのクエリが書き直されない場合、クエリが失敗します。[#19774](https://github.com/StarRocks/starrocks/pull/19774)
- マテリアライズドビューがプライマリキーやユニークキーのテーブルを基に作成された場合、そのマテリアライズドビューのクエリは書き直すことができません。[#19600](https://github.com/StarRocks/starrocks/pull/19600)
- マテリアライズドビューの列名は大文字と小文字を区別します。ただし、テーブルを作成するとき、テーブルの「PROPERTIES」で列名が間違っていてもエラーメッセージが表示されず、さらにそのテーブルに作成されたマテリアライズドビューのクエリの書き直しが失敗します。[#19780](https://github.com/StarRocks/starrocks/pull/19780)
- マテリアライズドビューのクエリが書き直された後、クエリプランにはパーティション列ベースの無効な述語が含まれることがあり、クエリのパフォーマンスに影響します。[#19784](https://github.com/StarRocks/starrocks/pull/19784)
- 新しく作成されたパーティションにデータがロードされると、マテリアライズドビューのクエリの書き直しが失敗することがあります。[#20323](https://github.com/StarRocks/starrocks/pull/20323)
- マテリアライズドビューの作成時に`"storage_medium" = "SSD"`を設定すると、マテリアライズドビューのリフレッシュが失敗します。[#19539](https://github.com/StarRocks/starrocks/pull/19539) [#19626](https://github.com/StarRocks/starrocks/pull/19626)
- プライマリキーテーブルで同時コンパクションが発生することがあります。[#19692](https://github.com/StarRocks/starrocks/pull/19692)
- 大量のDELETE操作後にコンパクションがすぐに発生しないことがあります。[#19623](https://github.com/StarRocks/starrocks/pull/19623)
- ステートメントの表現に複数の低基数列が含まれている場合、その表現が適切に書き直されないことがあります。その結果、低基数最適化のグローバル辞書が効果を持ちません。[#20161](https://github.com/StarRocks/starrocks/pull/20161)

## 2.5.3

リリース日: 2023年3月10日

### 改善

- マテリアライズドビュー（MVs）のクエリの書き直しを最適化しました。
  - アウタージョインとクロスジョインを含むクエリの書き直しをサポートします。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - MVsのデータスキャンロジックを最適化し、書き直ししたクエリをさらに高速化しました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - シングルテーブルの集約クエリの書き直し機能を強化しました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - ビューデルタシナリオでの書き直し能力を強化しました。これは、クエリされるテーブルがMVsの基本テーブルのサブセットである場合です。[#18800](https://github.com/StarRocks/starrocks/pull/18800)
- ウィンドウ関数RANK()がフィルターまたはソートキーとして使用される場合のパフォーマンスとメモリ使用量を最適化しました。[#17553](https://github.com/StarRocks/starrocks/issues/17553)

### バグフィックス

以下のバグが修正されています：

- 配列データでの`[]`のnullリテラルのエラーが修正されました。[#18563](https://github.com/StarRocks/starrocks/pull/18563)
- 一部の複雑なクエリシナリオでの低基数最適化辞書の誤用。辞書のマッピングチェックが辞書を適用する前に追加されました。[#17318](https://github.com/StarRocks/starrocks/pull/17318)
- 単一のBE環境で、ローカルシャッフルがGROUP BYで重複した結果をもたらす問題が修正されました。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- パーティション関連のPROPERTIESが非パーティションMVsに誤って使用されると、MVのリフレッシュが失敗する可能性があります。ユーザーがMVを作成する際にパーティションPROPERTIESのチェックが行われるようになりました。[#18741](https://github.com/StarRocks/starrocks/pull/18741)
- ParquetのRepetition列の解析エラーが修正されました。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 取得された列のnullable情報が正しくありません。解決策：CTASを使用してプライマリキーテーブルを作成すると、プライマリキーカラムのみが非NULLで、プライマリキー以外の列はNULL可能になります。[#16431](https://github.com/StarRocks/starrocks/pull/16431)
- プライマリキーテーブルからデータを削除することによって引き起こされる問題が修正されました。[#18768](https://github.com/StarRocks/starrocks/pull/18768)

## 2.5.2

リリース日: 2023年2月21日

### 新機能

- AWS S3とAWS GlueへのアクセスにInstance ProfileとAssumed Roleベースの認証方法を使用することをサポートしています。[#15958](https://github.com/StarRocks/starrocks/pull/15958)
- bit_shift_left、bit_shift_right、bit_shift_right_logicalといったビット関数をサポートしています。[#14151](https://github.com/StarRocks/starrocks/pull/14151)

### 改善

- 大量の集約クエリを含むクエリのピークメモリ使用量を大幅に減らすためにメモリ解放ロジックを最適化しました。[#16913](https://github.com/StarRocks/starrocks/pull/16913)
- ソートのメモリ使用量を削減しました。ウィンドウ関数やソートを含むクエリのメモリ消費量が半減しました。[#16937](https://github.com/StarRocks/starrocks/pull/16937) [#17362](https://github.com/StarRocks/starrocks/pull/17362) [#17408](https://github.com/StarRocks/starrocks/pull/17408)

### バグフィックス

以下のバグが修正されています：

- MAPとARRAYデータを含むApache Hive外部テーブルのリフレッシュができない問題が修正されました。[#17548](https://github.com/StarRocks/starrocks/pull/17548)
- Supersetがマテリアライズドビューの列のタイプを識別できない問題が修正されました。[#17686](https://github.com/StarRocks/starrocks/pull/17686)
- BIの接続が失敗する原因となるSET GLOBAL/SESSION TRANSACTIONが解析できない問題が修正されました。[#17295](https://github.com/StarRocks/starrocks/pull/17295)
- コロケートグループ内のダイナミックパーティションテーブルのバケット数を変更できない問題が修正され、エラーメッセージが返されるようになりました。[#17418](https://github.com/StarRocks/starrocks/pull/17418/)
- 準備ステージの失敗による潜在的な問題を修正しました。[#17323](https://github.com/StarRocks/starrocks/pull/17323)

### 動作の変更

- `enable_experimental_mv`のデフォルト値を`false`から`true`に変更しました。これにより、非同期マテリアライズドビューがデフォルトで有効になります。
- 予約キーワードリストにCHARACTERを追加しました。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.1

リリース日: 2023年2月5日

### 改善

- 外部カタログに基づいて作成された非同期マテリアライズドビューはクエリの書き直しをサポートしました。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#15791](https://github.com/StarRocks/starrocks/issues/15791)
- 自動CBO統計情報の収集のためのコレクション期間をユーザーが指定できるようにし、自動的な完全な収集によるクラスタのパフォーマンスの揺らぎを防ぎます。[#14996](https://github.com/StarRocks/starrocks/pull/14996)
- スラストサーバーキューを追加しました。INSERT INTO SELECT中にすぐに処理できないリクエストをThriftサーバーキューに保留し、リクエストが拒否されるのを防ぎます。[#14571](https://github.com/StarRocks/starrocks/pull/14571)
- FEパラメータ`default_storage_medium`を非推奨にしました。ユーザーがテーブルを作成する際に`storage_medium`が明示的に指定されていない場合、システムはBEディスクのタイプに基づいて自動的にテーブルのストレージメディアを推測します。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_VIEW.md)の`storage_medium`の記述をご覧ください。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### バグフィックス

以下のバグが修正されています：
- SET PASSWORDによって発生したヌルポインター例外（NPE）です。 [#15247](https://github.com/StarRocks/starrocks/pull/15247)
- 空のキーを持つJSONデータは解析できません。 [#16852](https://github.com/StarRocks/starrocks/pull/16852)
- 無効な型のデータを配列データに正常に変換できます。 [#16866](https://github.com/StarRocks/starrocks/pull/16866)
- 例外が発生した場合、ネステッドループ結合を中断できません。 [#16875](https://github.com/StarRocks/starrocks/pull/16875)

### 挙動の変更

- FEパラメーター`default_storage_medium`は非推奨となりました。表のストレージメディアは、システムによって自動的に推測されます。 [#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.5.0

リリース日：2023年1月22日

### 新機能

- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)および[Hudi外部テーブル](../data_source/External_table.md#deprecated-hudi-external-table)を使用して、Merge On Readテーブルのクエリをサポートします。 [#6780](https://github.com/StarRocks/starrocks/pull/6780)
- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、Hudiカタログ、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用して、STRUCTおよびMAPデータのクエリをサポートします。 [#10677](https://github.com/StarRocks/starrocks/issues/10677)
- [Data Cache](../data_source/data_cache.md)を提供して、HDFSなどの外部ストレージシステムに保存されたホットデータへのアクセスパフォーマンスを向上させます。 [#11597](https://github.com/StarRocks/starrocks/pull/11579)
- [Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)を作成できるようになりました。これにより、Delta Lakeのデータを直接クエリできます。 [#11972](https://github.com/StarRocks/starrocks/issues/11972)
- Hive、Hudi、およびIcebergカタログはAWS Glueと互換性があります。 [#12249](https://github.com/StarRocks/starrocks/issues/12249)
- [ファイル外部テーブル](../data_source/file_external_table.md)を作成できるようになりました。これにより、HDFSやオブジェクトストアからのParquetおよびORCファイルを直接クエリできます。 [#13064](https://github.com/StarRocks/starrocks/pull/13064)
- Hive、Hudi、Icebergカタログ、およびマテリアライズドビューに基づくマテリアライズドビューの作成をサポートします。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。 [#11116](https://github.com/StarRocks/starrocks/issues/11116) [#11873](https://github.com/StarRocks/starrocks/pull/11873)
- 主キーテーブルを使用するテーブルの条件付き更新をサポートします。詳細については、[主キーテーブルへのデータの変更](../loading/Load_to_Primary_Key_tables.md)を参照してください。 [#12159](https://github.com/StarRocks/starrocks/pull/12159)
- [Query Cache](../using_starrocks/query_cache.md)をサポートします。これにより、クエリの中間計算結果を格納し、高い並行性を持つシンプルなクエリのQPSを向上し、平均レイテンシを減少させます。 [#9194](https://github.com/StarRocks/starrocks/pull/9194)
- Broker Loadジョブの優先度を指定できるようになりました。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。 [#11029](https://github.com/StarRocks/starrocks/pull/11029)
- StarRocksネイティブテーブルのデータロードのレプリカ数を指定できるようになりました。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。 [#11253](https://github.com/StarRocks/starrocks/pull/11253)
- [クエリキュー](../administration/query_queues.md)をサポートします。 [#12594](https://github.com/StarRocks/starrocks/pull/12594)
- データローディングのリソース消費量を制限することで、データロードによって占有されたリソースを分離する機能をサポートします。詳細については、[リソースグループ](../administration/resource_group.md)を参照してください。 [#12606](https://github.com/StarRocks/starrocks/pull/12606)
- StarRocksネイティブテーブルのデータ圧縮アルゴリズムとして、LZ4、Zstd、Snappy、およびZlibを指定できるようになりました。詳細については、[データ圧縮](../table_design/data_compression.md)を参照してください。 [#10097](https://github.com/StarRocks/starrocks/pull/10097) [#12020](https://github.com/StarRocks/starrocks/pull/12020)
- [ユーザー定義変数](../reference/user_defined_variables.md)をサポートします。 [#10011](https://github.com/StarRocks/starrocks/pull/10011)
- [ラムダ式](../sql-reference/sql-functions/Lambda_expression.md)および[array_map](../sql-reference/sql-functions/array-functions/array_map.md)、[array_sum](../sql-reference/sql-functions/array-functions/array_sum.md)、[array_sortby](../sql-reference/sql-functions/array-functions/array_sortby.md)などの高階関数をサポートします。 [#9461](https://github.com/StarRocks/starrocks/pull/9461) [#9806](https://github.com/StarRocks/starrocks/pull/9806) [#10323](https://github.com/StarRocks/starrocks/pull/10323) [#14034](https://github.com/StarRocks/starrocks/pull/14034)
- [window関数](../sql-reference/sql-functions/Window_function.md)の結果をフィルタリングするQUALIFY句を提供します。 [#13239](https://github.com/StarRocks/starrocks/pull/13239)
- uuid()およびuuid_numeric()関数によって返された結果を、テーブルを作成する際の列のデフォルト値として使用できるようになりました。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。 [#11155](https://github.com/StarRocks/starrocks/pull/11155)
- [map_size](../sql-reference/sql-functions/map-functions/map_size.md)、[map_keys](../sql-reference/sql-functions/map-functions/map_keys.md)、[map_values](../sql-reference/sql-functions/map-functions/map_values.md)、[max_by](../sql-reference/sql-functions/aggregate-functions/max_by.md)、[sub_bitmap](../sql-reference/sql-functions/bitmap-functions/sub_bitmap.md)、[bitmap_to_base64](../sql-reference/sql-functions/bitmap-functions/bitmap_to_base64.md)、[host_name](../sql-reference/sql-functions/utility-functions/host_name.md)、および[date_slice](../sql-reference/sql-functions/date-time-functions/date_slice.md)といった関数をサポートします。 [#11299](https://github.com/StarRocks/starrocks/pull/11299) [#11323](https://github.com/StarRocks/starrocks/pull/11323) [#12243](https://github.com/StarRocks/starrocks/pull/12243) [#11776](https://github.com/StarRocks/starrocks/pull/11776) [#12634](https://github.com/StarRocks/starrocks/pull/12634) [#14225](https://github.com/StarRocks/starrocks/pull/14225)

### 改善

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用して外部データをクエリする際のメタデータのアクセスパフォーマンスを最適化しました。 [#11349](https://github.com/StarRocks/starrocks/issues/11349)
- [Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を使用してARRAYデータのクエリをサポートします。 [#9693](https://github.com/StarRocks/starrocks/pull/9693)
- マテリアライズドビューの以下の側面を最適化しました。
  - 非同期マテリアライズドビューは、SPJGタイプのマテリアライズドビューを基に、自動的および透過的なクエリの書き換えをサポートします。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md#rewrite-and-accelerate-queries-with-the-asynchronous-materialized-view)を参照してください。 [#13193](https://github.com/StarRocks/starrocks/issues/13193)
  - 非同期マテリアライズドビューは、複数の非同期リフレッシュメカニズムをサポートします。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md#manually-refresh-an-asynchronous-materialized-view)を参照してください。[#12712](https://github.com/StarRocks/starrocks/pull/12712) [#13171](https://github.com/StarRocks/starrocks/pull/13171) [#13229](https://github.com/StarRocks/starrocks/pull/13229) [#12926](https://github.com/StarRocks/starrocks/pull/12926)
  - マテリアライズドビューのリフレッシュの効率が向上しました。 [#13167](https://github.com/StarRocks/starrocks/issues/13167)
- データローディングの以下の側面を最適化しました。
- 複数のレプリカのシナリオにおいて、「単一のリーダーレプリケーション」モードをサポートすることで、ロードパフォーマンスを最適化しました。データのロードにおいて1倍のパフォーマンス向上が得られます。「単一のリーダーレプリケーション」の詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) の`replicated_storage`を参照してください。[10138](https://github.com/StarRocks/starrocks/pull/10138)
- Broker LoadとSpark Loadは、1つのHDFSクラスタまたは1つのKerberosユーザが構成されている場合、データロードにおいてもはやブローカーに依存する必要がありません。ただし、複数のHDFSクラスタや複数のKerberosユーザが存在する場合は、引き続きブローカーを展開する必要があります。詳細については、[HDFSまたはクラウドストレージからデータをロード](../loading/BrokerLoad.md) および [Apache Spark™を使用したバルクロード](../loading/SparkLoad.md) を参照してください。[9049](https://github.com/starrocks/starrocks/pull/9049) [9228](https://github.com/StarRocks/starrocks/pull/9228)
- 大量の小さなORCファイルがロードされる際には、Broker Loadのパフォーマンスを最適化しました。[11380](https://github.com/StarRocks/starrocks/pull/11380)
- プライマリキーのテーブルへのデータのロード時にメモリ使用量を削減しました。
- `information_schema`データベースとその内部の`tables`および`columns`テーブルを最適化し、新しい`table_config`テーブルを追加しました。詳細については、[Information Schema](../reference/overview-pages/information_schema.md) を参照してください。[10033](https://github.com/StarRocks/starrocks/pull/10033)
- データのバックアップとリストアを最適化しました:
  - 1回の操作でデータベース内の複数のテーブルからデータのバックアップとリストアをサポートします。詳細については、[データのバックアップおよびリストア](../administration/Backup_and_restore.md) を参照してください。[11619](https://github.com/StarRocks/starrocks/issues/11619)
  - プライマリキーのテーブルからのデータのバックアップとリストアをサポートします。詳細については、バックアップとリストアを参照してください。[11885](https://github.com/StarRocks/starrocks/pull/11885)
- 以下の機能を最適化しました:
  - [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) 関数にオプションパラメータを追加し、時間間隔の始まりまたは終わりを返すようにしました。[11216](https://github.com/StarRocks/starrocks/pull/11216)
  - [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md) 関数に新しい`INCREASE`モードを追加し、重複するタイムスタンプの計算を回避するようにしました。[10134](https://github.com/StarRocks/starrocks/pull/10134)
  - [unnest](../sql-reference/sql-functions/array-functions/unnest.md) 関数で複数の引数を指定できるようにしました。[12484](https://github.com/StarRocks/starrocks/pull/12484)
  - lead()およびlag()関数でHLLおよびBITMAPデータのクエリをサポートしました。詳細については、[Window function](../sql-reference/sql-functions/Window_function.md) を参照してください。[12108](https://github.com/StarRocks/starrocks/pull/12108)
  - 以下のARRAY関数でJSONデータのクエリをサポートします: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md), [array_sort](../sql-reference/sql-functions/array-functions/array_sort.md), [array_concat](../sql-reference/sql-functions/array-functions/array_concat.md), [array_slice](../sql-reference/sql-functions/array-functions/array_slice.md),および [reverse](../sql-reference/sql-functions/array-functions/reverse.md)。[13155](https://github.com/StarRocks/starrocks/pull/13155)
  - 一部の関数の使用を最適化しました。`current_date`、`current_timestamp`、`current_time`、`localtimestamp`、および `localtime` 関数は、`()`を使用せずに実行できるようになりました。例えば、 `select current_date;` を直接実行できます。[14319](https://github.com/StarRocks/starrocks/pull/14319)
- FEログから余分な情報を削除しました。[15374](https://github.com/StarRocks/starrocks/pull/15374)

### 不具合修正

以下の不具合が修正されました:

- `append_trailing_char_if_absent()` 関数は、最初の引数が空の場合に不正な結果を返す場合があります。[13762](https://github.com/StarRocks/starrocks/pull/13762)
- `RECOVER` ステートメントを使用してテーブルを復元した後、テーブルが存在しないことがあります。[13921](https://github.com/StarRocks/starrocks/pull/13921)
- `SHOW CREATE MATERIALIZED VIEW` ステートメントが返す結果に、マテリアライズドビューの作成時にクエリステートメントで指定されたデータベースとカタログが含まれない場合があります。[12833](https://github.com/StarRocks/starrocks/pull/12833)
- `waiting_stable` 状態のスキーマ変更ジョブをキャンセルできないことがあります。[12530](https://github.com/StarRocks/starrocks/pull/12530)
- リーダーFEおよび非リーダーFEで `SHOW PROC '/statistic';` コマンドを実行すると、異なる結果が返されることがあります。[12491](https://github.com/StarRocks/starrocks/issues/12491)
- `SHOW CREATE TABLE` によって返される結果における `ORDER BY` 句の位置が正しくありません。[13809](https://github.com/StarRocks/starrocks/pull/13809)
- ユーザーがHive Catalogを使用してHiveデータをクエリする際、FEによって生成された実行計画がパーティションIDを含まない場合、BEがHiveパーティションデータをクエリできないことがあります。[15486](https://github.com/StarRocks/starrocks/pull/15486)。

### 動作の変更

- `AWS_EC2_METADATA_DISABLED` パラメータのデフォルト値を `False` に変更しました。これにより、Amazon EC2のメタデータが取得され、AWSリソースにアクセスできるようになります。
- セッション変数 `is_report_success` の名前を `enable_profile` に変更しました。これは、 `SHOW VARIABLES` ステートメントを使用してクエリできます。
- 4つの予約語 `CURRENT_DATE`、 `CURRENT_TIME`、 `LOCALTIME`、および `LOCALTIMESTAMP` を追加しました。[14319](https://github.com/StarRocks/starrocks/pull/14319)
- テーブルおよびデータベースの最大長さは、1023文字まで拡張されました。[14929](https://github.com/StarRocks/starrocks/pull/14929) [15020](https://github.com/StarRocks/starrocks/pull/15020)
- BE構成項目 `enable_event_based_compaction_framework` と `enable_size_tiered_compaction_strategy` のデフォルト値を `true` に設定しました。これにより、大量のタブレットが存在する場合や単一のタブレットに大量のデータがある場合の圧縮オーバーヘッドが大幅に削減されます。

### アップグレードノート

- クラスタを2.0.x、2.1.x、2.2.x、2.3.x、または2.4.xから2.5.0にアップグレードできます。ただし、ロールバックする必要がある場合は、2.4.xにのみロールバックすることをお勧めします。