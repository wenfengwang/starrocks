---
displayed_sidebar: Chinese
---

# StarRocks バージョン 2.4

## 2.4.5

リリース日：2023年4月21日

### 機能の最適化

- リストパーティションの構文は使用禁止となりました。これはメタデータのアップグレードエラーを引き起こす可能性があるためです。 [#15401](https://github.com/StarRocks/starrocks/pull/15401)
- マテリアライズドビューは、BITMAP、HLL、PERCENTILEのタイプをサポートしています。 [#15731](https://github.com/StarRocks/starrocks/pull/15731)
- `storage_medium` の推論メカニズムが最適化されました。BEがSSDとHDDを同時にストレージメディアとして使用する場合、`storage_cooldown_time`が設定されている場合、StarRocksは`storage_medium`を`SSD`に設定します。それ以外の場合、StarRocksは`storage_medium`を`HDD`に設定します。 [#18649](https://github.com/StarRocks/starrocks/pull/18649)
- スレッドダンプの精度が最適化されました。 [#16748](https://github.com/StarRocks/starrocks/pull/16748)
- インポートの効率を最適化するために、メタデータのコンパクションをインポート前にトリガーします。 [#19347](https://github.com/StarRocks/starrocks/pull/19347)
- Stream Load Plannerのタイムアウトが最適化されました。 [#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- 値の列の統計データの収集を禁止することで、更新モデルテーブルのパフォーマンスを最適化しました。 [#19563](https://github.com/StarRocks/starrocks/pull/19563)

### 問題の修正

以下の問題が修正されました：

- CREATE TABLEでサポートされていないデータ型を使用すると、NPEが返されます。 [#20999](https://github.com/StarRocks/starrocks/issues/20999)
- ブロードキャストジョインとショートサーキットを使用したクエリが誤った結果を返すことがあります。 [#20952](https://github.com/StarRocks/starrocks/issues/20952)
- 誤ったデータ削除ロジックによるディスク使用量の問題が修正されました。 [#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoaderプラグインをインストールまたは削除できない問題が修正されました。 [#20468](https://github.com/StarRocks/starrocks/issues/20468)
- 1つのタブレットのスケジュール時に例外がスローされると、同じバッチの他のタブレットのスケジュールができなくなります。 [#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 同期マテリアライズドビューの作成時にサポートされていないSQL関数が使用されると、未知のエラーが返されます。 [#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 複数のCOUNT DISTINCTのリライトエラーが修正されました。 [#19714](https://github.com/StarRocks/starrocks/pull/19714)
- コンパクション中のタブレットのクエリが誤った結果を返す問題が修正されました。 [#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 集計クエリが誤った結果を返す問題が修正されました。 [#19725](https://github.com/StarRocks/starrocks/issues/19725)
- NULLのParquetデータをNOT NULLの列にロードすると、エラーメッセージが返されません。 [#19885](https://github.com/StarRocks/starrocks/pull/19885)
- リソース分離の並行制限を継続的にトリガーする場合、クエリの並行数の指標が遅く減少します。 [#19363](https://github.com/StarRocks/starrocks/pull/19363)
- `InsertOverwriteJob`のステータス変更ログをリプレイすると、FEが起動に失敗します。 [#19061](https://github.com/StarRocks/starrocks/issues/19061)
- プライマリキーモデルテーブルのデッドロックが修正されました。 [#18488](https://github.com/StarRocks/starrocks/pull/18488)
- Colocationテーブルでは、`ADMIN SET REPLICA STATUS PROPERTIES("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`のようなコマンドで手動でレプリカのステータスを`bad`に指定することができます。BEの数がレプリカの数以下の場合、そのレプリカは修復できません。 [#17876](https://github.com/StarRocks/starrocks/issues/17876)
- ARRAY関連の問題が修正されました。 [#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

リリース日：2023年2月22日

### 機能の最適化

- インポートのキャンセルを高速化するためのサポートが追加されました。 [#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- CompactionフレームワークのCPU使用率が最適化されました。 [#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 欠落しているデータバージョンのタブレットに対する累積的なCompactionがサポートされました。  [#17030](https://github.com/StarRocks/starrocks/pull/17030)

### 問題の修正

以下の問題が修正されました：

- 動的パーティションの作成時に無効なDATE値を指定すると、システムが大量の動的パーティションを作成します。 [#17966](https://github.com/StarRocks/starrocks/pull/17966)
- デフォルトのHTTPSポートでElasticsearch外部テーブルに接続できない問題が修正されました。 [#13726](https://github.com/StarRocks/starrocks/pull/13726)
- Stream Loadトランザクションのタイムアウト後、BEがトランザクションをキャンセルできない問題が修正されました。 [#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 単一のBE上のローカルシャッフル集約が誤ったクエリ結果を返す問題が修正されました。 [#17845](https://github.com/StarRocks/starrocks/pull/17845)
- クエリが失敗し、「wait_for_version version:failed:apply stopped」というエラーメッセージが返されることがあります。 [#17848](https://github.com/StarRocks/starrocks/pull/17850)
- OLAP Scanのバケット式が正しくクリアされず、誤ったクエリ結果が返される問題が修正されました。 [#17666](https://github.com/StarRocks/starrocks/pull/17666)
- Colocateグループ内の動的パーティションテーブルのバケット数を変更できず、エラーメッセージが返される問題が修正されました。 [#17418](https://github.com/StarRocks/starrocks/pull/17418)
- リーダーではないFEノードに接続し、「USE <catalog_name>.<database_name>」というリクエストを送信すると、リーダーFEにリクエストが転送される際に`catalog_name`が転送されず、リーダーFEは`default_catalog`を使用するため、対応するデータベースを見つけることができません。 [#17302](https://github.com/StarRocks/starrocks/pull/17302)
- リライト前のdictmappingチェックのロジックエラーが修正されました。 [#17405](https://github.com/StarRocks/starrocks/pull/17405)
- FEがBEに単発のハートビートを送信し、ハートビート接続がタイムアウトすると、FEはBEが利用できないと判断し、最終的にはそのBE上でのトランザクションの実行に失敗します。 [#16386](https://github.com/StarRocks/starrocks/pull/16386)
- FEフォロワーで新しくクローンされたタブレットをクエリすると、`get_applied_rowsets`が失敗します。 [#17192](https://github.com/StarRocks/starrocks/pull/17192)
- FEフォロワーで`SET variable = default`を実行すると、NullPointerException（NPE）が発生します。 [#17549](https://github.com/StarRocks/starrocks/pull/17549)
- プロジェクション内の式が辞書によって書き換えられない問題が修正されました。 [#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 週単位で動的パーティションを作成する際のロジックエラーが修正されました。 [#17163](https://github.com/StarRocks/starrocks/pull/17163)
- ローカルシャッフルが誤ったクエリ結果を返す問題が修正されました。 [#17130](https://github.com/StarRocks/starrocks/pull/17130)
- 増分クローンが失敗する可能性があります。 [#16930](https://github.com/StarRocks/starrocks/pull/16930)
- 一部の場合において、CBOの比較演算子の等価性のロジックが誤っています。[#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- JuiceFSスキーマのチェックと解析が正しく行われないため、JuiceFSへのアクセスが失敗する問題が修正されました。 [#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

リリース日：2023年1月19日

### 機能の最適化

- Analyze時に、NPEを回避するためにStarRocksは事前にDatabaseとTableの存在をチェックします。[#14467](https://github.com/StarRocks/starrocks/pull/14467)
- 外部テーブルのクエリ時に、サポートされていない列のデータ型は物質化されません。[#13305](https://github.com/StarRocks/starrocks/pull/13305)
- FEの起動スクリプト **start_fe.sh** にJavaバージョンのチェックが追加されました。[#14333](https://github.com/StarRocks/starrocks/pull/14333)

### 問題の修正

以下の問題が修正されました：

- タイムアウトが設定されていないため、Stream Loadが失敗することがあります。[#16241](https://github.com/StarRocks/starrocks/pull/16241)
- メモリ使用量が多い場合、bRPC Sendがクラッシュすることがあります。[#16046](https://github.com/StarRocks/starrocks/issues/16046)
- 低バージョンのStarRocks外部テーブルの方法でデータをインポートできない問題が修正されました。[#16130](https://github.com/StarRocks/starrocks/pull/16130)
- マテリアライズドビューのリフレッシュに失敗すると、メモリリークが発生することがあります。[#16041](https://github.com/StarRocks/starrocks/pull/16041)
- スキーマ変更がPublishの段階でスタックすることがあります。[#14148](https://github.com/StarRocks/starrocks/issues/14148)
- マテリアライズドビューQeProcessorImplの問題がメモリリークを引き起こす可能性があります。[#15699](https://github.com/StarRocks/starrocks/pull/15699)
- LIMITクエリの結果が一貫していません。[#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERTのインポートがメモリリークを引き起こすことがあります。[#14718](https://github.com/StarRocks/starrocks/pull/14718)
- 主キーのテーブルでのタブレットの移行が修正されました。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- Broker LoadがBEをクラッシュさせる可能性があります。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- セッション変数`statistic_collect_parallel`が機能しない問題が修正されました。[#14352](https://github.com/StarRocks/starrocks/pull/14352)

### 動作の変更

- Thrift ListenのBacklogのデフォルト値を`1024`に変更しました。 [#13911](https://github.com/StarRocks/starrocks/pull/13911)
- `FORBID_INVALID_DATES`のSQLモードが追加され、デフォルトでは無効になっています。有効にすると、日付の入力が検証され、不正な日付の場合にエラーが発生します。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

リリース日：2022年12月14日

### 機能の最適化

- 大量のバケットが存在する場合のBucket Hintのパフォーマンスが最適化されました。[#13142](https://github.com/StarRocks/starrocks/pull/13142)

### 問題の修正

以下の問題が修正されました：

- プライマリキーインデックスのディスク書き込みがBEのクラッシュを引き起こす可能性があります。[#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- マテリアライズドビューテーブルのタイプが`SHOW FULL TABLES`で正しく識別されない問題が修正されました。[#13954](https://github.com/StarRocks/starrocks/pull/13954)
- StarRocks v2.2からv2.4へのアップグレードがBEのクラッシュを引き起こす可能性があります。[#13795](https://github.com/StarRocks/starrocks/pull/13795)
- Broker LoadがBEをクラッシュさせる可能性があります。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- セッション変数`statistic_collect_parallel`が機能しない問題が修正されました。[#14352](https://github.com/StarRocks/starrocks/pull/14352)

- INSERT INTO によってBEがクラッシュする可能性があります。[#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDFによってBEがクラッシュする可能性があります。[#13947](https://github.com/StarRocks/starrocks/pull/13947)
- Partial Update時にレプリカのクローンがBEのクラッシュを引き起こし、再起動できなくなる可能性があります。[#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Colocate Joinが機能しない可能性があります。[#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 動作変更

- セッション変数 `query_timeout` に最大値 `259200` と最小値 `1` の制限を追加しました。
- FEパラメータ `default_storage_medium` をキャンセルし、テーブルのストレージメディアはシステムによって自動的に推測されるようになりました。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

リリース日：2022年11月14日

### 新機能

- 非等値のLEFT SEMI/ANTI JOINのサポートを追加し、JOIN機能を強化しました。[#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 機能の最適化

- `HeartbeatResponse`に`aliveStatus`プロパティを追加し、ノードのオンライン状態を判断するためのロジックを最適化しました。[#12713](https://github.com/StarRocks/starrocks/pull/12713)

- Routine Loadのエラーメッセージの表示を最適化しました。[#12155](https://github.com/StarRocks/starrocks/pull/12155)

### 問題の修正

以下の問題を修正しました：

- 2.4.0 RCから2.4.0にアップグレードすると、BEがクラッシュする問題を修正しました。[#13128](https://github.com/StarRocks/starrocks/pull/13128)

- データレイクのクエリ時に遅延マテリアライズがクエリ結果の誤りを引き起こす問題を修正しました。[#13133](https://github.com/StarRocks/starrocks/pull/13133)

- get_json_int関数のエラーを修正しました。[#12997](https://github.com/StarRocks/starrocks/pull/12997)

- インデックスがディスクに書き込まれた主キーテーブルでデータを削除すると、データの不整合が発生する可能性がある問題を修正しました。[#12719](https://github.com/StarRocks/starrocks/pull/12719)

- 主キーテーブルのCompactionがBEのクラッシュを引き起こす可能性がある問題を修正しました。[#12914](https://github.com/StarRocks/starrocks/pull/12914)

- json_object関数に空の文字列が含まれている場合に、エラーの結果が返される問題を修正しました。[#13030](https://github.com/StarRocks/starrocks/issues/13030)

- RuntimeFilterがBEのクラッシュを引き起こす問題を修正しました。[#12807](https://github.com/StarRocks/starrocks/pull/12807)

- CBO内の過剰な再帰計算がFEをハングアップさせる問題を修正しました。[#12788](https://github.com/StarRocks/starrocks/pull/12788)

- Graceful Shutdown時にBEがクラッシュしたりエラーが発生する可能性がある問題を修正しました。[#12852](https://github.com/StarRocks/starrocks/pull/12852)

- 新しい列を追加した後、削除がCompactionのクラッシュを引き起こす問題を修正しました。[#12907](https://github.com/StarRocks/starrocks/pull/12907)

- OLAP外部テーブルのメタデータ同期により、データの不整合が発生する問題を修正しました。[#12368](https://github.com/StarRocks/starrocks/pull/12368)

- 1つのBEがクラッシュした後、関連するクエリが他のBEでタイムアウトまで実行され続ける可能性がある問題を修正しました。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 動作変更

- Hive外部テーブルの解析エラーが発生した場合、StarRocksはエラーを報告し、関連する列をNULLに設定しません。[#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

リリース日：2022年10月20日

### 新機能

- 非同値のLEFT SEMI/ANTI JOINをサポートし、JOIN機能を強化しました。非同値のJOINは、複数のテーブルを結合するための演算子です。[#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 機能の最適化

- `HeartbeatResponse`に`aliveStatus`プロパティを追加し、ノードのオンライン状態を判断するためのロジックを最適化しました。[#12713](https://github.com/StarRocks/starrocks/pull/12713)

- Routine Loadのエラーメッセージの表示を最適化しました。[#12155](https://github.com/StarRocks/starrocks/pull/12155)

### 問題の修正

以下の問題を修正しました：

- DESCコマンドで表示されるテーブルの構造情報のフィールドタイプが、テーブル作成時に指定されたフィールドタイプと異なる問題を修正しました。[#7309](https://github.com/StarRocks/starrocks/pull/7309)

- FEの安定性に影響を与えるメタデータの問題を修正しました。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- インポートに関連する問題を修正しました：
  - Broker LoadでARRAY列を設定する際に失敗する問題を修正しました。[#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - Broker Loadを使用して非ディテールモデルテーブルにデータをインポートした後、レプリカのデータが一致しない問題を修正しました。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - ALTER ROUTINE LOADの実行中にNPEエラーが発生する問題を修正しました。[#7804](https://github.com/StarRocks/starrocks/pull/7804)

- データレイク分析に関連する問題を修正しました：
  - Hive外部テーブルのParquet形式のデータのクエリに失敗する問題を修正しました。[#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - Elasticsearch外部テーブルのLIMITクエリの結果が正しくない問題を修正しました。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 複雑なデータ型を持つApache Icebergテーブルのクエリが未知のエラーを返す問題を修正しました。[#11298](https://github.com/StarRocks/starrocks/pull/11298)

- リーダーFEノードとフォロワーFEノード間のメタデータの同期が行われない問題を修正しました。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

- BITMAP型のデータが2GBを超えると、BEが停止する問題を修正しました。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 動作変更


```
- デフォルトで Page Cache が有効化されています（"disable_storage_page_cache" = "false"）。キャッシュサイズ (`storage_page_cache_limit`) はシステムメモリの 20% です。
- デフォルトで CBO オプティマイザーが有効化され、session 変数 `enable_cbo` は廃止されました。
- デフォルトでベクトル化エンジンが有効化され、session 変数 `vectorized_engine_enable` は廃止されました。

### その他

- リソース隔離機能が正式にサポートされました。
- JSON データ型および関連する関数が正式にサポートされました。

