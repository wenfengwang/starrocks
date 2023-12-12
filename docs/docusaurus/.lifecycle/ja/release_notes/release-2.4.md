---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 2.4

## 2.4.5

リリース日: 2023年4月21日

### 改善点

- リストパーティション構文を禁止しました。これはメタデータのアップグレードでエラーが発生する可能性があるためです。[#15401](https://github.com/StarRocks/starrocks/pull/15401)
- マテリアライズド・ビューでのBITMAP、HLL、およびPERCENTILEタイプをサポートしました。[#15731](https://github.com/StarRocks/starrocks/pull/15731)
- `storage_medium`の推論を最適化しました。BEがSSDとHDDの両方をストレージデバイスとして使用する場合、プロパティ`storage_cooldown_time`が指定されている場合、StarRocksは`storage_medium`を`SSD`に設定します。それ以外の場合は、StarRocksは`storage_medium`を`HDD`に設定します。[#18649](https://github.com/StarRocks/starrocks/pull/18649)
- スレッドダンプの精度を最適化しました。[#16748](https://github.com/StarRocks/starrocks/pull/16748)
- ロード効率を最適化するために、ロード前にメタデータのコンパクションをトリガーしました。[#19347](https://github.com/StarRocks/starrocks/pull/19347)
- ストリームロードプランナーのタイムアウトを最適化しました。[#18992](https://github.com/StarRocks/starrocks/pull/18992/files)
- ユニークキーのテーブルパフォーマンスを最適化しました。これにより、値の列から統計情報の収集が禁止されます。[#19563](https://github.com/StarRocks/starrocks/pull/19563)

### バグ修正

次のバグが修正されています。

- CREATE TABLEでサポートされていないデータ型が使用されると、NPEが返されます。[# 20999](https://github.com/StarRocks/starrocks/issues/20999)
- ショートサーキットを使用したBroadcast Joinのクエリに誤った結果が返されます。[#20952](https://github.com/StarRocks/starrocks/issues/20952)
- メタデータの誤ったコンパクションロジックによって引き起こされるディスク占有問題。[#20590](https://github.com/StarRocks/starrocks/pull/20590)
- AuditLoaderプラグインをインストールまたは削除できません。[#20468](https://github.com/StarRocks/starrocks/issues/20468)
- タブレットのスケジュール中に例外がスローされると、同じバッチ内の他のタブレットはスケジュールされません。[#20681](https://github.com/StarRocks/starrocks/pull/20681)
- サポートされていないSQL関数が同期マテリアライズド・ビューの作成時に使用されると、Unknown Errorが返されます。[#20348](https://github.com/StarRocks/starrocks/issues/20348)
- 複数のCOUNT DISTINCT計算が誤って書き換えられます。[#19714](https://github.com/StarRocks/starrocks/pull/19714)
- コンパクション中のタブレットへのクエリに誤った結果が返されます。[#20084](https://github.com/StarRocks/starrocks/issues/20084)
- 集約を使用したクエリに誤った結果が返されます。[#19725](https://github.com/StarRocks/starrocks/issues/19725)
- NULLのParquetデータをNOT NULL列にロードする際にエラーメッセージが返されません。[#19885](https://github.com/StarRocks/starrocks/pull/19885)
- リソースグループの同時実行制限が連続して達成された場合、クエリの同時実行メトリックが遅く減少します。[#19363](https://github.com/StarRocks/starrocks/pull/19363)
- `InsertOverwriteJob`の状態変更ログをリプレイする際にFEの起動に失敗します。[#19061](https://github.com/StarRocks/starrocks/issues/19061)
- プライマリキーのテーブルでデッドロックが発生します。[#18488](https://github.com/StarRocks/starrocks/pull/18488)
- Colocationテーブルでは、`ADMIN SET REPLICA STATUS PROPERTIES ("tablet_id" = "10003", "backend_id" = "10001", "status" = "bad");`のようなステートメントを使用してレプリカの状態を手動で指定することができます。BEの数がレプリカの数以下である場合、破損したレプリカを修復することができません。[#17876](https://github.com/StarRocks/starrocks/issues/17876)
- ARRAY関連の機能に起因する問題。[#18556](https://github.com/StarRocks/starrocks/pull/18556)

## 2.4.4

リリース日: 2023年2月22日

### 改善点

- ロードの高速キャンセルをサポートしました。[#15514](https://github.com/StarRocks/starrocks/pull/15514) [#15398](https://github.com/StarRocks/starrocks/pull/15398) [#15969](https://github.com/StarRocks/starrocks/pull/15969)
- コンパクションフレームワークのCPU使用率を最適化しました。[#11747](https://github.com/StarRocks/starrocks/pull/11747)
- 欠落したバージョンのタブレットでの累積コンパクションをサポートしました。[#17030](https://github.com/StarRocks/starrocks/pull/17030)

### バグ修正

次のバグが修正されています:

- 無効なDATE値が指定されると過剰なダイナミックパーティションが作成されます。[#17966](https://github.com/StarRocks/starrocks/pull/17966)
- デフォルトのHTTPSポートでElasticsearch外部テーブルに接続できない問題が修正されました。[#13726](https://github.com/StarRocks/starrocks/pull/13726)
- BEがトランザクションのタイムアウト後にストリームロードトランザクションをキャンセルできない問題が修正されました。[#15738](https://github.com/StarRocks/starrocks/pull/15738)
- 単一のBEでのローカルシャッフル集約から誤ったクエリ結果が返される問題が修正されました。[#17845](https://github.com/StarRocks/starrocks/pull/17845)
- クエリが "wait_for_version version: failed: apply stopped" エラーメッセージで失敗する可能性があります。[#17848](https://github.com/StarRocks/starrocks/pull/17850)
- OLAPスキャンのバケツ式が正しくクリアされないため、誤ったクエリ結果が返されます。[#17666](https://github.com/StarRocks/starrocks/pull/17666)
- コロケートグループ内のダイナミックパーティションテーブルのバケット数を変更できず、エラーメッセージが返される問題が修正されました。[#17418](https://github.com/StarRocks/starrocks/pull/17418)
- 非リーダーFEノードに接続してSQLステートメント「USE <catalog_name>.<database_name>」を送信し、「<catalog_name>」が除外された状態で非リーダーFEノードがSQLステートメントを転送し、結果としてリーダーFEノードが「default_catalog」を使用し、指定されたデータベースを見つけられなくなる問題が修正されました。[#17302](https://github.com/StarRocks/starrocks/pull/17302)
- リライト前の辞書マッピングのチェックロジックが正しくない問題が修正されました。[#17405](https://github.com/StarRocks/starrocks/pull/17405)
- FEが時々BEにハートビートを送信し、ハートビート接続がタイムアウトした場合、FEはBEを利用できないと見なし、結果としてBEでトランザクションが失敗する問題が修正されました。[#16386](https://github.com/StarRocks/starrocks/pull/16386)
- 新たにクローンされたタブレットに関するクエリで`get_applied_rowsets`が失敗する問題が修正されました。[#17192](https://github.com/StarRocks/starrocks/pull/17192)
- フォロワーFEで`SET variable = default`を実行するとNPEが発生する問題が修正されました。[#17549](https://github.com/StarRocks/starrocks/pull/17549)
- プロジェクション内の式が辞書によってリライトされない問題が修正されました。[#17558](https://github.com/StarRocks/starrocks/pull/17558)
- 週ごとの基準でダイナミックパーティションを作成するためのロジックが正しくない問題が修正されました。[#17163](https://github.com/StarRocks/starrocks/pull/17163)
- ローカルシャッフルから誤ったクエリ結果が返される問題が修正されました。[#17130](https://github.com/StarRocks/starrocks/pull/17130)
- インクリメンタルクローンに失敗する可能性があります。[#16930](https://github.com/StarRocks/starrocks/pull/16930)
- CBOがいくつかの場合に、2つの演算子が等価かどうかを比較するための誤った論理を使用する可能性があります。[#17199](https://github.com/StarRocks/starrocks/pull/17199) [#17227](https://github.com/StarRocks/starrocks/pull/17227)
- JuiceFSへのアクセスが失敗する問題が修正されました。JuiceFSスキーマが適切にチェックおよび解決されていないためです。[#16940](https://github.com/StarRocks/starrocks/pull/16940)

## 2.4.3

リリース日: 2023年1月19日

### 改善点

- NPEを防ぐために、StarRocksはAnalyze中に対応するデータベースとテーブルが存在するかどうかをチェックします。[#14467](https://github.com/StarRocks/starrocks/pull/14467)
- サポートされていないデータ型を持つ列は外部テーブルのクエリでマテリアライズされません。[#13305](https://github.com/StarRocks/starrocks/pull/13305)
- FE起動スクリプト**start_fe.sh**のJavaバージョンチェックを追加しました。[#14333](https://github.com/StarRocks/starrocks/pull/14333)

### バグ修正

次のバグが修正されています:
- タイムアウトが設定されていない場合、Stream Loadは失敗する可能性があります。[#16241](https://github.com/StarRocks/starrocks/pull/16241)
- メモリ使用量が高い場合、bRPC Sendがクラッシュする可能性があります。[#16046](https://github.com/StarRocks/starrocks/issues/16046)
- StarRocksは、以前のバージョンのStarRocksからの外部テーブルのデータの読み込みに失敗する可能性があります。[#16130](https://github.com/StarRocks/starrocks/pull/16130)
- Materialized viewのリフレッシュの失敗がメモリリークを引き起こす可能性があります。[#16041](https://github.com/StarRocks/starrocks/pull/16041)
- スキーマ変更がPublish段階でハングする場合があります。[#14148](https://github.com/StarRocks/starrocks/issues/14148)
- materialized viewのQeProcessorImplの問題によるメモリリークが発生しています。[#15699](https://github.com/StarRocks/starrocks/pull/15699)
- `limit`を持つクエリの結果が一貫していません。[#13574](https://github.com/StarRocks/starrocks/pull/13574)
- INSERTによるメモリリークが発生しています。[#14718](https://github.com/StarRocks/starrocks/pull/14718)
- Primary KeyテーブルがTablet Migrationを実行します。[#13720](https://github.com/StarRocks/starrocks/pull/13720)
- Broker Load中にBroker Kerberosチケットがタイムアウトすることがあります。[#16149](https://github.com/StarRocks/starrocks/pull/16149)
- テーブルのビューで`nullable`情報が誤って推論されています。[#15744](https://github.com/StarRocks/starrocks/pull/15744)

### 動作変更

- Thrift Listenのデフォルトのバックログを`1024`に変更しました。[#13911](https://github.com/StarRocks/starrocks/pull/13911)
- SQLモード`FORBID_INVALID_DATES`を追加しました。このSQLモードはデフォルトで無効です。有効にすると、StarRocksはDATEタイプの入力を検証し、入力が無効な場合にはエラーを返します。[#14143](https://github.com/StarRocks/starrocks/pull/14143)

## 2.4.2

リリース日: 2022年12月14日

### 改善

- 大量のバケットが存在する場合のBucket Hintのパフォーマンスを最適化しました。[#13142](https://github.com/StarRocks/starrocks/pull/13142)

### バグ修正

以下のバグが修正されました:

- Primary KeyインデックスのFlushがBEをクラッシュさせる可能性があります。[#14857](https://github.com/StarRocks/starrocks/pull/14857) [#14819](https://github.com/StarRocks/starrocks/pull/14819)
- materialized viewのタイプが`SHOW FULL TABLES`で正しく識別されない場合があります。[#13954](https://github.com/StarRocks/starrocks/pull/13954)
- StarRocks v2.2からv2.4にアップグレードすると、BEがクラッシュする場合があります。[#13795](https://github.com/StarRocks/starrocks/pull/13795)
- Broker LoadがBEをクラッシュさせる可能性があります。[#13973](https://github.com/StarRocks/starrocks/pull/13973)
- セッション変数`statistic_collect_parallel`が効果を持たない場合があります。[#14352](https://github.com/StarRocks/starrocks/pull/14352)
- INSERT INTOがBEをクラッシュさせる可能性があります。[#14818](https://github.com/StarRocks/starrocks/pull/14818)
- JAVA UDFがBEをクラッシュさせる可能性があります。[#13947](https://github.com/StarRocks/starrocks/pull/13947)
- 部分更新時のレプリカのクローニングがBEをクラッシュさせ、再起動に失敗する場合があります。[#13683](https://github.com/StarRocks/starrocks/pull/13683)
- Co-located Joinが効果を持たない場合があります。[#13561](https://github.com/StarRocks/starrocks/pull/13561)

### 動作変更

- セッション変数`query_timeout`を上限`259200`、下限`1`で制約しました。
- FEパラメータ`default_storage_medium`を非推奨にしました。テーブルのストレージメディアはシステムによって自動的に推論されます。[#14394](https://github.com/StarRocks/starrocks/pull/14394)

## 2.4.1

リリース日: 2022年11月14日

### 新機能

- LEFT SEMI JOINおよびANTI JOINをサポートする非等価結合をサポートします。JOIN関数が最適化されました。[#13019](https://github.com/StarRocks/starrocks/pull/13019)

### 改善

- `HeartbeatResponse`の`aliveStatus`プロパティをサポートします。`aliveStatus`は、クラスタ内のノードが動作しているかどうかを示します。`aliveStatus`を判定するメカニズムがさらに最適化されました。[#12713](https://github.com/StarRocks/starrocks/pull/12713)
- Routine Loadのエラーメッセージを最適化しました。[#12155](https://github.com/StarRocks/starrocks/pull/12155)

### バグ修正

- v2.4.0RCからv2.4.0にアップグレードした後、BEがクラッシュすることがあります。[#13128](https://github.com/StarRocks/starrocks/pull/13128)
- 遅延マテリアリゼーションにより、データレイク上のクエリが不正な結果を返すことがあります。[#13133](https://github.com/StarRocks/starrocks/pull/13133)
- `get_json_int`関数が例外をスローすることがあります。[#12997](https://github.com/StarRocks/starrocks/pull/12997)
- PRIMARY KEYテーブルからのデータ削除後、データが一貫していない場合があります。[#12719](https://github.com/StarRocks/starrocks/pull/12719)
- PRIMARY KEYテーブルのコンパクション中にBEがクラッシュする場合があります。[#12914](https://github.com/StarRocks/starrocks/pull/12914)
- 入力が空の場合、json_object関数が不正な結果を返すことがあります。[#13030](https://github.com/StarRocks/starrocks/issues/13030)
- `RuntimeFilter`によってBEがクラッシュすることがあります。[#12807](https://github.com/StarRocks/starrocks/pull/12807)
- CBOで過度な再帰的な計算が原因でFEがハングすることがあります。[#12788](https://github.com/StarRocks/starrocks/pull/12788)
- グレースフルに終了する際にBEがクラッシュしたりエラーを報告することがあります。[#12852](https://github.com/StarRocks/starrocks/pull/12852)
- テーブルに新しいカラムが追加された後にデータが削除されるとコンパクションがクラッシュすることがあります。[#12907](https://github.com/StarRocks/starrocks/pull/12907)
- OLAP外部テーブルメタデータ同期のメカニズムが正しくないため、データが一貫していない場合があります。[#12368](https://github.com/StarRocks/starrocks/pull/12368)
- 1つのBEがクラッシュすると、他のBEはタイムアウトまで関連するクエリを実行することがあります。[#12954](https://github.com/StarRocks/starrocks/pull/12954)

### 動作変更

- Hive外部テーブルの解析に失敗する場合、StarRocksは関連する列をNULL列に変換する代わりにエラーメッセージをスローします。[#12382](https://github.com/StarRocks/starrocks/pull/12382)

## 2.4.0

リリース日: 2022年10月20日

### 新機能

- 複数の基本テーブルに基づく非同期マテリアライズドビューの作成をサポートして、JOIN操作を高速化します。非同期マテリアライズドビューはすべての[table types](../table_design/table_types/table_types.md)をサポートしています。詳細については、[Materialized View](../using_starrocks/Materialized_view.md)を参照してください。
- INSERT OVERWRITEを使用してデータを上書きできるようになりました。詳細については、[Load data using INSERT](../loading/InsertInto.md)を参照してください。
- [プレビュー]状態レスのCompute Nodes (CN)をサポートし、水平スケールが可能です。StarRocks Operatorを使用して、CNをKubernetes (K8s) クラスタに展開して、自動的な水平スケーリングを実現できます。詳細については、[StarRocks OperatorでKubernetes上のCNを展開および管理](../deployment/sr_operator.md)を参照してください。
- OUTER JOINが、`<`, `<=`, `>`, `>=`, `<>`などの比較演算子によって関連付けられる非等価な結合をサポートします。詳細については、[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)を参照してください。
- Apache IcebergおよびApache Hudiのデータは、IcebergカタログおよびHudiカタログの作成をサポートします。詳細については、[Iceberg catalog](../data_source/catalog/iceberg_catalog.md)および[Hudi catalog](../data_source/catalog/hudi_catalog.md)を参照してください。
- CSV形式でのApache Hive™テーブルのARRAY型列のクエリをサポートします。詳細については、[External table](../data_source/External_table.md)を参照してください。
- DESCを使用して外部データのスキーマを表示することをサポートします。詳細については、[DESC](../sql-reference/sql-statements/Utility/DESCRIBE.md)を参照してください。
- 特定の役割またはIMPERSOANATE権限をGRANTを使用してユーザーに付与し、REVOKEを使用してそれらを取り消すことをサポートし、IMPERSOANATE権限でSQLステートメントを実行するEXECUTE ASもサポートしています。詳細については、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)、および[EXECUTE AS](../sql-reference/sql-statements/account-management/EXECUTE_AS.md)を参照してください。

- FQDNアクセスをサポート: 現在、ドメイン名またはホスト名とポートの組み合わせをBEまたはFEノードの一意の識別子として使用できます。これにより、IPアドレスの変更によるアクセスの失敗を防ぐことができます。詳細については、[FQDNアクセスの有効化](../administration/enable_fqdn.md)を参照してください。

- flink-connector-starrocksは、プライマリキー表の部分更新をサポートしています。詳細については、[flink-connector-starrocksを使用してデータをロードする](../loading/Flink-connector-starrocks.md)を参照してください。

- 次の新しい関数を提供します:

  - array_contains_all: 特定の配列が別の配列のサブセットであるかどうかをチェックします。詳細については、[array_contains_all](../sql-reference/sql-functions/array-functions/array_contains_all.md)を参照してください。
  - percentile_cont: 線形補間を使用してパーセンタイル値を計算します。詳細については、[percentile_cont](../sql-reference/sql-functions/aggregate-functions/percentile_cont.md)を参照してください。

### 改善

- プライマリキー表は、VARCHARタイプのプライマリキーインデックスをディスクにフラッシュできるようにサポートされています。 バージョン2.4.0以降、プライマリキー表は、永続的なプライマリキーインデックスがオンになっているかどうかに関係なく同じデータ型をプライマリキーインデックスにサポートしています。

- 外部テーブルのクエリパフォーマンスが最適化されています。

  - Parquet形式の外部テーブルでのクエリ中に遅延素材化をサポートし、小規模のフィルタリングが含まれるデータレイクのクエリパフォーマンスを最適化します。
  - 小規模なI/O操作をマージして、データレイクのクエリ遅延を減少させ、外部テーブルのクエリパフォーマンスを向上させます。

- ウィンドウ関数のパフォーマンスが最適化されています。

- Cross Joinのパフォーマンスが最適化され、述語のプッシュダウンをサポートしています。

- CBO統計にヒストグラムが追加されました。完全な統計の収集がさらに最適化されています。詳細については、[CBO統計の収集](../using_starrocks/Cost_based_optimizer.md)を参照してください。

- タブレットスキャンのために適応的なマルチスレッディングが有効になり、スキャンパフォーマンスがタブレット数に依存しないようになりました。その結果、バケツの数をより簡単に設定できます。詳細については、[バケットの数を決定する方法](../table_design/Data_distribution.md#how-to-determine-the-number-of-buckets)を参照してください。

- Apache Hiveで圧縮されたTXTファイルのクエリをサポートしています。

- デフォルトのPageCacheサイズの計算とメモリの整合性チェックのメカニズムを調整して、マルチインスタンス展開中のOOMの問題を避けるようにしました。

- プライマリキー表の大容量バッチロードのパフォーマンスを2倍に向上させるために、final_merge操作を削除することで改善しました。

- 外部システム（Apache Flink®およびApache Kafka®）からのデータをロードする際に、2相コミット（2PC）を実装するためのStream Loadトランザクションインターフェースをサポートします。これにより、高い並行ストリームロードのパフォーマンスが向上します。

- 関数:

  - 1つの文で複数のCOUNT(DISTINCT)を使用できます。詳細については、[count](../sql-reference/sql-functions/aggregate-functions/count.md)を参照してください。
  - ウィンドウ関数min()およびmax()はスライディングウィンドウをサポートします。詳細については、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)を参照してください。
  - window_funnel関数のパフォーマンスが最適化されました。詳細については、[window_funnel](../sql-reference/sql-functions/aggregate-functions/window_funnel.md)を参照してください。

### バグ修正

以下のバグが修正されました:

- DESCによって返されるDECIMALデータ型が、CREATE TABLEステートメントで指定されているものと異なる。 [#7309](https://github.com/StarRocks/starrocks/pull/7309)

- FEメタデータ管理に関連する問題により、FEの安定性に影響を与えました。[#6685](https://github.com/StarRocks/starrocks/pull/6685) [#9445](https://github.com/StarRocks/starrocks/pull/9445) [#7974](https://github.com/StarRocks/starrocks/pull/7974) [#7455](https://github.com/StarRocks/starrocks/pull/7455)

- データロード関連の問題:

  - ARRAY列が指定されている場合にLoadが失敗する問題が解消されました。[#9158](https://github.com/StarRocks/starrocks/pull/9158)
  - ブローカーロードを介してデータをロードした後に、非重複キー表にデータをロードした場合にレプリカが一貫性がなくなる問題が解消されました。[#8714](https://github.com/StarRocks/starrocks/pull/8714)
  - ALTER ROUTINE LOADを実行するとNPEが発生する問題が解消されました。[#7804](https://github.com/StarRocks/starrocks/pull/7804)

- データレイクの分析関連の問題:

  - Hive外部テーブルのParquetデータのクエリが失敗する問題が解消されました。[#7413](https://github.com/StarRocks/starrocks/pull/7413) [#7482](https://github.com/StarRocks/starrocks/pull/7482) [#7624](https://github.com/StarRocks/starrocks/pull/7624)
  - Elasticsearch外部テーブルで`limit`句を含むクエリの結果が間違って返される問題が解消されました。[#9226](https://github.com/StarRocks/starrocks/pull/9226)
  - 複雑なデータ型を持つApache Icebergテーブルでクエリ中に不明なエラーが発生する問題が解消されました。[#11298](https://github.com/StarRocks/starrocks/pull/11298)

- メタデータがリーダーFEとフォローアーFEノード間で一貫性がない問題が解消されました。[#11215](https://github.com/StarRocks/starrocks/pull/11215)

- BITMAPデータのサイズが2GBを超えるとBEがクラッシュする問題が解消されました。[#11178](https://github.com/StarRocks/starrocks/pull/11178)

### 動作の変更

- デフォルトでPage Cacheが有効になりました（"disable_storage_page_cache" = "false"）。 デフォルトのキャッシュサイズ（`storage_page_cache_limit`）はシステムメモリの20%です。
- デフォルトでCBOが有効になりました。セッション変数`enable_cbo`は非推奨となりました。
- デフォルトでベクトル化エンジンが有効になりました。セッション変数`vectorized_engine_enable`は非推奨となりました。

### その他

- Resource Groupの安定したリリースを発表します。
- JSONデータ型および関連する機能の安定したリリースを発表します。