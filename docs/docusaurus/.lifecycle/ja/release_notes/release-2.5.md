---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.5

## 2.5.16

リリース日: 2023年12月1日

### バグ修正

以下の問題を修正しました:

- 特定のシナリオで Global Runtime Filter がBEのクラッシュを引き起こすことがあります。 [#35776](https://github.com/StarRocks/starrocks/pull/35776)

## 2.5.15

リリース日: 2023年11月29日

### 改善点

- 遅いリクエストを追跡するために遅いリクエストログを追加しました。 [#33908](https://github.com/StarRocks/starrocks/pull/33908)
- 大量のファイルが存在する場合のParquetおよびORCファイルの読み込みにおけるSpark Loadのパフォーマンスを最適化しました。 [#34787](https://github.com/StarRocks/starrocks/pull/34787)
- ビットマップに関連するいくつかの操作のパフォーマンスを最適化しました:
  - ネストされたループの結合を最適化しました。 [#340804](https://github.com/StarRocks/starrocks/pull/34804) [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor` 関数を最適化しました。 [#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - ビットマップのパフォーマンスを最適化し、メモリ消費を削減するためにCopy on Writeをサポートします。 [#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 互換性の変更

#### パラメータ

- FEの動的パラメータ `enable_new_publish_mechanism` が静的パラメータに変更されました。 パラメータの設定を変更した後にFEを再起動する必要があります。 [#35338](https://github.com/StarRocks/starrocks/pull/35338)

### バグ修正

- Broker Load jobでフィルタリング条件が指定されている場合、特定の状況でBEがクラッシュする可能性があります。 [#29832](https://github.com/StarRocks/starrocks/pull/29832)
- レプリカ操作の再生中に失敗するとFEがクラッシュする可能性があります。 [#32295](https://github.com/StarRocks/starrocks/pull/32295)
- FEパラメータ `recover_with_empty_tablet` を `true` に設定するとFEがクラッシュする可能性があります。 [#33071](https://github.com/StarRocks/starrocks/pull/33071)
- クエリで「get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction」というエラーが返されます。 [#33246](https://github.com/StarRocks/starrocks/pull/33246)
- ウィンドウ関数を含むクエリがBEをクラッシュさせる可能性があります。 [#33671](https://github.com/StarRocks/starrocks/pull/33671)
- `show proc '/statistic'` を実行するとデッドロックが発生する可能性があります。 [#34237](https://github.com/StarRocks/starrocks/pull/34237/files)
- 大量のデータがPersistent Indexを有効にしたPrimary Keyテーブルにロードされるとエラーがスローされることがあります。 [#34566](https://github.com/StarRocks/starrocks/pull/34566)
- StarRocksがv2.4またはそれ以前から後のバージョンにアップグレードされると、コンパクションスコアが予期せず上昇する可能性があります。 [#34618](https://github.com/StarRocks/starrocks/pull/34618)
- データベースドライバMariaDB ODBCを使用して`INFORMATION_SCHEMA`がクエリされると、`schemata` ビューで `CATALOG_NAME` 列が `null` の値のみを保持する。 [#34627](https://github.com/StarRocks/starrocks/pull/34627)
- Stream Load jobが**PREPARD**状態で実行されている間にスキーマ変更が実行されていると、ジョブによってロードされるソースデータの一部が失われることがあります。 [#34381](https://github.com/StarRocks/starrocks/pull/34381)
- HDFSストレージパスの末尾に2つ以上のスラッシュ(`/`)を含めると、HDFSからデータのバックアップおよび復元が失敗することがあります。 [#34601](https://github.com/StarRocks/starrocks/pull/34601)
- ローディングタスクまたはクエリを実行すると、FEがハングすることがあります。 [#34569](https://github.com/StarRocks/starrocks/pull/34569)

## 2.5.14

リリース日: 2023年11月14日

### 改善点

- システムデータベース `INFORMATION_SCHEMA` の `COLUMNS` テーブルで、ARRAY、MAP、STRUCTの列を表示できるようにしました。 [#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 互換性の変更

#### システム変数

- DECIMAL型をSTRING型に変換する場合のCBOの挙動を制御するセッション変数 `cbo_decimal_cast_string_strict` を追加しました。 この変数が `true` に設定されている場合、v2.5.xおよびそれ以降のバージョンに組み込まれたロジックが優先され、システムは厳密な変換を実行します（すなわち、システムは生成された文字列を切り捨て、スケール長に基づいて0を埋めます）。 この変数が `false` に設定されている場合、v2.5.xより前のバージョンに組み込まれたロジックが優先され、システムはすべての有効な桁を処理して文字列を生成します。 デフォルト値は `true` です。 [#34208](https://github.com/StarRocks/starrocks/pull/34208)
- DECIMAL型のデータとSTRING型のデータの比較に使用されるデータ型を指定するセッション変数 `cbo_eq_base_type` を追加しました。 デフォルト値は `VARCHAR` であり、DECIMALも有効な値です。 [#34208](https://github.com/StarRocks/starrocks/pull/34208)

### バグ修正

以下の問題を修正しました:

- ON条件がサブクエリとネストされている場合に `java.lang.IllegalStateException: null` エラーが報告されます。 [#30876](https://github.com/StarRocks/starrocks/pull/30876)
- `INSERT INTO SELECT ... LIMIT` の直後にCOUNT(*)が実行されると、レプリカ間でCOUNT(*)の結果が一貫しないことがあります。 [#24435](https://github.com/StarRocks/starrocks/pull/24435)
- キャスト() 関数で指定されたターゲットデータ型が元のデータ型と同じ場合、特定のデータ型の場合にBEがクラッシュすることがあります。 [#31465](https://github.com/StarRocks/starrocks/pull/31465)
- 特定のパス形式が使用されると、Broker Load経由でデータをロードする際に `msg:Fail to parse columnsFromPath, expected: [rec_dt]` エラーが報告されます。 [#32721](https://github.com/StarRocks/starrocks/issues/32721)
- 3.xへのアップグレード中、一部の列型もアップグレードされる場合（たとえば、DecimalがDecimal v3にアップグレードされる場合）、特定の特性を持つテーブルでCompactionが実行されるとBEがクラッシュします。 [#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードする際、高度に並列なロードジョブとHTTPおよびスキャンスレッド数が上限に達した場合、ロードジョブが予期せず中断されることがあります。 [#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlを呼び出すとBEがクラッシュします。 [#31667](https://github.com/StarRocks/starrocks/pull/31667)
- BITMAP列をPrimary Keyテーブルに追加しようとすると、次のエラーが発生します: `Analyze columnDef error: No aggregate function specified for 'userid'`。 [#31763](https://github.com/StarRocks/starrocks/pull/31763)
- Persistent Indexを有効にしたPrimary Keyテーブルに長時間および頻繁にデータをロードすると、BEがクラッシュすることがあります。 [#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効になっていると、クエリの結果が正しくありません。 [#32778](https://github.com/StarRocks/starrocks/pull/32778)
- Primary Keyテーブルを作成する際にNULL可能なSort Keyを指定すると、Compactionが失敗します。 [#29225](https://github.com/StarRocks/starrocks/pull/29225)
- 複雑なJoinクエリの場合に「StarRocks planner use long time 10000 ms in logical phase」エラーが発生することがあります。 [#34177](https://github.com/StarRocks/starrocks/pull/34177)

## 2.5.13

リリース日: 2023年9月28日

### 改善点

- Window関数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP がORDER BY句およびWindow句をサポートするようになりました。 [#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型データのクエリ中に10進オーバーフローが発生した場合、NULLの代わりにエラーが返されます。 [#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドの実行が、MySQLと一貫した結果を返すようになりました。 [#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 削除されたタブレットに対応するローセットがクリーンアップされ、BEの起動時のメモリ使用量が削減されました。 [#30625](https://github.com/StarRocks/starrocks/pull/30625)
```markdown
### バグ修正

次の問題が修正されました：

- スパークコネクタまたはFlinkコネクタを使用してStarRocksからデータを読む際に「MemoryScratchSinkOperatorによってセットがキャンセルされました」というエラーが発生する問題を修正しました。[#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- 集約関数を含むORDER BY句でクエリを実行する際に「java.lang.IllegalStateException: null」というエラーが発生する問題を修正しました。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 非アクティブなマテリアライズドビューがある場合に、FEが再起動に失敗する問題を修正しました。[#30015](https://github.com/StarRocks/starrocks/pull/30015)
- 重複したパーティションにINSERT OVERWRITE操作を行うと、メタデータが破損し、FEの再起動に失敗する問題を修正しました。[#27545](https://github.com/StarRocks/starrocks/pull/27545)
- 主キーのテーブルに存在しない列を変更しようとすると「java.lang.NullPointerException: null」というエラーが発生する問題を修正しました。[#30366](https://github.com/StarRocks/starrocks/pull/30366)
- パーティション化されたStarRocks外部テーブルにデータをロードする際に「TNetworkAddressからTableMetaの取得に失敗しました」というエラーが発生する問題を修正しました。[#30124](https://github.com/StarRocks/starrocks/pull/30124)
- 特定のシナリオで、CloudCanalを介してデータをロードする際にエラーが発生する問題を修正しました。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- Flinkコネクタを介してデータをロードし、DELETEおよびINSERT操作を行う際に「データベースxxxの現在実行中のトランザクションは200で、制限の200よりも大きい」というエラーが発生する問題を修正しました。[#18393](https://github.com/StarRocks/starrocks/pull/18393)
- 集約関数を含むHAVING句を使用する非同期マテリアライズドビューがクエリを適切に書き直せない問題を修正しました。[#29976](https://github.com/StarRocks/starrocks/pull/29976)

## 2.5.12

リリース日：2023年9月4日

### 改善点

- SQL中のコメントを監査ログに保持するようになりました。[#29747](https://github.com/StarRocks/starrocks/pull/29747)
- INSERT INTO SELECTのCPUおよびメモリの統計情報が監査ログに追加されました。[#29901](https://github.com/StarRocks/starrocks/pull/29901)

### バグ修正

次の問題が修正されました：

- ブローカーロードを使用してデータをロードする際に、一部のフィールドのNOT NULL属性がBEのクラッシュや「msg:mismatched row count」エラーの原因となる場合があります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- ORC形式のファイルに対するクエリが失敗する問題を、Apache ORCのバグ修正ORC-1304 ([apache/orc#1299](https://github.com/apache/orc/pull/1299)) がマージされていないために修正しました。[#29804](https://github.com/StarRocks/starrocks/pull/29804)
- プライマリキーテーブルを復元すると、BEの再起動後にメタデータの不整合が発生する問題を修正しました。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 2.5.11

リリース日：2023年8月28日

### 改善点

- WHERE句で複合述語およびすべての式の暗黙の変換をサポートするようになりました。[セッション変数](https://docs.starrocks.io/en-us/3.1/reference/System_variable) `enable_strict_type`を使用して暗黙の変換を有効または無効にできます。デフォルト値は `false` です。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- Icebergカタログを作成する際に `hive.metastore.uri` を指定しないときのエラープロンプトを最適化しました。エラープロンプトが正確になりました。[#16543](https://github.com/StarRocks/starrocks/issues/16543)
- エラーメッセージ `xxx too many versions xxx` に追加のプロンプトを追加しました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- ダイナミックパーティショニングが`year`をパーティショニング単位としてさらにサポートされるようになりました。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

### バグ修正

次の問題が修正されました：

- 複数のレプリカを持つテーブルにデータをロードすると、テーブルの一部のパーティションが空の場合、無効なログレコードが大量に書き込まれる問題を修正しました。[#28824](https://github.com/StarRocks/starrocks/issues/28824)
- WHERE条件で使われるフィールドがBITMAPまたはHLLフィールドの場合、DELETE操作が失敗する問題を修正しました。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 非同期マテリアライズドビューを手動でリフレッシュすると、同期呼び出し（SYNC MODE）により`information_schema.task_runs` テーブルに複数のINSERT OVERWRITEレコードが結果として記録される問題を修正しました。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- エラーステートにあるタブレットに対してCLONE操作がトリガーされると、ディスク使用量が増加する問題を修正しました。[#28488](https://github.com/StarRocks/starrocks/pull/28488)
- Join Reorderが有効になっている場合、クエリの結果がクエリ対象の列が定数である場合に正しくない問題を修正しました。[#29239](https://github.com/StarRocks/starrocks/pull/29239)
- SSDとHDD間でのタブレットの移行中に、FEが過剰な移行タスクをBEに送信すると、BEでOOMの問題が発生する問題を修正しました。[#29055](https://github.com/StarRocks/starrocks/pull/29055)
- `/apache_hdfs_broker/lib/log4j-1.2.17.jar` のセキュリティ脆弱性を修正しました。[#28866](https://github.com/StarRocks/starrocks/pull/28866)
- Hiveカタログを介したデータクエリ中に、パーティショニング列とOR演算子がWHERE句で使用されると、クエリ結果が正しくない問題を修正しました。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- データクエリ中に「java.util.ConcurrentModificationException: null」というエラーが一部で発生する問題を修正しました。[#29296](https://github.com/StarRocks/starrocks/pull/29296)
- 非同期マテリアライズドビューのベーステーブルが削除された場合、FEを再起動できない問題を修正しました。[#29318](https://github.com/StarRocks/starrocks/pull/29318)
- データを書き込んでいる間に非同期マテリアライズドビューのベーステーブルを跨いだ非同期マテリアライズドビューがデッドロックに遭遇することがある問題を修正しました。[#29432](https://github.com/StarRocks/starrocks/pull/29432)

## 2.5.10

リリース日：2023年8月7日

### 新機能

- 集約関数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md) をサポートするようになりました。
- 以下の[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)をサポートするようになりました：COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP。

### 改善点

- TabletCheckerのスケジューリングロジックを最適化し、修復されないタブレットを繰り返しスケジューリングするのを防ぎました。[#27648](https://github.com/StarRocks/starrocks/pull/27648)
- スキーマ変更とルーチンロードが同時に発生した場合、スキーマ変更が先に完了した場合にルーチンロードのジョブが失敗することがある問題が最適化されたエラーメッセージを報告するようになりました。[#28425](https://github.com/StarRocks/starrocks/pull/28425)
- 外部テーブルを作成する際にNOT NULL列を定義することを禁止しました（NOT NULL列が定義されていると、アップグレード後にエラーが発生し、テーブルをもう一度作成する必要があります）。外部カタログは、外部テーブルの代わりにv2.3.0から推奨されます。[#25485](https://github.com/StarRocks/starrocks/pull/25441)
- Broker Loadのリトライでエラーが発生した場合にエラーメッセージを追加しました。これによりデータのロード中のトラブルシューティングとデバッグが容易になります。[#21982](https://github.com/StarRocks/starrocks/pull/21982)
- UPSERTおよびDELETE操作を含むロードジョブの大規模なデータ書き込みをサポートするようになりました。[#17264](https://github.com/StarRocks/starrocks/pull/17264)
- マテリアライズドビューを使用したクエリのリライトを最適化しました。[#27934](https://github.com/StarRocks/starrocks/pull/27934) [#25542](https://github.com/StarRocks/starrocks/pull/25542) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#27557](https://github.com/StarRocks/starrocks/pull/27557) [#22300](https://github.com/StarRocks/starrocks/pull/22300) [#26957](https://github.com/StarRocks/starrocks/pull/26957) [#27728](https://github.com/StarRocks/starrocks/pull/27728) [#27900](https://github.com/StarRocks/starrocks/pull/27900)

### バグ修正

次の問題が修正されました：
```
- CASTを使用して文字列を配列に変換すると、入力に定数が含まれる場合、結果が正しくないことがあります。 [#19793](https://github.com/StarRocks/starrocks/pull/19793)
- SHOW TABLETはORDER BYとLIMITを含む場合、間違った結果を返します。 [#23375](https://github.com/StarRocks/starrocks/pull/23375)
- 外部結合とマテリアライズド・ビューの反結合の再書き込みエラー。 [#28028](https://github.com/StarRocks/starrocks/pull/28028)
- FEにおける不正確なテーブルレベルのスキャン統計は、テーブルクエリとローディングのメトリクスに誤差をもたらします。 [#27779](https://github.com/StarRocks/starrocks/pull/27779)
- 現在の長いリンクを使用してメタストアにアクセスしようとしたときに例外が発生しました。msg: 最後のイベントIDに基づいて次の通知を取得できませんでした：707602。これは、イベントリスナーがHMSで構成され、Hiveメタデータを増分更新するためにイベントリスナーが構成されている場合、FEログに報告されます。 [#21056](https://github.com/StarRocks/starrocks/pull/21056)
- ソートキーがパーティションテーブルの場合、クエリ結果が安定しないことがあります。 [#27850](https://github.com/StarRocks/starrocks/pull/27850)
- Spark Loadを使用してロードしたデータは、バケット化された列がDATE、DATETIME、またはDECIMAL列の場合、誤ったバケットに分散されることがあります。 [#27005](https://github.com/StarRocks/starrocks/pull/27005)
- regex_replace関数は、特定のシナリオでBEがクラッシュする場合があります。 [#27117](https://github.com/StarRocks/starrocks/pull/27117)
- sub_bitmap関数の入力がBITMAP値でない場合、BEがクラッシュします。 [#27982](https://github.com/StarRocks/starrocks/pull/27982)
- Join Reorderが有効になっている場合、「Unknown error」がクエリのために返されます。 [#27472](https://github.com/StarRocks/starrocks/pull/27472)
- 平均行サイズの不正確な推定により、主キー部分的な更新は過剰に大きなメモリを占有します。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 低カーディナリティ最適化が有効になっている場合、一部のINSERTジョブが「[42000][1064] Dict Decode failed, Dict can't take cover all key :0」と返されることがあります。 [#26463](https://github.com/StarRocks/starrocks/pull/26463)
- ユーザーがHDFSからデータをロードするために作成したBroker Loadジョブで、「"hadoop.security.authentication" = "simple"」を指定すると、ジョブが失敗します。 [#27774](https://github.com/StarRocks/starrocks/pull/27774)
- マテリアライズド・ビューのリフレッシュモードを変更すると、リーダーFEとフォロワーFE間でメタデータが一貫しなくなります。 [#28082](https://github.com/StarRocks/starrocks/pull/28082) [#28097](https://github.com/StarRocks/starrocks/pull/28097)
- 指定された場合、「hadoop.security.authentication」 = 「simple」を使用して作成されたBroker Loadジョブによって、特定の情報をクエリすると、パスワードが非表示になりません。 [#28059](https://github.com/StarRocks/starrocks/pull/28059)
- ブロックされたLabelCleanerスレッドによるFEメモリリーク。 [#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 2.5.9

リリース日：2023年7月19日

### 新機能

- マテリアライズド・ビューとは異なるタイプの結合を含むクエリを再書き込むことができます。 [#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改善点

- StarRocksの外部テーブルは、現在のStarRocksクラスターの宛先クラスターで作成できません。 [#25441](https://github.com/StarRocks/starrocks/pull/25441)
- クエリされるフィールドがマテリアライズド・ビューの出力列に含まれていないが、マテリアライズド・ビューの述語に含まれている場合、クエリは再書き込むことができます。 [#23028](https://github.com/StarRocks/starrocks/issues/23028)
- データベース`Information_schema`のテーブル`tables_config`に新しいフィールド`table_id`を追加しました。この`table_id`列を使用して`tables_config`を`be_tablets`と結合し、タブレットが属するデータベースとテーブルの名前をクエリできます。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### バグ修正

以下の問題が修正されました。

- 重複キーのテーブルに対してCount Distinctの結果が正しくありません。 [#24222](https://github.com/StarRocks/starrocks/pull/24222)
- 結合キーが大きなBINARY列の場合、BEがクラッシュする可能性があります。 [#25084](https://github.com/StarRocks/starrocks/pull/25084)
- INSERT操作がハングする場合があります。これは、挿入対象とするSTRUCT列のCHARデータの長さがSTRUCT列で定義されている最大CHAR長を超える場合に発生します。 [#25942](https://github.com/StarRocks/starrocks/pull/25942)
- coalesce()の結果が正しくありません。 [#26250](https://github.com/StarRocks/starrocks/pull/26250)
- データを復元した後、テーブルのタブレットのバージョン番号がBEとFEで一貫していません。 [#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- 復旧されたテーブルのパーティションは自動的に作成されません。 [#26813](https://github.com/StarRocks/starrocks/pull/26813)

## 2.5.8

リリース日：2023年6月30日

### 改善点

- パーティションが非パーティションテーブルに追加されたときに報告されるエラーメッセージが最適化されました。 [#25266](https://github.com/StarRocks/starrocks/pull/25266)
- テーブルのデフォルトのコメントが最適化されました。 [#24803](https://github.com/StarRocks/starrocks/pull/24803)
- 非同期マテリアライズド・ビューの手動リフレッシュが最適化されました。REFRESH MATERIALIZED VIEW WITH SYNC MODE構文を使用してマテリアライズド・ビューのリフレッシュタスクを同期的に呼び出すことがサポートされます。 [#25910](https://github.com/StarRocks/starrocks/pull/25910)

### バグ修正

以下の問題が修正されました。

- 非同期マテリアライズド・ビューのCOUNT結果が、マテリアライズド・ビューがUnion結果に基づいて構築された場合、正確でない場合があります。 [#24460](https://github.com/StarRocks/starrocks/issues/24460)
- ルートパスワードを強制的にリセットしようとしたときに、"Unknown error"が報告されます。 [#25492](https://github.com/StarRocks/starrocks/pull/25492)
- INSERT OVERWRITEが3つ未満のアクティブBEを持つクラスターで実行されると、不正確なエラーメッセージが表示されます。 [#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 2.5.7

リリース日：2023年6月14日

### 新機能

- 非アクティブなマテリアライズド・ビューは`ALTER MATERIALIZED VIEW <mv_name> ACTIVE`を使用して手動でアクティブ化できます。このSQLコマンドを使用して、基本テーブルが削除され、それを再作成するマテリアライズド・ビューをアクティブ化できます。詳細については、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を参照してください。 [#24001](https://github.com/StarRocks/starrocks/pull/24001)
- StarRocksは、テーブルを作成したり、パーティションを追加したりする際に、自動的に適切な数のタブレットを設定できるようになりました。詳細については、「タブレットの数を決定する」を参照してください。[#10614](https://github.com/StarRocks/starrocks/pull/10614)

### 改善点

- 外部テーブルクエリで使用されるScanノードのI/O並列処理が最適化されました。これにより、メモリ使用量が削減され、外部テーブルからのデータの安定性が向上します。 [#23617](https://github.com/StarRocks/starrocks/pull/23617) [#23624](https://github.com/StarRocks/starrocks/pull/23624) [#23626](https://github.com/StarRocks/starrocks/pull/23626)
- Broker Loadジョブのエラーメッセージが最適化されました。エラーメッセージには、再試行情報とエラーの発生ファイル名が含まれます。 [#18038](https://github.com/StarRocks/starrocks/pull/18038) [#21982](https://github.com/StarRocks/starrocks/pull/21982)
- CREATE TABLEのタイムアウト時に返されるエラーメッセージが最適化され、パラメータチューニングのヒントが追加されました。 [#24510](https://github.com/StarRocks/starrocks/pull/24510)
- ALTER TABLEが失敗すると、テーブルの状態が正常でないため、返されるエラーメッセージが最適化されました。 [#24381](https://github.com/StarRocks/starrocks/pull/24381)
- CREATE TABLEステートメントの全角スペースが無視されるようになりました。 [#23885](https://github.com/StarRocks/starrocks/pull/23885)
- ブローカーアクセスタイムアウトを最適化し、ブローカーロードジョブの成功率を向上させました。[#22699](https://github.com/StarRocks/starrocks/pull/22699)
- プライマリキーテーブルの場合、SHOW TABLETによって返される `VersionCount` フィールドにはペンディング状態のRowsetsが含まれています。[#23847](https://github.com/StarRocks/starrocks/pull/23847)
- パーシステントインデックスポリシーを最適化しました。[#22140](https://github.com/StarRocks/starrocks/pull/22140)

### バグ修正

以下の問題を修正しました：

- ユーザーがParquetデータをStarRocksにロードする際、タイプ変換中にDATETIME値がオーバーフローしてデータエラーが発生する問題を修正しました。[#22356](https://github.com/StarRocks/starrocks/pull/22356)
- ダイナミックパーティショニングを無効にした後、バケット情報が失われる問題を修正しました。[#22595](https://github.com/StarRocks/starrocks/pull/22595)
- CREATE TABLEステートメントでサポートされていないプロパティを使用すると、nullポインタ例外（NPE）が発生する問題を修正しました。[#23859](https://github.com/StarRocks/starrocks/pull/23859)
- `information_schema` のテーブル権限フィルタリングが無効になり、ユーザーが権限を持たないテーブルを表示できるという問題を修正しました。[#23804](https://github.com/StarRocks/starrocks/pull/23804)
- SHOW TABLE STATUSで返される情報が不十分な問題を修正しました。[#24279](https://github.com/StarRocks/starrocks/issues/24279)
- スキーマ変更とデータロードが同時に発生すると、スキーマ変更が一部の場合に中断する問題を修正しました。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- RocksDB WALフラッシュが、高頻度のプライマリキーテーブルへのデータロードを中断するbrpcワーカーの処理をブロックしてしまう問題を修正しました。[#22489](https://github.com/StarRocks/starrocks/pull/22489)
- StarRocksでサポートされていないTIME型の列が正常に作成される問題を修正しました。[#23474](https://github.com/StarRocks/starrocks/pull/23474)
- マテリアライズドビューのUnionリライトが失敗する問題を修正しました。[#22922](https://github.com/StarRocks/starrocks/pull/22922)

## 2.5.6

リリース日：2023年5月19日

### 改善

- `INSERT INTO ... SELECT` が `thrift_server_max_worker_thread` の値が小さいために期限切れになったときに報告されるエラーメッセージを最適化しました。[#21964](https://github.com/StarRocks/starrocks/pull/21964)
- CTASを使用して作成されたテーブルは、通常のテーブルのデフォルトのレプリカ数と同様に、デフォルトで3つのレプリカを持っています。[#22854](https://github.com/StarRocks/starrocks/pull/22854)

### バグ修正

- TRUNCATE操作がパーティション名の大文字と小文字を区別するため、パーティションの切り捨てに失敗する問題を修正しました。[#21809](https://github.com/StarRocks/starrocks/pull/21809)
- マテリアライズドビューの一時的なパーティション作成に失敗したため、BEの廃止の処理が失敗する問題を修正しました。[#22745](https://github.com/StarRocks/starrocks/pull/22745)
- ARRAY値が必要な動的FEパラメータに空の配列を設定できない問題を修正しました。[#22225](https://github.com/StarRocks/starrocks/pull/22225)
- `partition_refresh_number` プロパティが指定されたマテリアライズドビューが完全に更新されない場合がある問題を修正しました。[#21619](https://github.com/StarRocks/starrocks/pull/21619)
- SHOW CREATE TABLEでクラウド資格情報がマスクされ、メモリ内の誤った資格情報を引き起こす問題を修正しました。[#21311](https://github.com/StarRocks/starrocks/pull/21311)
- 外部テーブルを介してクエリされる一部のORCファイルに対して述語が効果を発揮しない問題を修正しました。[#21901](https://github.com/StarRocks/starrocks/pull/21901)
- カラム名に大文字と小文字が混在する場合、min-maxフィルターが適切に処理できない問題を修正しました。[#22626](https://github.com/StarRocks/starrocks/pull/22626)
- 遅延マテリアライズにより、複雑なデータ型（STRUCTまたはMAP）をクエリする際にエラーが発生する問題を修正しました。[#22862](https://github.com/StarRocks/starrocks/pull/22862)
- プライマリキーテーブルを復元する際に発生する問題を修正しました。[#23384](https://github.com/StarRocks/starrocks/pull/23384)

## 2.5.5

リリース日：2023年4月28日

### 新機能

プライマリキーテーブルのタブレットステータスを監視するメトリックを追加しました：

- FEメトリック `err_state_metric` を追加しました。
- `SHOW PROC '/statistic/'` の出力に `ErrorStateTabletNum` 列を追加して **err_state** タブレットの数を表示します。
- `SHOW PROC '/statistic/<db_id>/'` の出力に `ErrorStateTablets` 列を追加して **err_state** タブレットのIDを表示します。

詳細については、[SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md) を参照してください。

### 改善

- 複数のBEが追加されたときのディスクバランシング速度を最適化しました。[#19418](https://github.com/StarRocks/starrocks/pull/19418)
- BEがSSDとHDDの両方を使用する場合、`storage_cooldown_time` プロパティが指定されている場合、StarRocksは `storage_medium` を `SSD` に設定します。それ以外の場合、StarRocksは `storage_medium` を `HDD` に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- Unique Keyテーブルのパフォーマンスを最適化し、値列からの統計情報の収集を禁止しました。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

- コロケーションテーブルの場合、「ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");」などのステートメントを使用してレプリカステータスを手動で「bad」に指定できます。BEの数がレプリカの数以下の場合、破損したレプリカを修復できません。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- BEが起動した後、プロセスは存在しますがBEポートが有効にならない問題を修正しました。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- サブクエリがウィンドウ関数とネストされている集計クエリの誤った結果を返す問題を修正しました。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- 最初のマテリアライズドビュー（MV）の更新時に `auto_refresh_partitions_limit` が効果を発揮しないため、すべてのパーティションが更新される問題を修正しました。[#19759](https://github.com/StarRocks/starrocks/issues/19759)
- CSV Hive外部テーブルのクエリで、MAPやSTRUCTなどの複雑なデータでネストされた配列データを使用するとエラーが発生する問題を修正しました。[#20233](https://github.com/StarRocks/starrocks/pull/20233)
- Sparkコネクタを使用するクエリでタイムアウトが発生する問題を修正しました。[#20264](https://github.com/StarRocks/starrocks/pull/20264)
- 2つのレプリカテーブルの1つのレプリカが破損している場合、テーブルを回復できない問題を修正しました。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- MVクエリのリライトの失敗によるクエリの失敗を修正しました。[#19549](https://github.com/StarRocks/starrocks/issues/19549)
- データベースロックにより、メトリックインタフェースが期限切れになる問題を修正しました。[#20790](https://github.com/StarRocks/starrocks/pull/20790)
- ブロードキャスト結合に誤った結果が返される問題を修正しました。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- CREATE TABLEでサポートされていないデータ型が使用された場合にNPEが返される問題を修正しました。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- クエリキャッシュ機能でwindow_funnel()を使用した場合に発生する問題を修正しました。[#21474](https://github.com/StarRocks/starrocks/issues/21474)
- CTEのリライト後に最適化プランの選択に予期しないほどの時間がかかる問題を修正しました。[#16515](https://github.com/StarRocks/starrocks/pull/16515)

## 2.5.4

リリース日：2023年4月4日

### 改善

- クエリプランニング中のマテリアライズドビュー上のクエリのリライトのパフォーマンスを最適化し、クエリプランニングにかかる時間を約70%削減しました。[#19579](https://github.com/StarRocks/starrocks/pull/19579)
- タイプ推論ロジックを最適化しました。`SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;` のような `SELECT sum(CASE WHEN XXX);` といったクエリに、`SELECT sum(CASE WHEN k1 = 1 THEN v1 ELSE 0 END) FROM test;` のような定数 `0` が含まれていると、自動的に事前集約が有効になり、クエリが加速されます。[#19474](https://github.com/StarRocks/starrocks/pull/19474)
- `SHOW CREATE VIEW` を使用してマテリアライズドビューの作成ステートメントを表示することをサポートしました。[#19999](https://github.com/StarRocks/starrocks/pull/19999)
- 単一のbRPCリクエストでBEノード間で2GB以上のパケットを転送することをサポートします。[#20283](https://github.com/StarRocks/starrocks/pull/20283) [#20230](https://github.com/StarRocks/starrocks/pull/20230)
- [SHOW CREATE CATALOG](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) を使用して外部カタログの作成ステートメントをクエリすることをサポートします。

### バグ修正

以下のバグが修正されました：

- マテリアライズドビューのクエリを再構築した後、低カーディナリティ最適化のためのグローバル辞書が効果を発揮しません。[#19615](https://github.com/StarRocks/starrocks/pull/19615)
- マテリアライズドビューのクエリが再構築に失敗すると、クエリが失敗します。[#19774](https://github.com/StarRocks/starrocks/pull/19774)
- マテリアライズドビューがプライマリキーまたはユニークキーのテーブルをベースに作成された場合、そのマテリアライズドビューに対するクエリは再構築できません。[#19600](https://github.com/StarRocks/starrocks/pull/19600)
- マテリアライズドビューの列名は大文字と小文字を区別しますが、テーブルの`PROPERTIES`で列名が間違っていてもテーブルはエラーメッセージなしで正常に作成され、さらにそのテーブルに作成されたマテリアライズドビューのクエリが再構築に失敗します。[#19780](https://github.com/StarRocks/starrocks/pull/19780)
- マテリアライズドビューのクエリが再構築された後、クエリプランにはパーティションベースの列や無効な述語が含まれる可能性があり、クエリのパフォーマンスに影響を与えます。[#19784](https://github.com/StarRocks/starrocks/pull/19784)
- 新しく作成されたパーティションにデータがロードされると、マテリアライズドビューのクエリの再構築が失敗する可能性があります。[#20323](https://github.com/StarRocks/starrocks/pull/20323)
- マテリアライズドビューの作成時に `"storage_medium" = "SSD"` を設定すると、マテリアライズドビューのリフレッシュに失敗します。[#19539](https://github.com/StarRocks/starrocks/pull/19539) [#19626](https://github.com/StarRocks/starrocks/pull/19626)
- プライマリキーテーブルで同時コンパクションが発生する可能性があります。[#19692](https://github.com/StarRocks/starrocks/pull/19692)
- 大量のDELETE操作の後にコンパクションがすぐに発生しないことがあります。[#19623](https://github.com/StarRocks/starrocks/pull/19623)
- ステートメントの式に複数の低カーディナリティ列が含まれている場合、式を適切に再構築できない可能性があります。その結果、低カーディナリティ最適化のためのグローバル辞書が効果を発揮しません。[#20161](https://github.com/StarRocks/starrocks/pull/20161)

## 2.5.3

リリース日: 2023年3月10日

### 改善

- マテリアライズドビュー（MVs）のクエリ再構築を最適化しました。
  - アウタージョインとクロスジョインを使用してクエリを再構築することをサポートします。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - MVのデータスキャンロジックを最適化し、再構築されたクエリの処理をさらに高速化しました。[#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - 単一テーブルの集計クエリの再構築機能を強化しました。 [#18629](https://github.com/StarRocks/starrocks/pull/18629)
  - クエリされたテーブルがMVの基本テーブルのサブセットである場合のビューデルタシナリオでの再構築機能を強化しました。[#18800](https://github.com/StarRocks/starrocks/pull/18800)
- ウィンドウ関数RANK()がフィルターまたはソートキーとして使用された場合のパフォーマンスとメモリ使用を最適化しました。[#17553](https://github.com/StarRocks/starrocks/issues/17553)

### バグ修正

以下のバグが修正されました：

- ARRAYデータ内のnullリテラル`[]`によって引き起こされるエラーが修正されました。[#18563](https://github.com/StarRocks/starrocks/pull/18563)
- 一部の複雑なクエリシナリオで低カーディナリティ最適化辞書の誤用が発生していた問題が修正されました。辞書マッピングチェックが辞書を適用する前に追加されました。[#17318](https://github.com/StarRocks/starrocks/pull/17318)
- 単一のBE環境で、Local ShuffleによってGROUP BYが重複した結果を生成する問題が修正されました。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- パーティション関連のPROPERTIESの誤用により非パーティションMVのリフレッシュが失敗する可能性があります。ユーザーがMVを作成する際にパーティションPROPERTIESのチェックが実行されるようになりました。[#18741](https://github.com/StarRocks/starrocks/pull/18741)
- ParquetのRepetition列の解析エラーが修正されました。[#17626](https://github.com/StarRocks/starrocks/pull/17626) [#17788](https://github.com/StarRocks/starrocks/pull/17788) [#18051](https://github.com/StarRocks/starrocks/pull/18051)
- 取得した列のnull可能情報が正しくない問題が修正されました。解決策: CTASを使用してプライマリキーテーブルを作成すると、プライマリキーカラムのみがnullでない項目となり、それ以外のカラムはnull可能となります。[#16431](https://github.com/StarRocks/starrocks/pull/16431)
- プライマリキーテーブルからデータを削除することによって引き起こされる一部の問題が修正されました。[#18768](https://github.com/StarRocks/starrocks/pull/18768)

## 2.5.2

リリース日: 2023年2月21日

### 新機能

- インスタンスプロファイルおよびアサムドロールベースの資格情報メソッドを使用してAWS S3およびAWS Glueにアクセスすることをサポートします。[#15958](https://github.com/StarRocks/starrocks/pull/15958)
- bit_shift_left, bit_shift_right, bit_shift_right_logicalといったビット関数をサポートします。[#14151](https://github.com/StarRocks/starrocks/pull/14151)

### 改善

- 大量の集計クエリを含むクエリでのピークメモリ使用量を劇的に減少させるためのメモリリリースロジックを最適化しました。[#16913](https://github.com/StarRocks/starrocks/pull/16913)
- ソートのメモリ使用量を削減しました。ウィンドウ関数やソートが含まれるクエリではメモリ消費が半分になりました。[#16937](https://github.com/StarRocks/starrocks/pull/16937) [#17362](https://github.com/StarRocks/starrocks/pull/17362) [#17408](https://github.com/StarRocks/starrocks/pull/17408)

### バグ修正

以下のバグが修正されました：

- MAPとARRAYデータを含むApache Hive外部テーブルをリフレッシュできない問題が解決されました。[#17548](https://github.com/StarRocks/starrocks/pull/17548)
- Supersetがマテリアライズドビューの列タイプを識別できない問題が解決されました。[#17686](https://github.com/StarRocks/starrocks/pull/17686)
- SET GLOBAL/SESSION TRANSACTIONをパースできないため、BIの接続が失敗する問題が解決されました。[#17295](https://github.com/StarRocks/starrocks/pull/17295)
- Colocate Group内のダイナミックパーティションテーブルのバケット数を変更できず、エラーメッセージが返される問題が解決されました。[#17418](https://github.com/StarRocks/starrocks/pull/17418/)
- 準備段階の失敗によって引き起こされる潜在的な問題が解決されました。[#17323](https://github.com/StarRocks/starrocks/pull/17323)

### 動作の変更

- `enable_experimental_mv`のデフォルト値を`false`から`true`に変更しました。つまり、非同期マテリアライズドビューがデフォルトで有効になりました。
- 予約キーワードリストにCHARACTERを追加しました。[#17488](https://github.com/StarRocks/starrocks/pull/17488)

## 2.5.1

リリース日: 2023年2月5日

### 改善

- 外部カタログに基づいて作成された非同期マテリアライズドビューはクエリ再構築をサポートします。 [#11116](https://github.com/StarRocks/starrocks/issues/11116) [#15791](https://github.com/StarRocks/starrocks/issues/15791)
- クラスターパフォーマンスの揺らぎを防ぐため、自動的な全体の収集によって引き起こされるクラスターパフォーマンスの揺らぎを防ぐために、自動的なCBO統計収集のためのコレクション期間をユーザーが指定できるようにしました。[#14996](https://github.com/StarRocks/starrocks/pull/14996)
- Thriftサーバーキューを追加しました。INSERT INTO SELECT中に即座に処理できないリクエストはThriftサーバーキューに保留することができ、リクエストが拒否されるのを防ぎます。[#14571](https://github.com/StarRocks/starrocks/pull/14571)
- FEパラメータ`default_storage_medium`を非推奨としました。ユーザーがテーブルを作成する際に`storage_medium`が明示的に指定されていない場合、システムはBEディスクタイプに基づいてテーブルのストレージメディアを自動的に推測します。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_VIEW.md)の`storage_medium`の説明を参照してください。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

### バグ修正

以下のバグが修正されました:
- `SET PASSWORD`によって発生するヌルポインタ例外（NPE）です。 [#15247](https://github.com/StarRocks/starrocks/pull/15247)
- 空のキーを持つJSONデータを解析できません。[#16852](https://github.com/StarRocks/starrocks/pull/16852)
- 無効なタイプのデータを正常に配列データに変換できます。[#16866](https://github.com/StarRocks/starrocks/pull/16866)
- 例外が発生した場合にネストされたループ結合を中断できない場合があります。 [#16875](https://github.com/StarRocks/starrocks/pull/16875)

### 挙動の変更

- FEパラメータ `default_storage_medium` を非推奨としました。テーブルのストレージメディアは自動的にシステムによって推測されます。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.5.0

リリース日: 2023年1月22日

### 新機能

- [Hudiカタログ](../data_source/catalog/hudi_catalog.md)および[Hudi外部テーブル](../data_source/External_table.md#deprecated-hudi-external-table)を使用してMerge On Readテーブルのクエリをサポートします。[#6780](https://github.com/StarRocks/starrocks/pull/6780)
- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、Hudiカタログ、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用してSTRUCTおよびMAPデータのクエリをサポートします。[#10677](https://github.com/StarRocks/starrocks/issues/10677)
- [外部ストレージシステム（HDFSなど）に格納されたホットデータへのアクセス性能を向上させるための[データキャッシュ](../data_source/data_cache.md)を提供します。[#11597](https://github.com/StarRocks/starrocks/pull/11579)
- Delta Lakeからのデータに対する直接クエリを可能にする[Delta Lakeカタログ](../data_source/catalog/deltalake_catalog.md)を作成できます。[#11972](https://github.com/StarRocks/starrocks/issues/11972)
- Hive、Hudi、およびIcebergカタログはAWS Glueと互換性があります。[#12249](https://github.com/StarRocks/starrocks/issues/12249)
- ParquetおよびORCファイルに直接クエリを実行する[ファイル外部テーブル](../data_source/file_external_table.md)を作成できます。[#13064](https://github.com/StarRocks/starrocks/pull/13064)
- Hive、Hudi、Icebergカタログ、およびマテリアライズドビューを基にしたマテリアライズドビューを作成できます。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。[#11116](https://github.com/StarRocks/starrocks/issues/11116) [#11873](https://github.com/StarRocks/starrocks/pull/11873)
- プライマリキーテーブルを使用するテーブルの条件付き更新をサポートします。詳細については、[プライマリキーテーブルへのデータの反映](../loading/Load_to_Primary_Key_tables.md)を参照してください。[#12159](https://github.com/StarRocks/starrocks/pull/12159)
- [クエリキャッシュ](../using_starrocks/query_cache.md)をサポートします。これにより、クエリの中間計算結果を保存し、高い同時実行数のシンプルなクエリのQPSを向上させ、平均遅延を減らすことができます。[#9194](https://github.com/StarRocks/starrocks/pull/9194)
- Broker Loadジョブの優先度を指定できます。詳細については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。[#11029](https://github.com/StarRocks/starrocks/pull/11029)
- StarRocksネイティブテーブルのデータロードのためのレプリカ数を指定できます。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11253](https://github.com/StarRocks/starrocks/pull/11253)
- [クエリキュー](../administration/query_queues.md)をサポートします。[#12594](https://github.com/StarRocks/starrocks/pull/12594)
- データロードのリソース消費を制限することで、データロードによって占有されるコンピュートリソースを分離することができます。詳細については、[リソースグループ](../administration/resource_group.md)を参照してください。[#12606](https://github.com/StarRocks/starrocks/pull/12606)
- StarRocksネイティブテーブルのデータ圧縮アルゴリズムとして、LZ4、Zstd、Snappy、およびZlibを指定できます。詳細については、[データ圧縮](../table_design/data_compression.md)を参照してください。[#10097](https://github.com/StarRocks/starrocks/pull/10097) [#12020](https://github.com/StarRocks/starrocks/pull/12020)
- [ユーザー定義変数](../reference/user_defined_variables.md)をサポートします。[#10011](https://github.com/StarRocks/starrocks/pull/10011)
- [lambda式](../sql-reference/sql-functions/Lambda_expression.md)および以下のハイアーオーダー関数:[array_map](../sql-reference/sql-functions/array-functions/array_map.md),[array_sum](../sql-reference/sql-functions/array-functions/array_sum.md),[array_sortby](../sql-reference/sql-functions/array-functions/array_sortby.md)をサポートします。[#9461](https://github.com/StarRocks/starrocks/pull/9461) [#9806](https://github.com/StarRocks/starrocks/pull/9806) [#10323](https://github.com/StarRocks/starrocks/pull/10323) [#14034](https://github.com/StarRocks/starrocks/pull/14034)
- [ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)の結果をフィルタリングする`QUALIFY`句を提供します。[#13239](https://github.com/StarRocks/starrocks/pull/13239)
- テーブルを作成する際、列のデフォルト値として`uuid()`および`uuid_numeric()`関数の結果を使用することができます。詳細については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。[#11155](https://github.com/StarRocks/starrocks/pull/11155)
- 次の関数をサポートします:[map_size](../sql-reference/sql-functions/map-functions/map_size.md),[map_keys](../sql-reference/sql-functions/map-functions/map_keys.md),[map_values](../sql-reference/sql-functions/map-functions/map_values.md),[max_by](../sql-reference/sql-functions/aggregate-functions/max_by.md),[sub_bitmap](../sql-reference/sql-functions/bitmap-functions/sub_bitmap.md),[bitmap_to_base64](../sql-reference/sql-functions/bitmap-functions/bitmap_to_base64.md),[host_name](../sql-reference/sql-functions/utility-functions/host_name.md),および[date_slice](../sql-reference/sql-functions/date-time-functions/date_slice.md)。[#11299](https://github.com/StarRocks/starrocks/pull/11299) [#11323](https://github.com/StarRocks/starrocks/pull/11323) [#12243](https://github.com/StarRocks/starrocks/pull/12243) [#11776](https://github.com/StarRocks/starrocks/pull/11776) [#12634](https://github.com/StarRocks/starrocks/pull/12634) [#14225](https://github.com/StarRocks/starrocks/pull/14225)

### 改善

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)、[Hudiカタログ](../data_source/catalog/hudi_catalog.md)、および[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)を使用して外部データをクエリする際のメタデータアクセス性能を最適化しました。[#11349](https://github.com/StarRocks/starrocks/issues/11349)
- [Elasticsearch外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-external-table)を使用して配列データのクエリをサポートします。[#9693](https://github.com/StarRocks/starrocks/pull/9693)
- マテリアライズドビューの以下の点を最適化しました:
  - 非同期マテリアライズドビューはSPJGタイプのマテリアライズドビューに基づいた自動および透過的なクエリの書き換えをサポートします。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md#rewrite-and-accelerate-queries-with-the-asynchronous-materialized-view)を参照してください。[#13193](https://github.com/StarRocks/starrocks/issues/13193)
  - 非同期マテリアライズドビューは複数の非同期リフレッシュメカニズムをサポートします。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md#manually-refresh-an-asynchronous-materialized-view)を参照してください。[#12712](https://github.com/StarRocks/starrocks/pull/12712) [#13171](https://github.com/StarRocks/starrocks/pull/13171) [#13229](https://github.com/StarRocks/starrocks/pull/13229) [#12926](https://github.com/StarRocks/starrocks/pull/12926)
  - マテリアライズドビューのリフレッシュの効率が向上しました。[#13167](https://github.com/StarRocks/starrocks/issues/13167)
- データロードの以下の点を最適化しました:
  - "single leader replication" mode support added, improving loading performance in multi-replica scenarios. Data loading performance has been increased by 100%. For more information about "single leader replication", refer to `replicated_storage` in [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md). [#10138](https://github.com/StarRocks/starrocks/pull/10138)
  - Broker Load and Spark Load no longer require dependency on brokers for data loading when only one HDFS cluster or one Kerberos user is configured. However, if you have multiple HDFS clusters or multiple Kerberos users, a broker is still required. For more information, refer to [Load data from HDFS or cloud storage](../loading/BrokerLoad.md) and [Bulk load using Apache Spark™](../loading/SparkLoad.md). [#9049](https://github.com/starrocks/starrocks/pull/9049) [#9228](https://github.com/StarRocks/starrocks/pull/9228)
  - Optimized the performance of Broker Load when loading a large number of small ORC files. [#11380](https://github.com/StarRocks/starrocks/pull/11380)
  - Reduced memory usage when loading data into Primary Key tables.
- Optimized the `information_schema` database, the `tables`, and `columns` tables. Added a new table `table_config`. For more information, refer to [Information Schema](../reference/overview-pages/information_schema.md). [#10033](https://github.com/StarRocks/starrocks/pull/10033)
- Optimized data backup and restore:
  - Support for backing up and restoring data from multiple tables in a database simultaneously. For more information, refer to [Backup and restore data](../administration/Backup_and_restore.md). [#11619](https://github.com/StarRocks/starrocks/issues/11619)
  - Support for backing up and restoring data from Primary Key tables. For more information, refer to Backup and restore. [#11885](https://github.com/StarRocks/starrocks/pull/11885)
- Optimized the following functions:
  - Added an optional parameter for the [time_slice](../sql-reference/sql-functions/date-time-functions/time_slice.md) function to determine whether the beginning or end of the time interval is returned. [#11216](https://github.com/StarRocks/starrocks/pull/11216)
  - Added a new mode `INCREASE` for the [window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md) function to avoid computing duplicate timestamps. [#10134](https://github.com/StarRocks/starrocks/pull/10134)
  - Support for specifying multiple arguments in the [unnest](../sql-reference/sql-functions/array-functions/unnest.md) function. [#12484](https://github.com/StarRocks/starrocks/pull/12484)
  - `lead()` and `lag()` functions now support querying HLL and BITMAP data. For more information, refer to [Window function](../sql-reference/sql-functions/Window_function.md). [#12108](https://github.com/StarRocks/starrocks/pull/12108)
  - The following ARRAY functions now support querying JSON data: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md), [array_sort](../sql-reference/sql-functions/array-functions/array_sort.md), [array_concat](../sql-reference/sql-functions/array-functions/array_concat.md), [array_slice](../sql-reference/sql-functions/array-functions/array_slice.md), and [reverse](../sql-reference/sql-functions/array-functions/reverse.md). [#13155](https://github.com/StarRocks/starrocks/pull/13155)
  - Optimized the usage of some functions. The `current_date`, `current_timestamp`, `current_time`, `localtimestamp`, and `localtime` functions can now be executed without using `()`. For example, you can directly run `select current_date;`. [# 14319](https://github.com/StarRocks/starrocks/pull/14319)
- Removed redundant information from FE logs. [# 15374](https://github.com/StarRocks/starrocks/pull/15374)

### Bug Fixes

The following bugs have been fixed:

- The `append_trailing_char_if_absent()` function may return an incorrect result when the first argument is empty. [#13762](https://github.com/StarRocks/starrocks/pull/13762)
- After a table is restored using the `RECOVER` statement, the table does not exist. [#13921](https://github.com/StarRocks/starrocks/pull/13921)
- The result returned by the `SHOW CREATE MATERIALIZED VIEW` statement does not contain the database and catalog specified in the query statement when the materialized view was created. [#12833](https://github.com/StarRocks/starrocks/pull/12833)
- Schema change jobs in the `waiting_stable` state cannot be canceled. [#12530](https://github.com/StarRocks/starrocks/pull/12530)
- Running the `SHOW PROC '/statistic';` command on a Leader FE and non-Leader FE returns different results. [#12491](https://github.com/StarRocks/starrocks/issues/12491)
- The position of the `ORDER BY` clause is incorrect in the result returned by `SHOW CREATE TABLE`. [# 13809](https://github.com/StarRocks/starrocks/pull/13809)
- When users use Hive Catalog to query Hive data, if the execution plan generated by FE does not contain partition IDs, BEs fail to query Hive partition data. [# 15486](https://github.com/StarRocks/starrocks/pull/15486)

### Behavior Change

- Changed the default value of the `AWS_EC2_METADATA_DISABLED` parameter to `False`, meaning that the metadata of Amazon EC2 is obtained to access AWS resources.
- Renamed session variable `is_report_success` to `enable_profile`, which can be queried using the `SHOW VARIABLES` statement.
- Added four reserved keywords: `CURRENT_DATE`, `CURRENT_TIME`, `LOCALTIME`, and `LOCALTIMESTAMP`. [# 14319](https://github.com/StarRocks/starrocks/pull/14319)
- The maximum length of table and database names can now be up to 1023 characters. [# 14929](https://github.com/StarRocks/starrocks/pull/14929) [# 15020](https://github.com/StarRocks/starrocks/pull/15020)
- BE configuration items `enable_event_based_compaction_framework` and `enable_size_tiered_compaction_strategy` are now set to `true` by default, significantly reducing compaction overheads when there are a large number of tablets or a single tablet has a large data volume.

### Upgrade Notes

- You can upgrade your cluster to 2.5.0 from 2.0.x, 2.1.x, 2.2.x, 2.3.x, or 2.4.x. However, if you need to perform a rollback, we recommend that you roll back only to 2.4.x.