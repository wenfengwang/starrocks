---
displayed_sidebar: English
---

# StarRocks バージョン 3.1

## 3.1.6

リリース日: 2023年12月18日

### 新機能

- 指定された秒の小数部の精度（マイクロ秒までの精度）で現在の日付と時刻を返す[now(p)関数](https://docs.starrocks.io/zh/docs/3.2/sql-reference/sql-functions/date-time-functions/now/)を追加しました。`p`が指定されていない場合、この関数は秒単位の精度で日付と時刻のみを返します。[#36676](https://github.com/StarRocks/starrocks/pull/36676)
- 行セットの最大許容数を設定する新しいメトリック`max_tablet_rowset_num`を追加しました。このメトリックは、コンパクションの問題を検出し、「バージョンが多すぎます」というエラーの発生を減らすのに役立ちます。[#36539](https://github.com/StarRocks/starrocks/pull/36539)
- コマンドラインツールを使用してヒーププロファイルを取得する機能をサポートし、トラブルシューティングが容易になります。[#35322](https://github.com/StarRocks/starrocks/pull/35322)
- 共通テーブル式（CTE）を使用した非同期マテリアライズドビューの作成をサポートします。[#36142](https://github.com/StarRocks/starrocks/pull/36142)
- 以下のビットマップ関数を追加しました：[subdivide_bitmap](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/subdivide_bitmap/)、[bitmap_from_binary](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/bitmap_from_binary/)、および[bitmap_to_binary](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/bitmap_to_binary/)。[#35817](https://github.com/StarRocks/starrocks/pull/35817) [#35621](https://github.com/StarRocks/starrocks/pull/35621)

### パラメータ変更

- FE動的パラメータ`enable_new_publish_mechanism`が静的パラメータに変更されました。パラメータ設定を変更した後はFEを再起動する必要があります。[#35338](https://github.com/StarRocks/starrocks/pull/35338)
- ごみ箱ファイルのデフォルト保持期間が元の3日から1日に変更されました。[#37113](https://github.com/StarRocks/starrocks/pull/37113)
- 新しいFE設定項目`routine_load_unstable_threshold_second`が追加されました。[#36222](https://github.com/StarRocks/starrocks/pull/36222)
- 新しいBE設定項目`enable_stream_load_verbose_log`が追加されました。デフォルト値は`false`です。このパラメータを`true`に設定すると、StarRocksはStream LoadジョブのHTTPリクエストとレスポンスを記録でき、トラブルシューティングが容易になります。[#36113](https://github.com/StarRocks/starrocks/pull/36113)
- 新しいBE設定項目`enable_lazy_delta_column_compaction`が追加されました。デフォルト値は`true`で、StarRocksがデルタ列に対して頻繁なコンパクション操作を行わないことを示します。[#36654](https://github.com/StarRocks/starrocks/pull/36654)

### 改善点

- セッション変数[sql_mode](https://docs.starrocks.io/zh/docs/reference/System_variable/#sql_mode)に新しい値オプション`GROUP_CONCAT_LEGACY`が追加され、v2.5より前のバージョンの[group_concat関数](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/string-functions/group_concat/)の実装ロジックとの互換性を提供します。[#36150](https://github.com/StarRocks/starrocks/pull/36150)
- [SHOW DATA](https://docs.starrocks.io/zh/docs/sql-reference/sql-statements/data-manipulation/SHOW_DATA/)ステートメントによって返されるプライマリキーテーブルのサイズには、**.cols**ファイル（部分的な列の更新と生成された列に関連するファイル）と永続インデックスファイルのサイズが含まれます。[#34898](https://github.com/StarRocks/starrocks/pull/34898)
- MySQL外部テーブルおよびJDBCカタログ内の外部テーブルに対するクエリは、WHERE句にキーワードを含むことをサポートします。[#35917](https://github.com/StarRocks/starrocks/pull/35917)
- プラグインの読み込みに失敗しても、エラーやFEの起動失敗は発生しなくなりました。代わりに、FEは正常に起動し、プラグインのエラーステータスは[SHOW PLUGINS](https://docs.starrocks.io/docs/sql-reference/sql-statements/Administration/SHOW_PLUGINS/)を使用して照会できます。[#36566](https://github.com/StarRocks/starrocks/pull/36566)
- 動的パーティショニングはランダム分散をサポートします。[#35513](https://github.com/StarRocks/starrocks/pull/35513)
- [SHOW ROUTINE LOAD](https://docs.starrocks.io/zh/docs/sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD/)ステートメントによって返される結果には、最後に失敗したタスクに関する情報を示す新しいフィールド`OtherMsg`が追加されました。[#35806](https://github.com/StarRocks/starrocks/pull/35806)
- Broker LoadジョブでのAWS S3の認証情報`aws.s3.access_key`と`aws.s3.access_secret`は監査ログに表示されなくなりました。[#36571](https://github.com/StarRocks/starrocks/pull/36571)
- `information_schema`データベースの`be_tablets`ビューには、永続インデックスのディスク使用量（バイト単位）を記録する新しいフィールド`INDEX_DISK`が追加されました。[#35615](https://github.com/StarRocks/starrocks/pull/35615)

### バグ修正

以下の問題を修正しました：

- データが破損している場合にユーザーが永続インデックスを作成すると、BEがクラッシュする問題。[#30841](https://github.com/StarRocks/starrocks/pull/30841)
- ユーザーが入れ子になったクエリを含む非同期マテリアライズドビューを作成すると、「パーティション列の解決に失敗しました」というエラーが報告される問題。[#26078](https://github.com/StarRocks/starrocks/issues/26078)
- データが破損しているベーステーブルに対してユーザーが非同期マテリアライズドビューを作成すると、「予期しない例外: null」というエラーが報告される問題。[#30038](https://github.com/StarRocks/starrocks/pull/30038)
- ユーザーがウィンドウ関数を含むクエリを実行すると、「[1064] [42000]: 定数列の行数が上限に達しました: 4294967296」というSQLエラーが報告される問題。[#33561](https://github.com/StarRocks/starrocks/pull/33561)
- FE設定項目`enable_collect_query_detail_info`を`true`に設定するとFEのパフォーマンスが大幅に低下する問題。[#35945](https://github.com/StarRocks/starrocks/pull/35945)
- StarRocksの共有データモードで、ユーザーがオブジェクトストレージからファイルを削除しようとすると「リクエストレートを減らしてください」というエラーが報告される問題。[#35566](https://github.com/StarRocks/starrocks/pull/35566)
- ユーザーがマテリアライズドビューをリフレッシュする際にデッドロックが発生する問題。[#35736](https://github.com/StarRocks/starrocks/pull/35736)
- DISTINCTウィンドウ演算子のプッシュダウン機能が有効になっている場合、ウィンドウ関数によって計算された列の複雑な式に対してSELECT DISTINCT操作を実行するとエラーが報告される問題。[#36357](https://github.com/StarRocks/starrocks/pull/36357)
- ソースデータファイルがORC形式であり、入れ子になった配列を含む場合にBEがクラッシュする問題。[#36127](https://github.com/StarRocks/starrocks/pull/36127)
- 一部のS3互換オブジェクトストレージが重複ファイルを返すためにBEがクラッシュする問題。[#36103](https://github.com/StarRocks/starrocks/pull/36103)

## 3.1.5

リリース日: 2023年11月28日

### 新機能

- StarRocksの共有データクラスタのCNノードがデータエクスポートをサポートするようになりました。[#34018](https://github.com/StarRocks/starrocks/pull/34018)

### 改善点

- システムデータベース`INFORMATION_SCHEMA`の[`COLUMNS`](../reference/information_schema/columns.md)ビューがARRAY、MAP、およびSTRUCT列を表示できるようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- LZOで圧縮され[Hive](../data_source/catalog/hive_catalog.md)に格納されているParquet、ORC、およびCSV形式のファイルに対するクエリをサポートします。[#30923](https://github.com/StarRocks/starrocks/pull/30923) [#30721](https://github.com/StarRocks/starrocks/pull/30721)
- 自動的にパーティション分割されたテーブルの指定されたパーティションへの更新をサポートします。指定されたパーティションが存在しない場合はエラーが返されます。[#34777](https://github.com/StarRocks/starrocks/pull/34777)
- マテリアライズドビューが作成されたテーブルとビュー（これらのビューに関連付けられている他のテーブルとマテリアライズドビューを含む）に対してスワップ、ドロップ、またはスキーマ変更操作が実行された場合のマテリアライズドビューの自動更新をサポートします。[#32829](https://github.com/StarRocks/starrocks/pull/32829)
- 以下を含む一部のビットマップ関連操作のパフォーマンスを最適化しました。

  - ネストされたループ結合を最適化しました。[#340804](https://github.com/StarRocks/starrocks/pull/34804) [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor` 関数を最適化しました。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - Copy on Writeをサポートして、Bitmapのパフォーマンスを最適化し、メモリ消費を削減します。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### バグ修正

以下の問題を修正しました：

- Broker Loadジョブでフィルタリング条件が指定されている場合、特定の状況でデータロード中にBEがクラッシュすることがあります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTSを実行すると未知のエラーが報告されます。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 式ベースの自動パーティショニングを使用するテーブルにデータをロードする際、「Error: The row create partition failed since Runtime error: failed to analyse partition value」というエラーが発生することがあります。[#33513](https://github.com/StarRocks/starrocks/pull/33513)
- クエリに対して「get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction」というエラーが返されます。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- StarRocksのシェアードナッシングクラスターでは、IcebergまたはHiveテーブルに対するクエリがBEをクラッシュさせる可能性があります。[#34682](https://github.com/StarRocks/starrocks/pull/34682)
- StarRocksのシェアードナッシングクラスターで、データロード中に複数のパーティションが自動的に作成されると、ロードされたデータが時々不適切なパーティションに書き込まれることがあります。[#34731](https://github.com/StarRocks/starrocks/pull/34731)
- 永続的なインデックスが有効な主キーテーブルに長時間頻繁にデータをロードすると、BEがクラッシュすることがあります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリに対して「Exception: java.lang.IllegalStateException: null」というエラーが返されます。[#33535](https://github.com/StarRocks/starrocks/pull/33535)
- `show proc '/current_queries';`を実行中にクエリが開始されると、BEがクラッシュすることがあります。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 永続的なインデックスが有効な主キーテーブルに大量のデータをロードするとエラーが発生することがあります。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- StarRocksをv2.4以前のバージョンからアップグレードした後、コンパクションスコアが予期せず上昇することがあります。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- データベースドライバMariaDB ODBCを使用して`INFORMATION_SCHEMA`をクエリすると、`schemata`ビューの`CATALOG_NAME`列に`null`値のみが返されます。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- 異常なデータがロードされたためにFEがクラッシュし、再起動できなくなります。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- Stream Loadジョブが**PREPARED**状態にある間にスキーマ変更が実行されていると、ジョブによってロードされるべきソースデータの一部が失われます。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- HDFSストレージパスの末尾にスラッシュ(`/`)を2つ以上含めると、HDFSからのデータのバックアップと復元が失敗します。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- セッション変数`enable_load_profile`を`true`に設定すると、Stream Loadジョブが失敗しやすくなります。[#34544](https://github.com/StarRocks/starrocks/pull/34544)
- 主キーテーブルに対してカラムモードで部分更新を実行すると、テーブルの一部のタブレットでレプリカ間のデータ不整合が発生することがあります。[#34555](https://github.com/StarRocks/starrocks/pull/34555)
- ALTER TABLEステートメントで追加された`partition_live_number`プロパティは効果がありません。[#34842](https://github.com/StarRocks/starrocks/pull/34842)
- FEが「failed to load journal type 118」というエラーを報告して起動に失敗します。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- FEパラメータ`recover_with_empty_tablet`を`true`に設定すると、FEがクラッシュする可能性があります。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- レプリカ操作のリプレイに失敗すると、FEがクラッシュすることがあります。[#32295](https://github.com/StarRocks/starrocks/pull/32295)

### 互換性の変更

#### パラメータ

- FE構成項目[`enable_statistics_collect_profile`](../administration/FE_configuration.md#enable_statistics_collect_profile)が追加され、統計クエリのプロファイルを生成するかどうかを制御します。デフォルト値は`false`です。[#33815](https://github.com/StarRocks/starrocks/pull/33815)
- FE構成項目[`mysql_server_version`](../administration/FE_configuration.md#mysql_server_version)が変更可能になり、新しい設定はFEの再起動を必要とせずに現在のセッションで有効になります。[#34033](https://github.com/StarRocks/starrocks/pull/34033)
- BE/CN構成項目[`update_compaction_ratio_threshold`](../administration/BE_configuration.md#update_compaction_ratio_threshold)が追加され、StarRocks共有データクラスタ内の主キーテーブルに対してコンパクションがマージできるデータの最大割合を制御します。デフォルト値は`0.5`です。単一のタブレットが過度に大きくなる場合は、この値を小さくすることをお勧めします。StarRocksシェアードナッシングクラスタでは、主キーテーブルのコンパクションでマージできるデータの割合は自動的に調整されます。[#35129](https://github.com/StarRocks/starrocks/pull/35129)

#### システム変数

- セッション変数`cbo_decimal_cast_string_strict`が追加され、CBOがDECIMAL型からSTRING型へのデータ変換をどのように行うかを制御します。この変数を`true`に設定すると、v2.5.x以降のバージョンで組み込まれたロジックが優先され、システムは厳密な変換を実装します（つまり、生成された文字列は切り捨てられ、スケールの長さに基づいて0が埋められます）。この変数を`false`に設定すると、v2.5.xより前のバージョンで組み込まれたロジックが優先され、システムはすべての有効な数字を処理して文字列を生成します。デフォルト値は`true`です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- セッション変数`cbo_eq_base_type`が追加され、DECIMAL型データとSTRING型データ間の比較に使用するデータ型を指定します。デフォルト値は`VARCHAR`で、`DECIMAL`も有効な値です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- セッション変数`big_query_profile_second_threshold`が追加されました。セッション変数[`enable_profile`](../reference/System_variable.md#enable_profile)が`false`に設定されており、クエリの実行時間が`big_query_profile_second_threshold`変数で指定されたしきい値を超える場合、そのクエリのプロファイルが生成されます。[#33825](https://github.com/StarRocks/starrocks/pull/33825)

## 3.1.4

リリース日：2023年11月2日

### 新機能

- 共有データStarRocksクラスタで作成された主キーテーブルのソートキーをサポートします。
- `str2date`関数を使用して、非同期マテリアライズドビューのパーティション式を指定することをサポートします。これにより、外部カタログに存在し、STRING型データをパーティション式として使用するテーブルに作成された非同期マテリアライズドビューの増分更新とクエリ書き換えが容易になります。[#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- 新しいセッション変数`enable_query_tablet_affinity`が追加され、同じタブレットに対する複数のクエリを固定レプリカに向けるかどうかを制御します。このセッション変数はデフォルトで`false`に設定されています。[#33049](https://github.com/StarRocks/starrocks/pull/33049)
- ユーティリティ関数`is_role_in_session`が追加され、指定されたロールが現在のセッションでアクティブ化されているかどうかを確認するために使用されます。ユーザーに付与されたネストされたロールのチェックをサポートします。[#32984](https://github.com/StarRocks/starrocks/pull/32984)
- グローバル変数 `enable_group_level_query_queue` (デフォルト値: `false`) によって制御されるリソースグループレベルのクエリキューの設定をサポートします。グローバルレベルまたはリソースグループレベルのリソース消費が事前定義されたしきい値に達すると、新しいクエリがキューに入れられ、グローバルレベルのリソース消費とリソースグループレベルのリソース消費の両方がしきい値を下回ったときに実行されます。
  - ユーザーは、各リソースグループの `concurrency_limit` を設定して、BEごとに許可される同時クエリの最大数を制限できます。
  - ユーザーは、各リソースグループの `max_cpu_cores` を設定して、BEごとに許可される最大CPU消費量を制限できます。
- リソースグループ分類子用の2つのパラメーター、`plan_cpu_cost_range` と `plan_mem_cost_range` が追加されました。
  - `plan_cpu_cost_range`: システムによって推定されたCPU消費範囲。デフォルト値 `NULL` は制限が課されないことを示します。
  - `plan_mem_cost_range`: システムによって推定されたメモリ消費範囲。デフォルト値 `NULL` は制限が課されないことを示します。

### 改善点

- ウィンドウ関数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、および STDDEV_SAMP が ORDER BY 句とウィンドウ句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型データに対するクエリ中に10進数のオーバーフローが発生した場合、NULLではなくエラーが返されます。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- クエリキューで許可される同時クエリの数は、リーダーFEが管理します。各フォロワーFEは、クエリの開始と終了時にリーダーFEに通知します。同時クエリの数がグローバルレベルまたはリソースグループレベルの `concurrency_limit` に達すると、新しいクエリは拒否されるかキューに入れられます。

### バグ修正

以下の問題を修正しました：

- SparkまたはFlinkが、メモリ使用量の統計が不正確なためにデータ読み取りエラーを報告することがあります。[#30702](https://github.com/StarRocks/starrocks/pull/30702)  [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- メタデータキャッシュのメモリ使用量の統計が不正確です。[#31978](https://github.com/StarRocks/starrocks/pull/31978)
- libcurlが呼び出されたときにBEがクラッシュする問題。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- Hiveビューに基づいて作成されたStarRocksのマテリアライズドビューを更新すると、「java.lang.ClassCastException: com.starrocks.catalog.HiveView cannot be cast to com.starrocks.catalog.HiveMetaStoreTable」というエラーが返される問題。[#31004](https://github.com/StarRocks/starrocks/pull/31004)
- ORDER BY句に集約関数が含まれている場合、「java.lang.IllegalStateException: null」というエラーが返される問題。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 共有データStarRocksクラスタで、`information_schema.COLUMNS`にテーブルキーの情報が記録されていないため、Flink Connectorを使用してデータをロードした際にDELETE操作が実行できない問題。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- Flink Connectorを使用してデータをロードする際に、高度に同時実行されるロードジョブがあり、HTTPスレッド数とスキャンスレッド数が上限に達した場合にロードジョブが予期せず中断される問題。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 数バイトのフィールドのみを追加した後、データ変更が完了する前にSELECT COUNT(*)を実行すると「error: invalid field name」というエラーが返される問題。[#33243](https://github.com/StarRocks/starrocks/pull/33243)
- クエリキャッシュを有効にした後にクエリ結果が不正確になる問題。[#32781](https://github.com/StarRocks/starrocks/pull/32781)
- ハッシュ結合中にクエリが失敗し、BEがクラッシュする問題。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- `DATA_TYPE` および `COLUMN_TYPE` が BINARY または VARBINARY データ型の場合、`information_schema.columns` ビューで `unknown` と表示される問題。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 動作変更

- v3.1.4以降、新しいStarRocksクラスタで作成されたプライマリキーテーブルに対して、永続的なインデックス作成がデフォルトで有効になります（これは、以前のバージョンからv3.1.4にアップグレードされた既存のStarRocksクラスタには適用されません）。[#33374](https://github.com/StarRocks/starrocks/pull/33374)
- デフォルトで `true` に設定されている新しいFEパラメータ `enable_sync_publish` が追加されました。このパラメータが `true` に設定されている場合、プライマリキーテーブルへのデータロードのパブリッシュフェーズは、アプライタスクが完了した後にのみ実行結果を返します。そのため、ロードジョブが成功メッセージを返した直後にロードされたデータをクエリできます。ただし、このパラメータを `true` に設定すると、プライマリキーテーブルへのデータロードに時間がかかる場合があります（このパラメータが追加される前は、アプライタスクはパブリッシュフェーズと非同期でした）。[#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

リリース日: 2023年9月25日

### 新機能

- 共有データStarRocksクラスタで作成されたプライマリキーテーブルは、シェアードナッシングStarRocksクラスタと同様に、ローカルディスクへのインデックスの永続化をサポートします。
- 集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) が DISTINCT キーワードと ORDER BY 句をサポートします。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Kafka Connector](../loading/Kafka-connector-starrocks.md)、[Flink Connector](../loading/Flink-connector-starrocks.md)、および [Spark Connector](../loading/Spark-connector-starrocks.md) が、プライマリキーテーブルの列モードでの部分更新をサポートします。[#28288](https://github.com/StarRocks/starrocks/pull/28288)
- パーティション内のデータは、時間の経過とともに自動的にクールダウンすることができます（この機能は [リストパーティショニング](../table_design/list_partitioning.md) にはサポートされていません）。[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改善点

無効なコメントを含むSQLコマンドを実行すると、MySQLと一致する結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)

### バグ修正

以下の問題を修正しました：

- DELETE ステートメントの WHERE 句で [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) または [HLL](../sql-reference/sql-statements/data-types/HLL.md) データ型が指定されている場合、ステートメントを正しく実行できない問題。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- フォロワーFEが再起動された後、CpuCores統計が最新でなくなり、クエリパフォーマンスが低下する問題。[#28472](https://github.com/StarRocks/starrocks/pull/28472) [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- [to_bitmap()](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 関数の実行コストが誤って計算され、マテリアライズドビューが書き換えられた後に関数に対して不適切な実行プランが選択される問題。[#29961](https://github.com/StarRocks/starrocks/pull/29961)
- 共有データアーキテクチャの特定のユースケースで、フォロワーFEが再起動された後、フォロワーFEに送信されたクエリが「Backend node not found. Check if any backend node is down」というエラーを返す問題。[#28615](https://github.com/StarRocks/starrocks/pull/28615)
- [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して変更されているテーブルにデータが継続的にロードされると、「Tablet is in error state」というエラーが発生する問題。[#29364](https://github.com/StarRocks/starrocks/pull/29364)
- `ADMIN SET FRONTEND CONFIG` コマンドを使用してFEダイナミックパラメータ `max_broker_load_job_concurrency` を変更しても効果がない問題。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- [date_diff()](../sql-reference/sql-functions/date-time-functions/date_diff.md)関数で時間単位が定数であるが日付が定数でない場合、BEはクラッシュします。[#29937](https://github.com/StarRocks/starrocks/issues/29937)
- 共有データアーキテクチャでは、非同期ロードが有効になった後、自動パーティショニングは機能しません。[#29986](https://github.com/StarRocks/starrocks/issues/29986)
- ユーザーが[CREATE TABLE LIKE](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md)ステートメントを使用してプライマリキーテーブルを作成する場合、エラー`Unexpected exception: Unknown properties: {persistent_index_type=LOCAL}`が発生します。[#30255](https://github.com/StarRocks/starrocks/pull/30255)
- BEを再起動した後にプライマリキーテーブルを復元すると、メタデータの不整合が発生します。[#30135](https://github.com/StarRocks/starrocks/pull/30135)
- ユーザーがトランケート操作とクエリが同時に実行されるプライマリキーテーブルにデータをロードする場合、特定のケースでエラー「java.lang.NullPointerException」が発生することがあります。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- マテリアライズドビューの作成ステートメントで述語式が指定されている場合、そのマテリアライズドビューのリフレッシュ結果が不正確です。[#29904](https://github.com/StarRocks/starrocks/pull/29904)
- ユーザーがStarRocksクラスターをv3.1.2にアップグレードした後、アップグレード前に作成されたテーブルのストレージボリュームプロパティが`null`にリセットされます。[#30647](https://github.com/StarRocks/starrocks/pull/30647)
- タブレットメタデータに対してチェックポイントと復元が同時に実行されると、一部のタブレットレプリカが失われ、回復できなくなります。[#30603](https://github.com/StarRocks/starrocks/pull/30603)
- ユーザーがCloudCanalを使用して`NOT NULL`でデフォルト値が指定されていないテーブルカラムにデータをロードすると、エラー「Unsupported dataFormat value is : \N」が発生します。[#30799](https://github.com/StarRocks/starrocks/pull/30799)

### 動作変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)関数を使用する際、ユーザーはSEPARATORキーワードを使用して区切り文字を宣言する必要があります。
- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)関数によって返される文字列のデフォルト最大長を制御するセッション変数[`group_concat_max_len`](../reference/System_variable.md#group_concat_max_len)のデフォルト値が無制限から`1024`に変更されました。

## 3.1.2

リリース日: 2023年8月25日

### バグ修正

以下の問題を修正しました：

- ユーザーがデフォルトで接続するデータベースを指定し、そのデータベース内のテーブルにのみ権限を持ち、データベース自体には権限を持っていない場合、データベースに対する権限がないというエラーが表示されます。[#29767](https://github.com/StarRocks/starrocks/pull/29767)
- クラウドネイティブテーブルの`show_data` RESTful APIアクションで返される値が正しくありません。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- [array_agg()](../sql-reference/sql-functions/array-functions/array_agg.md)関数が実行中にクエリがキャンセルされると、BEがクラッシュします。[#29400](https://github.com/StarRocks/starrocks/issues/29400)
- [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md)ステートメントで返される`Default`フィールドの値が、BITMAPまたはHLLデータ型の列に対して不正確です。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- クエリで[array_map()](../sql-reference/sql-functions/array-functions/array_map.md)関数が複数のテーブルを含む場合、プッシュダウン戦略の問題によりクエリが失敗します。[#29504](https://github.com/StarRocks/starrocks/pull/29504)
- Apache ORCのバグ修正ORC-1304([apache/orc#1299](https://github.com/apache/orc/pull/1299))がマージされていないため、ORC形式のファイルに対するクエリが失敗します。[#29804](https://github.com/StarRocks/starrocks/pull/29804)

### 動作変更

新しくデプロイされたStarRocks v3.1クラスタでは、[SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を実行してそのカタログに切り替えるためには、宛先の外部カタログに対するUSAGE権限が必要です。[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)を使用して必要な権限を付与することができます。

以前のバージョンからアップグレードされたv3.1クラスタでは、継承された権限でSET CATALOGを実行できます。

## 3.1.1

リリース日: 2023年8月18日

### 新機能

- [共有データクラスタ](../deployment/shared_data/s3.md)でAzure Blob Storageをサポート。
- [共有データクラスタ](../deployment/shared_data/s3.md)でリストパーティショニングをサポート。
- 集約関数[COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)をサポート。
- [ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMPをサポート。

### 改善点

すべての複合述語とWHERE句のすべての式に対して暗黙の型変換をサポートします。セッション変数[`enable_strict_type`](../reference/System_variable.md)を使用して暗黙の型変換を有効または無効にすることができます。このセッション変数のデフォルト値は`false`です。

### バグ修正

以下の問題を修正しました：

- 複数のレプリカを持つテーブルにデータがロードされると、テーブルの一部のパーティションが空の場合に多数の無効なログレコードが書き込まれます。[#28824](https://github.com/StarRocks/starrocks/issues/28824)
- 平均行サイズの見積もりが不正確であるため、プライマリキーテーブルの列モードでの部分更新が過度に大きなメモリを消費します。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- ERROR状態のタブレットに対してクローン操作がトリガされると、ディスク使用量が増加します。[#28488](https://github.com/StarRocks/starrocks/pull/28488)
- コンパクションが行われると、コールドデータがローカルキャッシュに書き込まれます。[#28831](https://github.com/StarRocks/starrocks/pull/28831)

## 3.1.0

リリース日: 2023年8月7日

### 新機能

#### 共有データクラスタ

- 永続インデックスを有効にできないプライマリキーテーブルをサポートしました。
- [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)列属性をサポートし、各データ行にグローバルに一意のIDを提供し、データ管理を簡素化します。
- ロード中にパーティションを自動的に作成し、パーティション式を使用してパーティションルールを定義することをサポートし、パーティション作成をより使いやすく柔軟にします。[詳細](../table_design/expression_partitioning.md)
- ストレージボリュームの抽象化をサポートし、共有データStarRocksクラスタでストレージの場所と認証情報を設定できます。ユーザーはデータベースやテーブルを作成する際に既存のストレージボリュームを直接参照でき、認証設定が容易になります。[詳細](../deployment/shared_data/s3.md#use-your-shared-data-starrocks-cluster)

#### データレイクアナリティクス

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)内のテーブルで作成されたビューにアクセスすることをサポートします。
- Parquet形式のIceberg v2テーブルにアクセスすることをサポートします。
- Parquet形式のIcebergテーブルへのデータシンクをサポートします。[詳細](../data_source/catalog/iceberg_catalog.md#sink-data-to-an-iceberg-table)
- [プレビュー] [Elasticsearchカタログ](../data_source/catalog/elasticsearch_catalog.md)を使用してElasticsearchに格納されたデータにアクセスすることをサポートします。これにより、Elasticsearch外部テーブルの作成が簡単になります。
- [プレビュー] [Paimonカタログ](../data_source/catalog/paimon_catalog.md)を使用してApache Paimonに格納されたストリーミングデータの分析をサポートします。

#### ストレージエンジン、データインジェスト、クエリ
- 自動パーティショニングを[式パーティショニング](../table_design/expression_partitioning.md)にアップグレードしました。ユーザーは、テーブル作成時に単純なパーティション式（時間関数式または列式）を使用してパーティショニング方法を指定するだけで、StarRocksはデータロード中にデータ特性とパーティション式で定義されたルールに基づいて自動的にパーティションを作成します。このパーティション作成方法は、ほとんどのシナリオに適しており、より柔軟でユーザーフレンドリーです。
- [リストパーティショニング](../table_design/list_partitioning.md)に対応しました。データは、特定の列に対して事前に定義された値のリストに基づいてパーティション分割され、クエリの高速化と明確に分類されたデータのより効率的な管理が可能になります。
- `Information_schema`データベースに`loads`という新しいテーブルを追加しました。ユーザーは、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)と[Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果を`loads`テーブルから照会できます。
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブによってフィルタリングされ除外された非適格データ行のロギングに対応しました。ユーザーはロードジョブで`log_rejected_record_num`パラメータを使用して、ログに記録できるデータ行の最大数を指定できます。
- [ランダムバケッティング](../table_design/Data_distribution.md#how-to-choose-the-bucketing-columns)に対応しました。この機能を使用すると、ユーザーはテーブル作成時にバケッティング列を設定する必要がなく、StarRocksはロードされたデータをランダムにバケットに分散します。この機能と、StarRocksがv2.5.7以降提供しているバケット数(`BUCKETS`)の自動設定機能を組み合わせることで、ユーザーはバケット設定を考慮する必要がなくなり、テーブル作成ステートメントが大幅に簡素化されます。ただし、ビッグデータや高性能が求められるシナリオでは、バケットプルーニングを使用してクエリを高速化できるため、ハッシュバケッティングを引き続き使用することを推奨します。
- [INSERT INTO](../loading/InsertInto.md)でテーブル関数`FILES()`を使用して、AWS S3に保存されているParquet形式またはORC形式のデータファイルからデータを直接ロードすることをサポートしました。`FILES()`関数はテーブルスキーマを自動的に推測できるため、データロード前に外部カタログやファイル外部テーブルを作成する必要がなくなり、データロードプロセスが大幅に簡素化されます。
- [生成列](../sql-reference/sql-statements/generated_columns.md)に対応しました。生成列機能を使用すると、StarRocksは列式の値を自動的に生成して保存し、クエリを自動的に書き換えてクエリパフォーマンスを向上させることができます。
- [Spark connector](../loading/Spark-connector-starrocks.md)を使用してSparkからStarRocksへのデータロードに対応しました。[Spark Load](../loading/SparkLoad.md)と比較して、Spark connectorはより包括的な機能を提供します。ユーザーはSparkジョブを定義してデータに対するETL操作を実行でき、Spark connectorはSparkジョブのシンクとして機能します。
- [MAP](../sql-reference/sql-statements/data-types/Map.md)および[STRUCT](../sql-reference/sql-statements/data-types/STRUCT.md)データ型の列へのデータロードに対応し、ARRAY、MAP、STRUCT内のFast Decimal値のネストにも対応しました。

#### SQLリファレンス

- 以下のストレージボリューム関連ステートメントを追加しました：[CREATE STORAGE VOLUME](../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)、[ALTER STORAGE VOLUME](../sql-reference/sql-statements/Administration/ALTER_STORAGE_VOLUME.md)、[DROP STORAGE VOLUME](../sql-reference/sql-statements/Administration/DROP_STORAGE_VOLUME.md)、[SET DEFAULT STORAGE VOLUME](../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)、[DESC STORAGE VOLUME](../sql-reference/sql-statements/Administration/DESC_STORAGE_VOLUME.md)、[SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW_STORAGE_VOLUMES.md)。

- [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を使用したテーブルコメントの変更に対応しました。[#21035](https://github.com/StarRocks/starrocks/pull/21035)

- 以下の関数を追加しました：

  - Struct関数：[struct (row)](../sql-reference/sql-functions/struct-functions/row.md)、[named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)
  - Map関数：[str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md)、[map_concat](../sql-reference/sql-functions/map-functions/map_concat.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)、[element_at](../sql-reference/sql-functions/map-functions/element_at.md)、[distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md)、[cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)
  - 高階Map関数：[map_filter](../sql-reference/sql-functions/map-functions/map_filter.md)、[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md)、[transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)
  - Array関数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)は`ORDER BY`をサポート、[array_generate](../sql-reference/sql-functions/array-functions/array_generate.md)、[element_at](../sql-reference/sql-functions/array-functions/element_at.md)、[cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)
  - 高階Array関数：[all_match](../sql-reference/sql-functions/array-functions/all_match.md)、[any_match](../sql-reference/sql-functions/array-functions/any_match.md)
  - 集約関数：[min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md)、[percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)
  - テーブル関数：[FILES](../sql-reference/sql-functions/table-functions/files.md)、[generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)
  - 日付関数：[next_day](../sql-reference/sql-functions/date-time-functions/next_day.md)、[previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md)、[last_day](../sql-reference/sql-functions/date-time-functions/last_day.md)、[makedate](../sql-reference/sql-functions/date-time-functions/makedate.md)、[date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)
  - Bitmap関数：[bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md)、[bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)
  - 数学関数：[cosine_similarity](../sql-reference/sql-functions/math-functions/cos_similarity.md)、[cosine_similarity_norm](../sql-reference/sql-functions/math-functions/cos_similarity_norm.md)

#### 権限とセキュリティ

ストレージボリュームに関連する[権限項目](../administration/privilege_item.md#storage-volume)と外部カタログに関連する[権限項目](../administration/privilege_item.md#catalog)を追加し、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)と[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)を使用してこれらの権限の付与と取り消しに対応しました。

### 改善点

#### 共有データクラスタ

共有データStarRocksクラスタのデータキャッシュを最適化しました。最適化されたデータキャッシュにより、ホットデータの範囲を指定できます。また、コールドデータに対するクエリがローカルディスクキャッシュを占有するのを防ぎ、ホットデータに対するクエリのパフォーマンスを確保できます。

#### マテリアライズドビュー

- 非同期マテリアライズドビューの作成を最適化しました：
  - ユーザーがバケット列を指定しない場合、デフォルトでランダムバケッティングを採用します。
  - `ORDER BY`を使用してソートキーを指定することをサポートします。
  - `colocate_group`、`storage_medium`、`storage_cooldown_time`などの属性の指定をサポートします。
  - セッション変数の使用をサポートします。ユーザーは`properties("session.<variable_name>" = "<value>")`構文を使用してこれらの変数を設定し、ビューの更新戦略を柔軟に調整できます。
  - すべての非同期マテリアライズドビューにスピル機能を有効にし、デフォルトで1時間のクエリタイムアウト期間を実装しました。
  - ビューに基づいてマテリアライズドビューを作成することをサポートします。これにより、ユーザーはデータモデリングシナリオでビューとマテリアライズドビューを柔軟に使用し、階層化モデリングを実装できます。
- 非同期マテリアライズドビューを使用したクエリの書き換えを最適化しました：
  - Stale Rewriteに対応し、マテリアライズドビューのベーステーブルが更新されているかどうかに関わらず、指定した時間間隔内に更新されないマテリアライズドビューをクエリの書き換えに使用できます。ユーザーはマテリアライズドビュー作成時に`mv_rewrite_staleness_second`プロパティを使用して時間間隔を指定できます。
  - Hive カタログテーブル上に作成されたマテリアライズドビューに対するView Delta Joinクエリの書き換えをサポートします（主キーと外部キーの定義が必要です）。
  - ユニオン演算を含むクエリの書き換えメカニズムを最適化し、COUNT DISTINCTやtime_sliceなどの関数を含む結合クエリの書き換えをサポートします。
- 非同期マテリアライズドビューのリフレッシュを最適化しました：
  - Hive カタログテーブル上に作成されたマテリアライズドビューのリフレッシュメカニズムを最適化しました。StarRocksはパーティションレベルのデータ変更を検知でき、各自動リフレッシュ時にデータ変更があったパーティションのみをリフレッシュします。
  - `REFRESH MATERIALIZED VIEW WITH SYNC MODE` 構文を使用して、マテリアライズドビューのリフレッシュタスクを同期的に呼び出すことをサポートします。
- 非同期マテリアライズドビューの利用を強化しました：
  - `ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}` を使用してマテリアライズドビューを有効または無効にすることをサポートします。無効化された（`INACTIVE`状態の）マテリアライズドビューはリフレッシュやクエリ書き換えには使用できませんが、直接クエリすることは可能です。
  - `ALTER MATERIALIZED VIEW SWAP WITH` を使用して2つのマテリアライズドビューを交換することをサポートします。ユーザーは新しいマテリアライズドビューを作成し、既存のマテリアライズドビューとアトミックに交換することで、既存のマテリアライズドビューのスキーマ変更を実装できます。
- 同期マテリアライズドビューを最適化しました：
  - SQLヒント `[_SYNC_MV_]` を使用した同期マテリアライズドビューに対する直接クエリをサポートし、稀にクエリが適切に書き換えられない問題を回避できます。
  - `CASE-WHEN`、`CAST`、数学演算などのより多くの式をサポートし、マテリアライズドビューをより多くのビジネスシナリオに適用できるようにしました。

#### データレイクアナリティクス

- Icebergのメタデータキャッシングとアクセスを最適化し、Icebergデータクエリのパフォーマンスを向上させました。
- データキャッシュをさらに最適化し、データレイクアナリティクスのパフォーマンスを向上させました。

#### ストレージエンジン、データ取り込み、クエリ

- 一部のブロッキングオペレーターの中間計算結果をディスクにスピルする機能の一般提供を発表しました。スピル機能を有効にすると、クエリに集約、ソート、または結合オペレーターが含まれる場合、StarRocksはオペレーターの中間計算結果をディスクにキャッシュしてメモリ消費を削減し、メモリ制限によるクエリ失敗を最小限に抑えることができます。[spill](../administration/spill_to_disk.md)
- カーディナリティ保存結合のプルーニングをサポートします。ユーザーがスタースキーマ（例：SSB）やスノーフレークスキーマ（例：TCP-H）で多数のテーブルを保持しているが、これらのテーブルの中で少数のみをクエリする場合、この機能は不要なテーブルをプルーニングして結合のパフォーマンスを向上させます。
- 列モードでの部分更新をサポートします。ユーザーは、[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) ステートメントを使用してプライマリキーテーブルの部分更新を行う際に列モードを有効にすることができます。列モードは少数の列だが多数の行を更新する場合に適しており、更新パフォーマンスを最大10倍向上させることができます。
- CBOの統計収集を最適化し、データ取り込みへの影響を軽減し、統計収集のパフォーマンスを向上させました。
- マージアルゴリズムを最適化し、順列シナリオで全体的なパフォーマンスを最大2倍に向上させました。
- クエリロジックを最適化し、データベースロックへの依存を減らしました。
- 動的パーティショニングはさらに、パーティション単位を年にすることをサポートします。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQLリファレンス

- 条件関数case、coalesce、if、ifnull、nullifはARRAY、MAP、STRUCT、JSONデータ型をサポートします。
- 以下のArray関数は、MAP、STRUCT、ARRAYのネストされた型をサポートします：
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice、array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 以下のArray関数はFast Decimalデータ型をサポートします：
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### バグ修正

以下の問題を修正しました：

- Routine LoadジョブのためのKafkaへの再接続リクエストが適切に処理されない問題。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- 複数のテーブルを含むSQLクエリで、`WHERE`句を含む場合、これらのSQLクエリが同じ意味を持つにも関わらず、各SQLクエリでテーブルの順序が異なると、関連するマテリアライズドビューからの利益を得るために適切に書き換えられない可能性がある問題。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- `GROUP BY`句を含むクエリで重複レコードが返される問題。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- lead()またはlag()関数を呼び出すとBEがクラッシュする問題。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 外部カタログテーブル上に作成されたマテリアライズドビューに基づいて部分パーティションクエリの書き換えが失敗する問題。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- バックスラッシュ(`\`)とセミコロン(`;`)の両方を含むSQLステートメントが適切に解析されない問題。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- テーブル上に作成されたマテリアライズドビューが削除された後、テーブルを切り捨てることができない問題。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 動作変更

- 共有データStarRocksクラスター用のテーブル作成構文から`storage_cache_ttl`パラメータが削除されました。現在、ローカルキャッシュ内のデータはLRUアルゴリズムに基づいて削除されます。
- BE構成項目`disable_storage_page_cache`と`alter_tablet_worker_count`、FE構成項目`lake_compaction_max_tasks`は、不変パラメータから可変パラメータに変更されました。
- BE構成項目`block_cache_checksum_enable`のデフォルト値が`true`から`false`に変更されました。
- BE構成項目`enable_new_load_on_memory_limit_exceeded`のデフォルト値が`false`から`true`に変更されました。
- FE構成項目`max_running_txn_num_per_db`のデフォルト値が`100`から`1000`に変更されました。
- FE構成項目`http_max_header_size`のデフォルト値が`8192`から`32768`に変更されました。
- FE構成項目`tablet_create_timeout_second`のデフォルト値が`1`から`10`に変更されました。
- FE構成項目`max_routine_load_task_num_per_be`のデフォルト値が`5`から`16`に変更され、大量のRoutine Loadタスクが作成された場合にエラー情報が返されます。
- FE構成項目`quorom_publish_wait_time_ms`は`quorum_publish_wait_time_ms`に名称変更され、FE構成項目`async_load_task_pool_size`は`max_broker_load_job_concurrency`に名称変更されました。
- BE構成項目`routine_load_thread_pool_size`は廃止されました。現在、BEノードごとのRoutine LoadスレッドプールサイズはFE構成項目`max_routine_load_task_num_per_be`のみによって制御されます。
- BE構成項目`txn_commit_rpc_timeout_ms`とシステム変数`tx_visible_wait_timeout`は廃止されました。
- FE構成項目`max_broker_concurrency`と`load_parallel_instance_num`は廃止されました。
- FEの設定項目 `max_routine_load_job_num` は非推奨となりました。StarRocksは、`max_routine_load_task_num_per_be` パラメータに基づいて、各BEノードがサポートできるルーチンロードタスクの最大数を動的に推定し、タスク失敗時に提案を行います。
- CNの設定項目 `thrift_port` の名称が `be_port` に変更されました。
- ルーチンロードジョブのプロパティとして、`task_consume_second` と `task_timeout_second` の2つが新たに追加され、ルーチンロードジョブ内の個々のロードタスクがデータを消費する最大時間とタイムアウト期間を制御することができるようになり、ジョブの調整がより柔軟になりました。ユーザーがこれら2つのプロパティをルーチンロードジョブで指定しない場合は、FEの設定項目 `routine_load_task_consume_second` と `routine_load_task_timeout_second` が適用されます。
- セッション変数 `enable_resource_group` は非推奨となりました。これは、v3.1.0以降、[リソースグループ](../administration/resource_group.md)機能がデフォルトで有効になっているためです。
- COMPACTION と TEXT の2つの新しい予約済みキーワードが追加されました。
