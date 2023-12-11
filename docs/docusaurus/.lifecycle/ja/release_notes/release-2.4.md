---
displayed_sidebar: "Japanese"
---

# StarRocksバージョン2.4

## 2.4.5

リリース日: 2023年4月21日

### 改善点

- List Partition構文を禁止しました。メタデータのアップグレード時にエラーが発生する可能性があるためです。[#15401](https://github.com/StarRocks/starrocks/pull/15401)
- マテリアライズドビューでBITMAP、HLL、PERCENTILEタイプをサポートしました。[#15731](https://github.com/StarRocks/starrocks/pull/15731)
- `storage_medium`の推論を最適化しました。BEがSSDとHDDの両方をストレージデバイスとして使用する場合、`storage_cooldown_time`プロパティが指定されている場合、StarRocksは`storage_medium`を`SSD`に設定します。 それ以外の場合、StarRocksは`storage_medium`を`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- スレッドダンプの精度を最適化しました。[#16748](https://github.com/StarRocks/starrocks/pull/16748)
- ロード前にメタデータの圧縮をトリガーすることでロード効率を最適化しました。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- Stream Loadプランナータイムアウトを最適化しました。[#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- ユニークキーテーブルのパフォーマンスを最適化しました。値の列から統計情報を収集することを禁止することで実現しました。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

以下のバグが修正されました：

- CREATE TABLEでサポートされていないデータ型が使用されると、NPEが返されます。[#20999](https://github.com/StarRocks/starrocks/issues/20999)
- 短絡的なBroadcast Joinを使用するクエリに誤った結果が返されます。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- メタデータの切り詰めロジックの間違ったデータトランケートによるディスク使用量の問題。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoaderプラグインをインストールまたは削除できません。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- タブレットのスケジュール中に例外がスローされると、同じバッチ内の他のタブレットは決してスケジュールされません。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- 同期マテリアライズドビューの作成時にサポートされていないSQL関数が使用されると、Unknown Errorが返されます。[#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 複数のCOUNT DISTINCT計算が誤って書き換えられます。[#19714](https://github.com/StarRocks/starrocks/pull/19714)
- コンパクション中のタブレットに対するクエリで誤った結果が返されます。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 集約を含むクエリで誤った結果が返されます。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- NULLのParquetデータをNOT NULLの列にロードする際にエラーメッセージが返されません。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- リソースグループの同時実行制限が連続して達成されると、クエリの同時実行メトリックが遅く減少します。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- `InsertOverwriteJob`ステータス変更ログを再生する際にFEが起動に失敗します。[#19061](https://github.com/StarRocks/starrocks/issues/19061)
- プライマリキーテーブルのデッドロック。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- Colocationテーブルでは、「ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");」のようなステートメントを使用してレプリカの状態を手動で指定できます。 BEの数がレプリカの数以下である場合、壊れたレプリカを修復できません。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- ARRAY関連関数によって引き起こされる問題。[#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

リリース日: 2023年2月22日

### 改善点

- ロードの高速なキャンセルをサポートしました。[#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- コンパクションフレームワークのCPU使用率を最適化しました。[#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 欠落したバージョンのタブレットで累積コンパクションをサポートしました。[#17030](https://github.com/StarRocks/starrocks/pull/17030)

### バグ修正

以下のバグが修正されました：

- 無効なDATE値が指定された場合、過剰なダイナミックパーティションが作成されます。[#17966](https://github.com/StarRocks/starrocks/pull/17966)
- デフォルトのHTTPSポートでElasticsearch外部テーブルに接続できない問題を修正しました。[#13726](https://github.com/StarRocks/starrocks/pull/13726)
- トランザクションのタイムアウト後にBEがStream Loadトランザクションをキャンセルできなかった問題を修正しました。[#15738](https://github.com/StarRocks/starrocks/pull/15738)
- シングルBEでのローカルシャッフル集約から誤ったクエリ結果が返される問題を修正しました。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- "wait_for_version version: failed: apply stopped"というエラーメッセージによりクエリが失敗する問題を修正しました。[#17848](https://github.com/StarRocks/starrocks/pull/17850)
- OLAPスキャンバケット式を正しくクリアしないため、誤ったクエリ結果が返される問題を修正しました。[#17666](https://github.com/StarRocks/starrocks/pull/17666)
- Colocateグループ内のダイナミックパーティション化されたテーブルのバケット数を変更できず、エラーメッセージが返される問題を修正しました。[#17418](https://github.com/StarRocks/starrocks/pull/17418)
- ノンリーダーFEノードに接続し、SQLステートメント`USE <catalog_name>.<database_name>`を送信すると、非リーダーFEノードはSQLステートメントをリーダーFEノードに転送し、`<catalog_name>`を除外して`default_catalog`を使用するようにします。その結果、リーダーFEノードは指定されたデータベースを見つけることができず、最終的に失敗します。[#17302](https://github.com/StarRocks/starrocks/pull/17302)
- リライト前のdictmappingチェックの不正確なロジックを修正しました。[#17405](https://github.com/StarRocks/starrocks/pull/17405)
- FEが時々BEにハートビートを送信し、ハートビート接続がタイムアウトすると、FEはBEを使用不能と見なし、BEでトランザクションの失敗を引き起こす問題を修正しました。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- 新しくクローンされたフォロワFE上のクエリで`get_applied_rowsets`が失敗する問題を修正しました。[#17192](https://github.com/StarRocks/starrocks/pull/17192)
- フォロワーFE上で`SET variable = default`を実行することによって発生するNPEを修正しました。[#17549](https://github.com/StarRocks/starrocks/pull/17549)
- プロジェクションの式がディクショナリによってリライトされない問題を修正しました。[#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 週単位のダイナミックパーティションの作成のための不正確なロジックを修正しました。[#17163](https://github.com/StarRocks/starrocks/pull/17163)
- ローカルシャッフルから誤ったクエリ結果が返される問題を修正しました。[#17130](https://github.com/StarRocks/starrocks/pull/17130)
- インクリメンタルクローンが失敗することがあります。[#16930](https://github.com/StarRocks/starrocks/pull/16930)
- CBOが2つの演算子が等価かどうかを比較するための不正確な論理を使用することがあります。[#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- JuiceFSへのアクセスに失敗します。JuiceFSスキーマが正しくチェックおよび解決されないための問題を修正しました。[#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

リリース日: 2023年1月19日

### 改善点

- NPEを防ぐために、StarRocksはAnalyze中に対応するデータベースとテーブルが存在するかをチェックします。[#14467](https://github.com/StarRocks/starrocks/pull/14467)
- サポートされていないデータ型を持つ列は、外部テーブルのクエリに対してマテリアライズされません。[#13305](https://github.com/StarRocks/starrocks/pull/13305)
- FEの起動スクリプト**start_fe.sh**に対してJavaバージョンのチェックを追加しました。[#14333](https://github.com/StarRocks/starrocks/pull/14333)

### バグ修正

以下のバグが修正されました：
- タイムアウトが設定されていない場合、Stream Loadは失敗する可能性があります。[#16241](https://github.com/StarRocks/starrocks/pull/16241)
- メモリ使用量が高いとbRPC Sendがクラッシュする可能性があります。[#16046](https://github.com/StarRocks/starrocks/issues/16046)
- 早いバージョンのStarRocksインスタンスから外部テーブルにデータをロードできない場合があります。[#16130](https://github.com/StarRocks/starrocks/pull/16130)
- Materialized viewのリフレッシュの失敗はメモリリークを引き起こす可能性があります。[#16041](https://github.com/StarRocks/starrocks/pull/16041)
- スキーマ変更がPublishステージでハングすることがあります。[#14148](https://github.com/StarRocks/starrocks/issues/14148)
- Materialized viewのQeProcessorImplの問題によるメモリリークが発生する可能性があります。[#15699](https://github.com/StarRocks/starrocks/pull/15699)
- `limit` を含むクエリの結果が一貫していません。[#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERTによるメモリリークが発生する可能性があります。[#14718](https://github.com/StarRocks/starrocks/pull/14718)
- プライマリキー テーブルがTablet Migrationを実行します。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- Broker Load中にBroker Kerberosチケットがタイムアウトすることがあります。[#16149](https://github.com/StarRocks/starrocks/pull/16149)
- テーブルのビューで`nullable`情報が誤って推論されます。[#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 動作変更

- Thrift Listenのデフォルトのバックログを `1024` に変更しました。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- SQLモード`FORBID_INVALID_DATES`を追加しました。このSQLモードはデフォルトで無効になっています。有効にすると、StarRocksはDATE型の入力を検証し、入力が無効な場合にエラーを返します。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

リリース日: 2022年12月14日

### 改善

- 大量のバケットが存在する場合のBucket Hintのパフォーマンスを最適化しました。[#13142](https://github.com/StarRocks/starrocks/pull/13142)

### バグ修正

次のバグが修正されました:

- プライマリキーインデックスをFlushするとBEがクラッシュする可能性があります。[#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- Materialized viewの種類が`SHOW FULL TABLES`で正しく識別されないことがあります。[#13954](https://github.com/StarRocks/starrocks/pull/13954)
- StarRocks v2.2をv2.4にアップグレードするとBEがクラッシュする場合があります。[#13795](https://github.com/StarRocks/starrocks/pull/13795)
- Broker LoadがBEをクラッシュさせる場合があります。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- セッション変数`statistic_collect_parallel`が効果を持たないことがあります。[#14352](https://github.com/StarRocks/starrocks/pull/14352)
- INSERT INTOがBEをクラッシュさせる可能性があります。[#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDFがBEをクラッシュさせる可能性があります。[#13947](https://github.com/StarRocks/starrocks/pull/13947)
- 部分更新中のレプリカのクローンがBEをクラッシュさせ、再起動に失敗する可能性があります。[#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Co-located Joinが効果を持たないことがあります。[#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 動作変更

- セッション変数`query_timeout`を上限`259200`と下限`1`で制約しました。
- FEパラメータ`default_storage_medium`を廃止しました。テーブルのストレージメディアはシステムによって自動的に推論されます。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

リリース日: 2022年11月14日

### 新機能

- 非等結合 - 左半分結合とアンチ結合をサポートします。JOIN機能を最適化しました。[#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 改善

- `HeartbeatResponse`の`aliveStatus`をサポートします。`aliveStatus`は、クラスタ内のノードが生きているかどうかを示します。`aliveStatus`を判定するメカニズムをさらに最適化しました。[#12713](https://github.com/StarRocks/starrocks/pull/12713)
- Routine Loadのエラーメッセージを最適化しました。[#12155](https://github.com/StarRocks/starrocks/pull/12155)

### バグ修正

- v2.4.0RCからv2.4.0にアップグレードした後にBEがクラッシュすることがあります。[#13128](https://github.com/StarRocks/starrocks/pull/13128)
- 遅延マテリアライゼーションにより、データレイク上のクエリが不正確な結果を返す可能性があります。[#13133](https://github.com/StarRocks/starrocks/pull/13133)
- get_json_int関数が例外をスローすることがあります。[#12997](https://github.com/StarRocks/starrocks/pull/12997)
- プライマリキーテーブルからの削除後、データが不整合になることがあります。[#12719](https://github.com/StarRocks/starrocks/pull/12719)
- プライマリキーテーブルのコンパクション中にBEがクラッシュする可能性があります。[#12914](https://github.com/StarRocks/starrocks/pull/12914)
- 入力が空の場合、json_object関数が間違った結果を返します。[#13030](https://github.com/StarRocks/starrocks/issues/13030)
- `RuntimeFilter`によりBEがクラッシュすることがあります。[#12807](https://github.com/StarRocks/starrocks/pull/12807)
- CBOで過剰な再帰計算によりFEがハングすることがあります。[#12788](https://github.com/StarRocks/starrocks/pull/12788)
- グレースフルな終了時にBEがクラッシュしたりエラーを報告したりすることがあります。[#12852](https://github.com/StarRocks/starrocks/pull/12852)
- テーブルに新しい列が追加された状態でのコンパクション後のクラッシュの可能性があります。[#12907](https://github.com/StarRocks/starrocks/pull/12907)
- OLAP外部テーブルメタデータ同期のメカニズムが不正確であるため、データが不整合になる可能性があります。[#12368](https://github.com/StarRocks/starrocks/pull/12368)
- 1つのBEがクラッシュすると、他のBEがタイムアウトまで関連するクエリを実行する可能性があります。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 動作変更

- Hive外部テーブルのパースに失敗すると、StarRocksは関連する列をNULL列に変換する代わりにエラーメッセージをスローします。[#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

リリース日: 2022年10月20日

### 新機能

- 複数の基本テーブルに基づく非同期マテリアライズドビューの作成をサポートし、JOIN操作を高速化します。非同期マテリアライズドビューはすべての[table types](../table_design/table_types/table_types.md)をサポートします。詳細は[Materialized View](../using_starrocks/Materialized_view.md)を参照してください。

- INSERT OVERWRITEを使用してデータを上書きする機能をサポートします。詳細は[Load data using INSERT](../loading/InsertInto.md)を参照してください。

- [プレビュー] 無状態のComputeノード（CN）を作成し、水平スケーリングを実現するためにStarRocks Operatorを使用してKubernetes（K8s）クラスタにCNをデプロイできます。詳細は[Deploy and manage CN on Kubernetes with StarRocks Operator](../deployment/sr_operator.md)を参照してください。

- Outer Joinで、`<`, `<=`, `>`, `>=`, `<>`などの比較演算子によって関連付けられる非等結合をサポートします。詳細は[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。

- Apache IcebergおよびApache Hudiのデータに直接クエリを実行できるIcebergカタログおよびHudiカタログを作成する機能をサポートします。詳細は[Iceberg catalog](../data_source/catalog/iceberg_catalog.md)および[Hudi catalog](../data_source/catalog/hudi_catalog.md)を参照してください。

- CSV形式のApache Hive™テーブルのARRAY型列をクエリする機能をサポートします。詳細は[External table](../data_source/External_table.md)を参照してください。

- DESCを使用して外部データのスキーマを表示する機能をサポートします。詳細は[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)を参照してください。
- ユーザーに特定のロールやIMPERSONATE権限をGRANTを介して付与およびREVOKEを介して取り消すことができ、IMPERSONATE権限を使用してSQLステートメントを実行することができます。詳細については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)、および[EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)を参照してください。

- FDQNアクセスをサポート: 現在、ドメイン名またはホスト名とポートの組み合わせをBEまたはFEノードのユニークな識別子として使用できます。これにより、IPアドレスの変更によるアクセスの失敗を防ぐことができます。詳細については、[FQDNアクセスの有効化](../administration/enable_fqdn.md)を参照してください。

- flink-connector-starrocksは、プライマリキー表の部分的な更新をサポートしています。詳細については、[flink-connector-starrocksを使用したデータのロード](../loading/Flink-connector-starrocks.md)を参照してください。

- 以下の新しい機能を提供しています:

  - array_contains_all: 特定の配列が別の配列の部分集合であるかどうかをチェックします。詳細については、[array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)を参照してください。
  - percentile_cont: 線形補間を使用してパーセンタイル値を計算します。詳細については、[percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)を参照してください。

### 改善

- プライマリキー表は、VARCHAR型のプライマリキーインデックスをディスクにフラッシュすることをサポートしています。バージョン2.4.0以降、プライマリキー表は永続的なプライマリキーインデックスがオンになっているかどうかにかかわらず、プライマリキーインデックスのための同じデータ型をサポートしています。

- 外部テーブルのクエリパフォーマンスを最適化しました。

  - Parquetフォーマットの外部テーブルのクエリ中に遅延マテリアライズがサポートされ、小規模のフィルタリングが関与するデータレイク上のクエリパフォーマンスを最適化します。
  - 小規模のI/O操作をマージして、データレイクのクエリ遅延を減少させ、外部テーブルのクエリパフォーマンスを向上させます。

- ウィンドウ関数のパフォーマンスを最適化しました。

- プレディケートプッシュダウンをサポートして、Cross Joinのパフォーマンスを最適化しました。

- CBO統計にヒストグラムが追加されました。完全な統計情報収集がさらに最適化されました。詳細については、[CBO統計の収集](../using_starrocks/Cost_based_optimizer.md)を参照してください。

- タブレットスキャンのために適応的なマルチスレッディングが有効化され、スキャンパフォーマンスの依存性がタブレット数により少なくなります。その結果、バケツの数をより簡単に設定できます。詳細については、[バケットの数を決定する](../table_design/Data_distribution.md#how-to-determine-the-number-of-buckets)を参照してください。

- Apache Hiveで圧縮されたTXTファイルのクエリをサポートします。

- デフォルトのPageCacheサイズ計算およびメモリ整合性チェックのメカニズムを調整し、マルチインスタンス展開中のOOM問題を避けるために改善しました。

- 最終マージ操作を削除することで、プライマリキー表への大規模なバッチロードのパフォーマンスを最大2倍に改善しました。

- 外部システム（Apache Flink®やApache Kafka®など）からデータをロードするために、2相コミット（2PC）を実装するためのストリームロードトランザクションインターフェースをサポートし、高い並行ストリームロードのパフォーマンスを向上させました。

- 関数:

  - 1つのステートメントで複数のCOUNT(DISTINCT)を使用できます。詳細については、[count](../sql-reference/sql-functions/aggregate-functions/count.md)を参照してください。
  - ウィンドウ関数min()およびmax()はスライディングウィンドウをサポートします。詳細については、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。
  - window_funnel関数のパフォーマンスを最適化しました。詳細については、[window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)を参照してください。

### バグ修正

以下のバグが修正されました:

- DESCによって返されるDECIMALデータ型がCREATE TABLEステートメントで指定されているものと異なります。[#7309](https://github.com/StarRocks/starrocks/pull/7309)

- FEメタデータ管理に影響を与える問題を修正しました。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- データロードに関連する問題:

  - ARRAY列が指定されているとLoadが失敗する問題を修正しました。[#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - ブローカーロードを介して非重複キー表にデータをロードした後、レプリカが一貫性がなくなる問題を修正しました。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - ALTER ROUTINE LOADを実行した際にNPEが発生する問題を修正しました。[#7804](https://github.com/StarRocks/starrocks/pull/7804)

- データレイク分析に関連する問題:

  - Hive外部テーブルのParquetデータのクエリが失敗する問題を修正しました。[#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - Elasticsearch外部テーブルで`limit`句を含むクエリの結果が不正確である問題を修正しました。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 複雑なデータ型を持つApache Icebergテーブルでのクエリ中に不明なエラーが発生する問題を修正しました。[#11298](https://github.com/StarRocks/starrocks/pull/11298)

- リーダーFEとフォロワーFEノード間のメタデータが一貫していない問題を修正しました。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

- BITMAPデータのサイズが2 GBを超えるとBEがクラッシュする問題を修正しました。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 動作の変更

- ページキャッシュはデフォルトで有効になります（"disable_storage_page_cache" = "false"）。デフォルトのキャッシュサイズ（`storage_page_cache_limit`）はシステムメモリの20%です。
- CBOはデフォルトで有効になります。セッション変数`enable_cbo`は非推奨です。
- ベクトル化エンジンはデフォルトで有効になります。セッション変数`vectorized_engine_enable`は非推奨です。

### その他

- リソースグループの安定リリースを発表します。
- JSONデータ型および関連する機能の安定リリースを発表します。