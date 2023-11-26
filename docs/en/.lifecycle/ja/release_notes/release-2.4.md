---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.4

## 2.4.5

リリース日: 2023年4月21日

### 改善点

- メタデータのアップグレード時にエラーが発生する可能性があるため、リストパーティションの構文を禁止しました。[#15401](https://github.com/StarRocks/starrocks/pull/15401)
- マテリアライズドビューでBITMAP、HLL、およびPERCENTILEのタイプをサポートしました。[#15731](https://github.com/StarRocks/starrocks/pull/15731)
- `storage_medium`の推論を最適化しました。BEがSSDとHDDの両方をストレージデバイスとして使用する場合、プロパティ`storage_cooldown_time`が指定されている場合、StarRocksは`storage_medium`を`SSD`に設定します。それ以外の場合、StarRocksは`storage_medium`を`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- スレッドダンプの精度を最適化しました。[#16748](https://github.com/StarRocks/starrocks/pull/16748)
- ロード前にメタデータのコンパクションをトリガーしてロード効率を最適化しました。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- ストリームロードプランナーのタイムアウトを最適化しました。[#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- ユニークキーテーブルのパフォーマンスを最適化しました。値の列から統計情報を収集しないようにしました。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

以下のバグが修正されました。

- CREATE TABLEでサポートされていないデータ型が使用された場合、NPEが返されます。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- ショートサーキットを使用したブロードキャストジョインのクエリに誤った結果が返されます。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- データの切り捨てロジックによって引き起こされるディスクの占有問題。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoaderプラグインをインストールまたは削除できません。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- タブレットのスケジュール中に例外がスローされると、同じバッチ内の他のタブレットはスケジュールされません。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 同期マテリアライズドビューの作成時にサポートされていないSQL関数が使用されると、不明なエラーが返されます。[#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 複数のCOUNT DISTINCT計算が誤って書き換えられます。[#19714](https://github.com/StarRocks/starrocks/pull/19714)
- コンパクション中のタブレットでクエリを実行すると、誤った結果が返されます。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 集計を含むクエリに誤った結果が返されます。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- NULLのparquetデータをNOT NULLの列にロードするとエラーメッセージが返されません。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- リソースグループの同時実行制限が連続して達成されると、クエリの同時実行メトリックが遅く減少します。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- `InsertOverwriteJob`の状態変更ログを再生すると、FEの起動に失敗します。[#19061](https://github.com/StarRocks/starrocks/issues/19061)
- プライマリキーテーブルのデッドロック。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- コロケーションテーブルでは、ステートメント`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`のようにして、レプリカのステータスを手動で`bad`に指定できます。BEの数がレプリカの数以下である場合、破損したレプリカを修復できません。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- ARRAY関連の関数によって引き起こされる問題。[#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

リリース日: 2023年2月22日

### 改善点

- ロードの高速キャンセルをサポートしました。[#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- コンパクションフレームワークのCPU使用率を最適化しました。[#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 欠落バージョンのタブレットで累積コンパクションをサポートしました。[#17030](https://github.com/StarRocks/starrocks/pull/17030)

### バグ修正

以下のバグが修正されました。

- 無効なDATE値が指定された場合、過剰な動的パーティションが作成されます。[#17966](https://github.com/StarRocks/starrocks/pull/17966)
- デフォルトのHTTPSポートを使用してElasticsearch外部テーブルに接続できません。[#13726](https://github.com/StarRocks/starrocks/pull/13726)
- トランザクションのタイムアウト後にBEがストリームロードトランザクションをキャンセルできません。[#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 単一のBEでのローカルシャッフル集計から誤ったクエリ結果が返されます。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- クエリが「wait_for_version version: failed: apply stopped」というエラーメッセージで失敗する場合があります。[#17848](https://github.com/StarRocks/starrocks/pull/17850)
- OLAPスキャンバケット式が正しくクリアされないため、誤ったクエリ結果が返されます。[#17666](https://github.com/StarRocks/starrocks/pull/17666)
- コロケーショングループ内の動的パーティションテーブルのバケット数を変更できず、エラーメッセージが返されます。[#17418](https://github.com/StarRocks/starrocks/pull/17418)
- 非リーダーFEノードに接続し、SQLステートメント`USE <catalog_name>.<database_name>`を送信すると、非リーダーFEノードはSQLステートメントをリーダーFEノードに転送します（`<catalog_name>`は除外）。その結果、リーダーFEノードは`default_catalog`を使用しようとし、指定されたデータベースを見つけることができません。[#17302](https://github.com/StarRocks/starrocks/pull/17302)
- 辞書の前にdictmappingチェックのロジックが正しくありません。[#17405](https://github.com/StarRocks/starrocks/pull/17405)
- FEがBEに時折ハートビートを送信し、ハートビート接続がタイムアウトした場合、FEはBEを使用できないと見なし、BEでトランザクションの失敗が発生します。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- フォロワーFE上の新しくクローンされたタブレットで`get_applied_rowsets`が失敗します。[#17192](https://github.com/StarRocks/starrocks/pull/17192)
- フォロワーFE上で`SET variable = default`を実行するとNPEが発生します。[#17549](https://github.com/StarRocks/starrocks/pull/17549)
- プロジェクション内の式が辞書によって書き換えられません。[#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 週単位で動的パーティションを作成するためのロジックが正しくありません。[#17163](https://github.com/StarRocks/starrocks/pull/17163)
- ローカルシャッフルから誤ったクエリ結果が返されます。[#17130](https://github.com/StarRocks/starrocks/pull/17130)
- 増分クローンが失敗する場合があります。[#16930](https://github.com/StarRocks/starrocks/pull/16930)
- CBOが2つの演算子が等価かどうかを比較する際に、正しいロジックを使用しない場合があります。[#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- JuiceFSへのアクセスに失敗するため、JuiceFSスキーマが正しくチェックおよび解決されません。[#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

リリース日: 2023年1月19日

### 改善点

- Analyze中に対応するデータベースとテーブルが存在するかどうかをStarRocksがチェックするようにしました。これにより、NPEを防止します。[#14467](https://github.com/StarRocks/starrocks/pull/14467)
- サポートされていないデータ型の列は、外部テーブルのクエリでマテリアライズされません。[#13305](https://github.com/StarRocks/starrocks/pull/13305)
- FEの起動スクリプト**start_fe.sh**に対してJavaバージョンのチェックを追加しました。[#14333](https://github.com/StarRocks/starrocks/pull/14333)

### バグ修正

以下のバグが修正されました。

- タイムアウトが設定されていない場合、Stream Loadが失敗する場合があります。[#16241](https://github.com/StarRocks/starrocks/pull/16241)
- メモリ使用量が高い場合、bRPC Sendがクラッシュする場合があります。[#16046](https://github.com/StarRocks/starrocks/issues/16046)
- StarRocksの早期バージョンのStarRocksインスタンスからの外部テーブルへのデータのロードに失敗します。[#16130](https://github.com/StarRocks/starrocks/pull/16130)
- マテリアライズドビューのリフレッシュの失敗がメモリリークを引き起こす場合があります。[#16041](https://github.com/StarRocks/starrocks/pull/16041)
- スキーマ変更が公開ステージでハングします。[#14148](https://github.com/StarRocks/starrocks/issues/14148)
- マテリアライズドビューQeProcessorImplの問題によるメモリリーク。[#15699](https://github.com/StarRocks/starrocks/pull/15699)
- `limit`句を持つクエリの結果が一貫していません。[#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERTによるメモリリーク。[#14718](https://github.com/StarRocks/starrocks/pull/14718)
- プライマリキーテーブルがタブレットのマイグレーションを実行します。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- Broker Load中にBroker Kerberosチケットがタイムアウトします。[#16149](https://github.com/StarRocks/starrocks/pull/16149)
- テーブルのビューで`nullable`情報が誤って推論されます。[#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 動作の変更

- Thrift Listenのデフォルトのバックログを`1024`に制約しました。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- SQLモード`FORBID_INVALID_DATES`を追加しました。このSQLモードはデフォルトで無効になっています。有効にすると、StarRocksはDATE型の入力を検証し、入力が無効な場合にエラーを返します。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

リリース日: 2022年12月14日

### 改善点

- バケットヒントのパフォーマンスを最適化しました。多数のバケットが存在する場合に対応します。[#13142](https://github.com/StarRocks/starrocks/pull/13142)

### バグ修正

以下のバグが修正されました。

- プライマリキーインデックスのフラッシュによってBEがクラッシュする場合があります。[#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- マテリアライズドビュータイプが`SHOW FULL TABLES`で正しく識別されません。[#13954](https://github.com/StarRocks/starrocks/pull/13954)
- StarRocks v2.2からv2.4にアップグレードすると、BEがクラッシュする場合があります。[#13795](https://github.com/StarRocks/starrocks/pull/13795)
- Broker LoadがBEをクラッシュさせる場合があります。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- セッション変数`statistic_collect_parallel`が効果を持ちません。[#14352](https://github.com/StarRocks/starrocks/pull/14352)
- INSERT INTOがBEをクラッシュさせる場合があります。[#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDFがBEをクラッシュさせる場合があります。[#13947](https://github.com/StarRocks/starrocks/pull/13947)
- 部分更新中にレプリカをクローンすると、BEがクラッシュし、再起動に失敗する場合があります。[#13683](https://github.com/StarRocks/starrocks/pull/13683)
- コロケートジョインが機能しない場合があります。[#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 動作の変更

- Thrift Listenのデフォルトのバックログを`1024`に制約しました。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- デフォルトでPage Cacheが有効になりました（"disable_storage_page_cache" = "false"）。デフォルトのキャッシュサイズ（`storage_page_cache_limit`）はシステムメモリの20%です。
- デフォルトでCBOが有効になりました。セッション変数`enable_cbo`は非推奨となりました。
- デフォルトでベクトル化エンジンが有効になりました。セッション変数`vectorized_engine_enable`は非推奨となりました。

## 2.4.1

リリース日: 2022年11月14日

### 新機能

- 複数の基本テーブルに基づいて非同期マテリアライズドビューを作成し、JOIN操作を高速化する機能をサポートしました。非同期マテリアライズドビューはすべての[テーブルタイプ](../table_design/table_types/table_types.md)をサポートしています。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。

- INSERT OVERWRITEによるデータの上書きをサポートしました。詳細については、[INSERTを使用したデータのロード](../loading/InsertInto.md)を参照してください。

- [プレビュー] ステートレスなCompute Nodes（CN）を提供し、水平スケーリングを実現できるようにしました。StarRocks Operatorを使用してCNをKubernetes（K8s）クラスタにデプロイし、自動的な水平スケーリングを実現できます。詳細については、[StarRocks Operatorを使用してKubernetes上でCNをデプロイおよび管理する](../deployment/sr_operator.md)を参照してください。

- アウタージョインは、JOINアイテムが`<`、`<=`、`>`、`>=`、および`<>`の比較演算子で関連付けられている非等価ジョインをサポートします。詳細については、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

- Apache IcebergおよびApache Hudiからのデータに直接クエリを実行できるIcebergカタログおよびHudiカタログの作成をサポートしました。詳細については、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)および[Hudiカタログ](../data_source/catalog/hudi_catalog.md)を参照してください。

- CSV形式のApache Hive™テーブルからARRAYタイプの列をクエリできるようになりました。詳細については、[外部テーブル](../data_source/External_table.md)を参照してください。

- DESCを使用して外部データのスキーマを表示できるようになりました。詳細については、[DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md)を参照してください。

- GRANTおよびREVOKEを使用して特定のロールまたはIMPERSONATE権限をユーザーに付与し、EXECUTE ASを使用してIMPERSONATE権限を持つユーザーでSQLステートメントを実行することをサポートしました。詳細については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)、および[EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)を参照してください。

- BEまたはFEノードの一意の識別子としてドメイン名またはホスト名とポートの組み合わせを使用できるようになりました。これにより、IPアドレスが変更されることによるアクセスの失敗を防ぐことができます。詳細については、[FQDNアクセスの有効化](../administration/enable_fqdn.md)を参照してください。

- flink-connector-starrocksがプライマリキーテーブルの部分更新をサポートしました。詳細については、[flink-connector-starrocksを使用したデータのロード](../loading/Flink-connector-starrocks.md)を参照してください。

- 次の新しい関数を提供します。

  - array_contains_all: 特定の配列が別の配列のサブセットであるかどうかをチェックします。詳細については、[array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)を参照してください。
  - percentile_cont: 線形補間を使用してパーセンタイル値を計算します。詳細については、[percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)を参照してください。

### 改善点

- プライマリキーテーブルは、永続的なプライマリキーインデックスの有効化の有無に関係なく、プライマリキーインデックスのデータ型を同じにサポートするようになりました。

- 外部テーブルのクエリパフォーマンスを最適化しました。

  - Parquet形式の外部テーブルのクエリ中に遅延マテリアライゼーションをサポートし、小規模なフィルタリングが含まれるデータレイクのクエリパフォーマンスを最適化します。
  - 小規模なI/O操作をマージしてデータレイクのクエリの遅延を減らし、外部テーブルのクエリパフォーマンスを向上させます。

- ウィンドウ関数のパフォーマンスを最適化しました。

- Cross Joinのパフォーマンスを最適化するために、プレディケートプッシュダウンをサポートしました。

- CBO統計情報にヒストグラムを追加しました。完全な統計情報の収集もさらに最適化されました。詳細については、[CBO統計情報の収集](../using_starrocks/Cost_based_optimizer.md)を参照してください。

- タブレットスキャンのための適応型マルチスレッディングを有効にし、スキャンパフォーマンスがタブレット数に依存しないようにしました。その結果、バケットの数をより簡単に設定できます。詳細については、[バケットの数を決定する方法](../table_design/Data_distribution.md#how-to-determine-the-number-of-buckets)を参照してください。

- Apache Hiveの圧縮されたTXTファイルのクエリをサポートしました。

- デフォルトのPageCacheサイズの計算およびメモリの整合性チェックのメカニズムを調整し、マルチインスタンス展開時のOOM問題を回避します。

- プライマリキーテーブルの大容量バッチロードのパフォーマンスを最大2倍に向上させるために、final_merge操作を削除しました。

- ストリームロードトランザクションインターフェースをサポートし、Apache Flink®やApache Kafka®などの外部システムからデータをロードするために実行されるトランザクションに2フェーズコミット（2PC）を実装し、高並行ストリームロードのパフォーマンスを向上させました。

- 関数:

  - 複数のCOUNT(DISTINCT)を1つのステートメントで使用できます。詳細については、[count](../sql-reference/sql-functions/aggregate-functions/count.md)を参照してください。
  - ウィンドウ関数min()およびmax()はスライディングウィンドウをサポートします。詳細については、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。
  - window_funnel関数のパフォーマンスを最適化しました。詳細については、[window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)を参照してください。

### バグ修正

以下のバグが修正されました。

- DESCによって返されるDECIMALデータ型がCREATE TABLEステートメントで指定されたものと異なる場合があります。[#7309](https://github.com/StarRocks/starrocks/pull/7309)

- FEメタデータ管理の問題がFEの安定性に影響します。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- データロード関連の問題:

  - ARRAY列が指定された場合、Broke Loadが失敗します。[#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - Broker Loadを介して非重複キーテーブルにデータがロードされた後、レプリカが一貫性がなくなります。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - ALTER ROUTINE LOADを実行するとNPEが発生します。[#7804](https://github.com/StarRocks/starrocks/pull/7804)

- データレイク分析関連の問題:

  - Hive外部テーブルのParquetデータのクエリが失敗します。[#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - Elasticsearch外部テーブルの`limit`句を持つクエリに誤った結果が返されます。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 複雑なデータ型を持つApache Icebergテーブルのクエリ中に不明なエラーが発生します。[#11298](https://github.com/StarRocks/starrocks/pull/11298)

- リーダーFEとフォロワーFEノード間のメタデータが一貫していません。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

- BEのサイズが2 GBを超える場合、BEがクラッシュする場合があります。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 動作の変更

- デフォルトでPage Cacheが有効になります（"disable_storage_page_cache" = "false"）。デフォルトのキャッシュサイズ（`storage_page_cache_limit`）はシステムメモリの20%です。
- デフォルトでCBOが有効になります。セッション変数`enable_cbo`は非推奨となります。
- デフォルトでベクトル化エンジンが有効になります。セッション変数`vectorized_engine_enable`は非推奨となります。

## 2.4.0

リリース日: 2022年10月20日

### 新機能

- 複数の基本テーブルに基づいて非同期マテリアライズドビューを作成し、JOIN操作を高速化する機能をサポートしました。非同期マテリアライズドビューはすべての[テーブルタイプ](../table_design/table_types/table_types.md)をサポートしています。詳細については、[マテリアライズドビュー](../using_starrocks/Materialized_view.md)を参照してください。

- INSERT OVERWRITEによるデータの上書きをサポートしました。詳細については、[INSERTを使用したデータのロード](../loading/InsertInto.md)を参照してください。

- [プレビュー] ステートレスなCompute Nodes（CN）を提供し、水平スケーリングを実現できるようにしました。StarRocks Operatorを使用してCNをKubernetes（K8s）クラスタにデプロイし、自動的な水平スケーリングを実現できます。詳細については、[StarRocks Operatorを使用してKubernetes上でCNをデプロイおよび管理する](../deployment/sr_operator.md)を参照してください。

- アウタージョインは、JOINアイテムが`<`、`<=`、`>`、`>=`、および`<>`の比較演算子で関連付けられている非等価ジョインをサポートします。詳細については、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

- Apache IcebergおよびApache Hudiからのデータに直接クエリを実行できるIcebergカタログおよびHudiカタログの作成をサポートしました。詳細については、[Icebergカタログ](../data_source/catalog/iceberg_catalog.md)および[Hudiカタログ](../data_source/catalog/hudi_catalog.md)を参照してください。

- CSV形式のApache Hive™テーブルからARRAYタイプの列をクエリできるようになりました。詳細については、[外部テーブル](../data_source/External_table.md)を参照してください。

- DESCを使用して外部データのスキーマを表示できるようになりました。詳細については、[DESCRIBE](../sql-reference/sql-statements/Utility/DESCRIBE.md)を参照してください。

- GRANTおよびREVOKEを使用して特定のロールまたはIMPERSONATE権限をユーザーに付与し、EXECUTE ASを使用してIMPERSONATE権限を持つユーザーでSQLステートメントを実行することをサポートしました。詳細については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)、および[EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)を参照してください。

- BEまたはFEノードの一意の識別子としてドメイン名またはホスト名とポートの組み合わせを使用できるようになりました。これにより、IPアドレスが変更されることによるアクセスの失敗を防ぐことができます。詳細については、[FQDNアクセスの有効化](../administration/enable_fqdn.md)を参照してください。

- flink-connector-starrocksがプライマリキーテーブルの部分更新をサポートしました。詳細については、[flink-connector-starrocksを使用したデータのロード](../loading/Flink-connector-starrocks.md)を参照してください。

- 次の新しい関数を提供します。

  - array_contains_all: 特定の配列が別の配列のサブセットであるかどうかをチェックします。詳細については、[array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)を参照してください。
  - percentile_cont: 線形補間を使用してパーセンタイル値を計算します。詳細については、[percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)を参照してください。

### 改善点

- プライマリキーテーブルは、永続的なプライマリキーインデックスの有効化の有無に関係なく、プライマリキーインデックスのデータ型を同じにサポートするようになりました。

- 外部テーブルのクエリパフォーマンスを最適化しました。

  - Parquet形式の外部テーブルのクエリ中に遅延マテリアライゼーションをサポートし、小規模なフィルタリングが含まれるデータレイクのクエリパフォーマンスを最適化します。
  - 小規模なI/O操作をマージしてデータレイクのクエリの遅延を減らし、外部テーブルのクエリパフォーマンスを向上させます。

- ウィンドウ関数のパフォーマンスを最適化しました。

- Cross Joinのパフォーマンスを最適化するために、プレディケートプッシュダウンをサポートしました。

- CBO統計情報にヒストグラムを追加しました。完全な統計情報の収集もさらに最適化されました。

- タブレットスキャンのための適応型マルチスレッディングを有効にし、スキャンパフォーマンスがタブレット数に依存しないようにしました。その結果、バケットの数をより簡単に設定できます。

- Apache Hiveの圧縮されたTXTファイルのクエリをサポートしました。

- デフォルトのPageCacheサイズの計算およびメモリの整合性チェックのメカニズムを調整し、マルチインスタンス展開時のOOM問題を回避します。

- プライマリキーテーブルの大容量バッチロードのパフォーマンスを最大2倍に向上させるために、final_merge操作を削除しました。

- ストリームロードトランザクションインターフェースをサポートし、Apache Flink®やApache Kafka®などの外部システムからデータをロードするために実行されるトランザクションに2フェーズコミット（2PC）を実装し、高並行ストリームロードのパフォーマンスを向上させました。

- 関数:

  - 複数のCOUNT(DISTINCT)を1つのステートメントで使用できます。
  - ウィンドウ関数min()およびmax()はスライディングウィンドウをサポートします。

### バグ修正

以下のバグが修正されました。

- DESCによって返されるDECIMALデータ型がCREATE TABLEステートメントで指定されたものと異なる場合があります。

- FEメタデータ管理の問題がFEの安定性に影響します。

- データロード関連の問題:

  - Broke LoadがARRAY列が指定された場合に失敗します。
  - Broker Loadを介して非重複キーテーブルにデータがロードされた後、レプリカが一貫性がなくなります。
  - ALTER ROUTINE LOADを実行するとNPEが発生します。

- データレイク分析関連の問題:

  - Hive外部テーブルのParquetデータのクエリが失敗します。
  - Elasticsearch外部テーブルの`limit`句を持つクエリに誤った結果が返されます。
  - Apache Icebergテーブルのクエリ中に不明なエラーが発生します。

- リーダーFEとフォロワーFEノード間のメタデータが一貫していません。

- BEがサイズが2 GBを超えるBITMAPデータを処理するとクラッシュする場合があります。

### 動作の変更

- デフォルトでPage Cacheが有効になります（"disable_storage_page_cache" = "false"）。デフォルトのキャッシュサイズ（`storage_page_cache_limit`）はシステムメモリの20%です。
- デフォルトでCBOが有効になります。セッション変数`enable_cbo`は非推奨となります。
- デフォルトでベクトル化エンジンが有効になります。セッション変数`vectorized_engine_enable`は非推奨となります。

## その他

- リソースグループの安定リリースを発表します。
- JSONデータ型および関連する関数の安定リリースを発表します。
