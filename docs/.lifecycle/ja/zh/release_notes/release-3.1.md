---
displayed_sidebar: Chinese
---

# StarRocks バージョン 3.1

## 3.1.6

リリース日：2023年12月18日

### 新機能

- [now(p)](https://docs.starrocks.io/zh/docs/3.2/sql-reference/sql-functions/date-time-functions/now/) 関数を追加し、指定した秒の精度（最大でミリ秒）で時間を取得できるようにしました。`p` を指定しない場合、now() 関数は引き続き秒の精度の時間のみを返します。[#36676](https://github.com/StarRocks/starrocks/pull/36676)
- モニタリングメトリック `max_tablet_rowset_num`（Rowsetの最大数を設定するためのもの）を追加し、Compactionに問題が発生する可能性を事前に検出し、適切な対応を行うことで、"too many versions"のエラーメッセージの発生を減らすことができます。[#36539](https://github.com/StarRocks/starrocks/pull/36539)
- Heap Profileをコマンドラインから取得できるようになり、問題発生後のトラブルシューティングを容易にしました。[#35322](https://github.com/StarRocks/starrocks/pull/35322)
- Common Table Expression (CTE)を使用して非同期マテリアライズドビューを作成できるようになりました。[#36142](https://github.com/StarRocks/starrocks/pull/36142)
- 以下のBitmap関数を追加しました：[subdivide_bitmap](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/subdivide_bitmap/)、[bitmap_from_binary](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/bitmap_from_binary/)、[bitmap_to_binary](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/bitmap-functions/bitmap_to_binary/)。[#35817](https://github.com/StarRocks/starrocks/pull/35817) [#35621](https://github.com/StarRocks/starrocks/pull/35621)

### パラメータの変更

- FEの設定項目 `enable_new_publish_mechanism` を静的パラメータに変更しました。変更後はFEを再起動する必要があります。[#35338](https://github.com/StarRocks/starrocks/pull/35338)
- Trashファイルのデフォルトの有効期限を1日に調整しました（元は3日でした）。[#37113](https://github.com/StarRocks/starrocks/pull/37113)
- [FEの設定項目](https://docs.starrocks.io/zh/docs/administration/Configuration/#配置-fe-动态参数) `routine_load_unstable_threshold_second` を追加しました。[#36222](https://github.com/StarRocks/starrocks/pull/36222)
- BEの設定項目 `enable_stream_load_verbose_log` を追加しました。デフォルト値は `false` で、オンにするとログにStream LoadのHTTPリクエストとレスポンスの情報が記録され、問題のトラブルシューティングが容易になります。[#36113](https://github.com/StarRocks/starrocks/pull/36113)
- BEの設定項目 `enable_lazy_delta_column_compaction` を追加しました。デフォルト値は `true` で、頻繁なデルタカラムのCompactionを行わないことを意味します。[#36654](https://github.com/StarRocks/starrocks/pull/36654)

### 機能の改善

- システム変数 [sql_mode](https://docs.starrocks.io/zh/docs/reference/System_variable/#sql_mode) に `GROUP_CONCAT_LEGACY` オプションを追加し、2.5（含まず）以前のバージョンでの [group_concat](https://docs.starrocks.io/zh/docs/sql-reference/sql-functions/string-functions/group_concat/) 関数の実装ロジックとの互換性を確保しました。[#36150](https://github.com/StarRocks/starrocks/pull/36150)
- 主キーモデルテーブルの [SHOW DATA](https://docs.starrocks.io/zh/docs/sql-reference/sql-statements/data-manipulation/SHOW_DATA/) の結果に、**.cols** ファイル（一部の列の更新と生成列に関連するファイル）と永続化インデックスファイルのファイルサイズ情報を追加しました。[#34898](https://github.com/StarRocks/starrocks/pull/34898)
- MySQL外部テーブルとJDBCカタログ外部テーブルのWHERE句にキーワードを含めることができるようになりました。[#35917](https://github.com/StarRocks/starrocks/pull/35917)
- StarRocksのプラグインのロードに失敗した場合、例外をスローしてFEを起動できなくするのではなく、FEは正常に起動し、[SHOW PLUGINS](https://docs.starrocks.io/docs/sql-reference/sql-statements/Administration/SHOW_PLUGINS/) を使用してプラグインのエラーステータスを確認できるようになりました。[#36566](https://github.com/StarRocks/starrocks/pull/36566)
- ダイナミックパーティションでランダムな分布をサポートしました。[#35513](https://github.com/StarRocks/starrocks/pull/35513)
- [SHOW ROUTINE LOAD](https://docs.starrocks.io/zh/docs/sql-reference/sql-statements/data-manipulation/SHOW_ROUTINE_LOAD/) の結果に `OtherMsg` を追加し、最後の失敗したタスクに関する情報を表示するようにしました。[#35806](https://github.com/StarRocks/starrocks/pull/35806)
- 監査ログ（Audit Log）のBroker Loadジョブに含まれるAWS S3の認証情報 `aws.s3.access_key` と `aws.s3.access_secret` を非表示にしました。[#36571](https://github.com/StarRocks/starrocks/pull/36571)
- `be_tablets` テーブルに持続化インデックスのディスク使用量を記録する `INDEX_DISK` を追加しました。単位はバイトです。[#35615](https://github.com/StarRocks/starrocks/pull/35615)

### 問題の修正

以下の問題を修正しました：

- データが破損している場合、永続化インデックスの作成がBEのクラッシュを引き起こす問題を修正しました。[#30841](https://github.com/StarRocks/starrocks/pull/30841)
- ネストされたクエリを含む非同期マテリアライズドビューを作成すると、「Resolve partition column failed」というエラーが発生する問題を修正しました。[#26078](https://github.com/StarRocks/starrocks/issues/26078)
- 基礎テーブルのデータが破損している場合、非同期マテリアライズドビューの作成時に「Unexpected exception: null」というエラーが発生する可能性がある問題を修正しました。[#30038](https://github.com/StarRocks/starrocks/pull/30038)
- ウィンドウ関数を含むクエリで「Row count of const column reach limit: 4294967296」というSQLエラーが発生する問題を修正しました。[#33561](https://github.com/StarRocks/starrocks/pull/33561)
- FEの設定項目 `enable_collect_query_detail_info` を有効にすると、FEのパフォーマンスが著しく低下する問題を修正しました。[#35945](https://github.com/StarRocks/starrocks/pull/35945)
- オブジェクトストレージからファイルを削除する際に「Reduce your request rate」というエラーが発生する場合がある問題を修正しました。[#35566](https://github.com/StarRocks/starrocks/pull/35566)
- 特定の状況下で、マテリアライズドビューのリフレッシュ時にデッドロックが発生する問題を修正しました。[#35736](https://github.com/StarRocks/starrocks/pull/35736)
- DISTINCTを使用したウィンドウ関数の出力列に複雑な式が含まれている場合にSELECT DISTINCT操作がエラーになる問題を修正しました。[#36357](https://github.com/StarRocks/starrocks/pull/36357)
- ORC形式のデータファイルからネストされた配列をインポートするとBEがクラッシュする問題を修正しました。[#36127](https://github.com/StarRocks/starrocks/pull/36127)
- S3プロトコルに対応した一部のオブジェクトストレージが重複したファイルを返すため、BEがクラッシュする問題を修正しました。[#36103](https://github.com/StarRocks/starrocks/pull/36103)

## 3.1.5

リリース日：2023年11月28日

### 新機能

- ストアと計算を分離したモードでCNノードがデータのエクスポートをサポートするようになりました。[#34018](https://github.com/StarRocks/starrocks/pull/34018)

### 機能の改善

- [`INFORMATION_SCHEMA.COLUMNS`](../reference/information_schema/columns.md) テーブルがARRAY、MAP、STRUCT型のフィールドを表示するようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- [Hive](../data_source/catalog/hive_catalog.md) でLZOアルゴリズムで圧縮されたParquet、ORC、およびCSV形式のファイルをクエリすることができるようになりました。[#30923](https://github.com/StarRocks/starrocks/pull/30923)  [#30721](https://github.com/StarRocks/starrocks/pull/30721)
- 自動パーティションテーブルでもパーティション名を指定して更新することができるようになりました。パーティションが存在しない場合はエラーが発生します。[#34777](https://github.com/StarRocks/starrocks/pull/34777)
- 物理テーブル、ビュー、およびビュー内のテーブルがSwap、Drop、またはSchema Change操作を実行した後に、物理ビューを自動的にリフレッシュできるようになりました。[#32829](https://github.com/StarRocks/starrocks/pull/32829)
- Bitmap関連の一部の操作のパフォーマンスを最適化しました。主な改善点は以下です：
  - ネストループ結合のパフォーマンスを最適化しました。[#340804](https://github.com/StarRocks/starrocks/pull/34804)  [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor` 関数のパフォーマンスを最適化しました。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - Copy on Write（COW）をサポートし、パフォーマンスを最適化し、メモリ使用量を減らしました。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### 問題の修正

以下の問題を修正しました：

- フィルタ条件を含むBroker Loadジョブを送信すると、データのインポート中に一部の場合にBEがクラッシュする問題を修正しました。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTS で `unknown error` と表示される問題を修正しました。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 式を自動パーティション列として使用する場合、データのインポート時に「Error: The row create partition failed since Runtime error: failed to analyse partition value」というエラーが発生する問題を修正しました。[#33513](https://github.com/StarRocks/starrocks/pull/33513)
- クエリ実行時に「get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction」というエラーが表示される問題を修正しました。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- ストアと計算を分離したモードで、IcebergまたはHive外部テーブルを単独でクエリするとBEがクラッシュする問題を修正しました。[#34682](https://github.com/StarRocks/starrocks/pull/34682)
- ストアと計算を分離したモードで、データをインポートする際に複数のパーティションが自動的に作成され、データが誤ったパーティションに書き込まれる場合がある問題を修正しました。[#34731](https://github.com/StarRocks/starrocks/pull/34731)
- 持続化インデックスをオープンしている主キーモデルテーブルに大量のデータを高頻度でインポートすると、BEがクラッシュする可能性がある問題を修正しました。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリ実行時に「Exception: java.lang.IllegalStateException: null」というエラーが表示される問題を修正しました。[#33535](https://github.com/StarRocks/starrocks/pull/33535)
- `show proc '/current_queries';` を実行すると、クエリが開始されたばかりの場合にBEがクラッシュする可能性がある問題を修正しました。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 持続化インデックスをオープンしている主キーモデルテーブルに大量のデータをインポートすると、エラーが発生する場合がある問題を修正しました。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- 2.4以前のバージョンからアップグレードすると、Compaction Scoreが非常に高くなる問題が発生する可能性がある問題を修正しました。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- MariaDB ODBC Driverを使用して `INFORMATION_SCHEMA` の情報をクエリすると、`schemata` ビューの `CATALOG_NAME` 列の値がすべて `null` になる問題を修正しました。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- データのインポート時にFEがクラッシュし、再起動できなくなる問題を修正しました。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- Stream Loadジョブが **PREPARD** 状態であり、同時にSchema Changeが実行されている場合にデータが失われる問題を修正しました。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- HDFSパスが2つ以上のスラッシュ(`/`)で終わる場合、HDFSのバックアップとリストアが失敗する問題を修正しました。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- `enable_load_profile` をオンにすると、Stream Loadが容易に失敗する問題を修正しました。[#34544](https://github.com/StarRocks/starrocks/pull/34544)

- 主キーモデルテーブルの一部の列を列モードで更新すると、レプリカ間でデータの不整合が発生する可能性があります。[#34555](https://github.com/StarRocks/starrocks/pull/34555)
- ALTER TABLEを使用して`partition_live_number`属性を追加しても効果がありません。[#34842](https://github.com/StarRocks/starrocks/pull/34842)
- FEの起動に失敗し、「failed to load journal type 118」というエラーが表示されます。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- `recover_with_empty_tablet`を`true`に設定すると、FEがクラッシュする可能性があります。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- レプリカ操作のリプレイに失敗すると、FEがクラッシュする可能性があります。[#32295](https://github.com/StarRocks/starrocks/pull/32295)

### 互換性の変更

#### 設定パラメータ

- 新しいFE設定項目[`enable_statistics_collect_profile`](../administration/FE_configuration.md#enable_statistics_collect_profile)が追加されました。これは、統計情報のクエリ時にプロファイルを生成するかどうかを制御するためのもので、デフォルト値は`false`です。[#33815](https://github.com/StarRocks/starrocks/pull/33815)
- FE設定項目[`mysql_server_version`](../administration/FE_configuration.md#mysql_server_version)が静的から動的（`mutable`）に変更されました。設定項目を変更した後、FEを再起動する必要はありません。現在のセッションで動的に有効になります。[#34033](https://github.com/StarRocks/starrocks/pull/34033)
- 新しいBE/CN設定項目[`update_compaction_ratio_threshold`](../administration/BE_configuration.md#update_compaction_ratio_threshold)が追加されました。これは、主キーモデルテーブルの単一のCompactionでマージされる最大データ比率を手動で設定するためのもので、デフォルト値は`0.5`です。単一のTabletが大きすぎる場合は、この設定値を適切に減らすことをお勧めします。ストアと計算が統合されたモードでは、主キーモデルテーブルの単一のCompactionでマージされる最大データ比率は、引き続き自動調整モードを維持します。[#35129](https://github.com/StarRocks/starrocks/pull/35129)

#### システム変数

- 新しいセッション変数`cbo_decimal_cast_string_strict`が追加されました。これは、DECIMAL型をSTRING型に変換する際のオプティマイザの動作を制御するためのものです。値が`true`の場合、v2.5.x以降のバージョンの処理ロジックが使用され、厳密な変換（つまり、スケールに基づいて切り捨てて`0`を補完）が実行されます。値が`false`の場合、v2.5.x以前のバージョンの処理ロジック（つまり、有効桁数に基づいて処理）が保持されます。デフォルト値は`true`です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 新しいセッション変数`cbo_eq_base_type`が追加されました。これは、DECIMAL型とSTRING型のデータ比較時の強制型を指定するためのもので、デフォルト値は`VARCHAR`で、`DECIMAL`を選択することもできます。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- 新しいセッション変数`big_query_profile_second_threshold`が追加されました。この変数は、セッション変数[`enable_profile`](../reference/System_variable.md#enable_profile)が`false`に設定され、クエリの実行時間が`big_query_profile_second_threshold`で設定された閾値を超える場合にプロファイルを生成するためのものです。[#33825](https://github.com/StarRocks/starrocks/pull/33825)

## 3.1.4

リリース日：2023年11月2日

### 新機能

- ストアと計算が分離された状態での主キーモデル（Primary Key）テーブルは、ソートキーをサポートしています。
- 非同期マテリアライズドビューは、str2date関数を使用してパーティション式を指定できるようになりました。これは、外部テーブルのパーティションタイプがSTRING型のデータの増分リフレッシュとクエリの書き換えに使用できます。[#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- 新しいセッション変数[`enable_query_tablet_affinity`](../reference/System_variable.md#enable_query_tablet_affinity25-及以后)が追加されました。これは、同じTabletを複数回クエリする場合に同じレプリカを選択するかどうかを制御するためのもので、デフォルトでは無効になっています。[#33049](https://github.com/StarRocks/starrocks/pull/33049)
- `is_role_in_session`というユーティリティ関数が追加されました。これは、指定されたロールが現在のセッションでアクティブになっているかどうかを確認し、ネストされたロールのアクティブ化もサポートしています。[#32984](https://github.com/StarRocks/starrocks/pull/32984)
- リソースグループレベルのクエリキューが追加されました。グローバル変数`enable_group_level_query_queue`を有効にする必要があります（デフォルトは`false`）。グローバルレベルまたはリソースグループレベルのいずれかのリソースが閾値に達すると、クエリはキューに入れられ、すべてのリソースが閾値を超えないまでクエリは実行されません。
  - 各リソースグループでは、単一のBEノードでの並行クエリの上限を制限するために`concurrency_limit`を設定できます。
  - 各リソースグループでは、単一のBEノードで使用できるCPUの上限を制限するために`max_cpu_cores`を設定できます。
- リソースグループの分類器には、`plan_cpu_cost_range`と`plan_mem_cost_range`の2つのパラメータが追加されました。
  - `plan_cpu_cost_range`：システムが推定するクエリのCPUコストの範囲です。デフォルトは`NULL`で、制限はありません。
  - `plan_mem_cost_range`：システムが推定するクエリのメモリコストの範囲です。デフォルトは`NULL`で、制限はありません。

### 機能の改善

- 窓関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMPは、ORDER BY句とWINDOW句をサポートしています。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型のデータのクエリ結果が範囲外の場合、NULLではなくエラーが返されます。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- クエリキューの並行クエリ数は、リーダーFEが管理するようになりました。各フォロワーFEは、クエリの開始と終了時にリーダーFEに通知します。グローバルレベルまたはリソースグループレベルの`concurrency_limit`を超える場合、クエリは拒否されるかキューに入ります。

### 問題の修正

以下の問題が修正されました：

- メモリの統計が正確でないため、SparkまたはFlinkでデータを読み取る際にエラーが発生する可能性があります。[#30702](https://github.com/StarRocks/starrocks/pull/30702) [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- メタデータキャッシュのメモリ使用量の統計が正確ではありません。[#31978](https://github.com/StarRocks/starrocks/pull/31978)
- libcurlを呼び出すと、BEがクラッシュする可能性があります。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- Hiveビューで作成されたStarRocksのマテリアライズドビューをリフレッシュすると、「java.lang.ClassCastException: com.starrocks.catalog.HiveView cannot be cast to com.starrocks.catalog.HiveMetaStoreTable」というエラーが発生します。[#31004](https://github.com/StarRocks/starrocks/pull/31004)
- ORDER BY句に集約関数が含まれている場合、「java.lang.IllegalStateException: null」というエラーが発生します。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- ストアと計算が分離されたモードでは、テーブルのキー情報が`information_schema.COLUMNS`に記録されず、Flink Connectorを使用してデータをインポートする際にDELETE操作が実行できません。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- Flink Connectorを使用してデータをインポートする際に、高い並行性と制限されたHTTPおよびScanスレッド数の場合にフリーズが発生する可能性があります。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- より小さいバイト型のフィールドを追加した場合、変更が完了する前にSELECT COUNT(*)を実行すると「error: invalid field name」というエラーが発生します。[#33243](https://github.com/StarRocks/starrocks/pull/33243)
- クエリキャッシュが有効になっている場合、クエリ結果にエラーがあります。[#32781](https://github.com/StarRocks/starrocks/pull/32781)
- ハッシュ結合時のクエリの失敗により、BEがクラッシュする可能性があります。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- BINARYまたはVARBINARY型は、`information_schema.columns`ビューの`DATA_TYPE`および`COLUMN_TYPE`で「unknown」と表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 動作の変更

- 3.1.4以降、新規に構築された主キーモデルの永続インデックスは、テーブル作成時にデフォルトでオンになります（3.1.4にアップグレードする場合は変更されません）。[#33374](https://github.com/StarRocks/starrocks/pull/33374)
- 新しいFEパラメータ`enable_sync_publish`が追加され、デフォルトで有効になります。`true`に設定すると、主キーモデルテーブルのインポートプロセスはApplyが完了するまで結果を返さず、インポートジョブが成功した後すぐにデータが表示されるようになりますが、元のよりも遅延が発生する可能性があります（以前はこのパラメータは存在せず、インポート時のPublishプロセスは非同期でした）。[#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

リリース日：2023年9月25日

### 新機能

- ストアと計算が分離された状態での主キーモデルは、ローカルディスク上の永続インデックスをサポートしています。使用方法はストアと計算が統合された場合と同じです。
- 集約関数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)は、DISTINCTキーワードとORDER BY句の使用をサポートしています。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Kafka Connector](../loading/Kafka-connector-starrocks.md)、[Flink Connector](../loading/Flink-connector-starrocks.md)、および[Spark Connector](../loading/Spark-connector-starrocks.md)は、主キーモデルテーブルの一部の列を更新する際に列モードを有効にすることができます。[#28288](https://github.com/StarRocks/starrocks/pull/28288)
- パーティション内のデータは時間の経過とともに自動的に冷却されるようになりました（[Listパーティショニング方式](../table_design/list_partitioning.md)はまだサポートされていません）。[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 機能の改善

- 不正なコメントを含むSQLコマンドの実行結果がMySQLと同じになるように改善されました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)

### 問題の修正

以下の問題が修正されました：

- DELETE文を実行する際、WHERE条件のフィールドの型がBITMAPまたはHLLの場合、文の実行に失敗することがあります。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- 特定のFollower FEが再起動されると、CpuCoresが同期されないため、クエリのパフォーマンスに影響があります。[#28472](https://github.com/StarRocks/starrocks/pull/28472) [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- [to_bitmap()](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数のコスト統計の計算が正しくないため、マテリアライズドビューの書き換え後に誤った実行計画が選択されることがあります。[#29961](https://github.com/StarRocks/starrocks/pull/29961)
- ストアと計算が分離されたアーキテクチャの特定のシナリオでは、Follower FEが再起動されると、そのFollower FEに送信されたクエリが「Backend node not found. Check if any backend node is down」というエラーメッセージを返します。[#28615](https://github.com/StarRocks/starrocks/pull/28615)
- ALTER TABLEのプロセス中に継続的なインポートが行われると、「Tablet is in error state」というエラーが発生することがあります。[#29364](https://github.com/StarRocks/starrocks/pull/29364)
```
- FEの動的パラメータ `max_broker_load_job_concurrency` をオンラインで変更しても効果がありません。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- [date_diff()](../sql-reference/sql-functions/date-time-functions/date_diff.md) 関数を呼び出す際、関数内で指定された時間単位が定数であり、日付が非定数である場合、BEがクラッシュする可能性があります。[#29937](https://github.com/StarRocks/starrocks/issues/29937)
- [Hive Catalog](../data_source/catalog/hive_catalog.md) が多階層ディレクトリであり、データが腾讯云COSに格納されている場合、クエリ結果が正しくないことがあります。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- ストレージと計算の分離アーキテクチャで、データの非同期書き込みを有効にしても、自動パーティションが機能しません。[#29986](https://github.com/StarRocks/starrocks/issues/29986)
- [CREATE TABLE LIKE](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md) ステートメントを使用してプライマリキーモデルテーブルを作成すると、`Unexpected exception: Unknown properties: {persistent_index_type=LOCAL}`というエラーが発生します。[#30255](https://github.com/StarRocks/starrocks/pull/30255)
- プライマリキーモデルテーブルをリストアした後、BEの再起動後にメタデータが破損し、メタデータが一貫しなくなります。[#30135](https://github.com/StarRocks/starrocks/pull/30135)
- プライマリキーモデルテーブルのインポート時に、Truncate操作とクエリが同時に行われると、「java.lang.NullPointerException」というエラーが発生する場合があります。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 谓詞式を含むマテリアライズドビューの作成文で、リフレッシュ結果が正しくありません。[#29904](https://github.com/StarRocks/starrocks/pull/29904)
- 3.1.2にアップグレードすると、以前に作成したテーブルのストレージボリュームプロパティが `null` にリセットされます。[#30647](https://github.com/StarRocks/starrocks/pull/30647)
- Tabletのメタデータのチェックポイントとリストア操作が並行して行われると、一部のレプリカが見つからなくなる可能性があります。[#30603](https://github.com/StarRocks/starrocks/pull/30603)
- テーブルのフィールドが `NOT NULL` であり、デフォルト値が設定されていない場合、CloudCanalを使用してインポートする際に「Unsupported dataFormat value is : \N」というエラーが発生します。[#30799](https://github.com/StarRocks/starrocks/pull/30799)

### 動作変更

- 集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) の区切り文字は `SEPARATOR` キーワードで宣言する必要があります。
- セッション変数 [`group_concat_max_len`](../reference/System_variable.md#group_concat_max_len)（集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) が返す文字列の最大長を制御するためのもの）のデフォルト値は、制限がない状態からデフォルトの `1024` に変更されました。

## 3.1.2

リリース日：2023年8月25日

### 問題の修正

以下の問題が修正されました：

- ユーザーがデフォルトデータベースを指定して接続し、そのデータベースにのみテーブルの権限があるが、そのデータベースの権限がない場合、そのデータベースにアクセスできないというエラーが発生します。[#29767](https://github.com/StarRocks/starrocks/pull/29767)
- RESTful API `show_data` は、クラウドネイティブテーブルの戻り値が正しくありません。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- [array_agg()](../sql-reference/sql-functions/array-functions/array_agg.md) 関数の実行中にクエリがキャンセルされると、BEがクラッシュします。[#29400](https://github.com/StarRocks/starrocks/issues/29400)
- [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) および [HLL](../sql-reference/sql-statements/data-types/HLL.md) の列は、[SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) クエリの結果の `Default` フィールドの値が正しくありません。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- [array_map()](../sql-reference/sql-functions/array-functions/array_map.md) は、複数のテーブルを同時に参照する場合、クエリの失敗を引き起こす下押し戦略の問題があります。[#29504](https://github.com/StarRocks/starrocks/pull/29504)
- 上流のApache ORCのBugFix ORC-1304（[apache/orc#1299](https://github.com/apache/orc/pull/1299)）がマージされていないため、ORCファイルのクエリが失敗することがあります。[#29804](https://github.com/StarRocks/starrocks/pull/29804)

### 動作変更

このバージョンから、SET CATALOG操作を実行するには、対象のCatalogにUSAGE権限が必要です。[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)コマンドを使用して権限を付与することができます。

既存のユーザーのアップグレードロジックがすでに設定されている場合、低いバージョンからアップグレードする場合は、再度権限を付与する必要はありません。[#29389](https://github.com/StarRocks/starrocks/pull/29389)。新しい権限を追加する場合は、対象のCatalogにUSAGE権限を付与する必要があります。

## 3.1.1

リリース日：2023年8月18日

### 新機能

- [ストレージと計算の分離アーキテクチャ](../deployment/shared_data/s3.md)で、以下の機能がサポートされています：
  - データのAzure Blob Storageへの保存。
  - リストパーティション。
- 集約関数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md) がサポートされています。
- [ウィンドウ関数](../sql-reference/sql-functions/Window_function.md) COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP がサポートされています。

### 機能の最適化

すべての複合述語およびWHERE句の式に対して暗黙の型変換がサポートされ、[セッション変数](../reference/System_variable.md) `enable_strict_type` を使用して暗黙の型変換を有効にするかどうかを制御できます（デフォルト値は `false`）。

### 問題の修正

以下の問題が修正されました：

- 複数のレプリカを持つテーブルにデータをインポートする際、一部のパーティションにデータが存在しない場合、無駄なログが書き込まれます。[#28824](https://github.com/StarRocks/starrocks/issues/28824)
- プライマリキーモデルテーブルの一部の列を更新する際、平均行サイズの推定が正確でないため、メモリ使用量が過剰になります。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 特定のTabletが特定のエラーステータスになった後にClone操作がトリガされると、ディスク使用量が増加します。[#28488](https://github.com/StarRocks/starrocks/pull/28488)
- コンパクションによって冷データがローカルキャッシュに書き込まれます。[#28831](https://github.com/StarRocks/starrocks/pull/28831)

## 3.1.0

リリース日：2023年8月7日

### 新機能

#### ストレージと計算の分離アーキテクチャ

- プライマリキーモデル（Primary Key）テーブルのサポートが追加されましたが、永続化インデックスはまだサポートされていません。
- [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md) 属性がサポートされ、テーブル内でグローバルに一意のIDを提供し、データ管理を簡素化します。
- [インポート時にパーティションを自動作成し、パーティション式でパーティションルールを定義する](../table_design/expression_partitioning.md)機能がサポートされ、パーティションの作成の使いやすさと柔軟性が向上しました。
- [ストレージボリューム（Storage Volume）の抽象化](../deployment/shared_data/s3.md)がサポートされ、ストレージと計算の分離アーキテクチャでのストレージの設定や認証などの関連情報の設定が容易になりました。後続のライブラリテーブルの作成時に直接参照することができ、使いやすさが向上しました。

#### データレイク分析

- [Hive Catalog](../data_source/catalog/hive_catalog.md) 内のビューにアクセスする機能がサポートされました。
- Parquet形式のIceberg v2データテーブルにアクセスする機能がサポートされました。
- [Parquet形式のIcebergテーブルにデータを書き込む](../data_source/catalog/iceberg_catalog.md#向-iceberg-表中插入数据)機能がサポートされました。
- 【パブリックベータ】外部の[Elasticsearch catalog](../data_source/catalog/elasticsearch_catalog.md)を介してElasticsearchにアクセスする機能がサポートされ、外部テーブルの作成などのプロセスが簡素化されました。
- 【パブリックベータ】[Paimon catalog](../data_source/catalog/paimon_catalog.md)がサポートされ、StarRocksを使用してストリーミングデータをデータレイク分析するためのヘルプが提供されます。

#### ストレージ、インポート、およびクエリ

- 自動パーティション作成機能が[式パーティション](../table_design/expression_partitioning.md)にアップグレードされ、テーブル作成時に単純なパーティション式（時間関数式またはリスト式）を使用して必要なパーティション方法を構成するだけで、StarRocksがデータとパーティション式の定義ルールに基づいて自動的にパーティションを作成します。このようなパーティションの作成方法は、より柔軟で使いやすく、ほとんどのシナリオに対応できます。
- [リストパーティション](../table_design/list_partitioning.md)がサポートされました。データはパーティション列の列挙値リストに基づいてパーティションされ、クエリの高速化と明確に分類されたデータの効率的な管理が可能です。
- `Information_schema` スキーマに `loads` テーブルが追加され、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) と [Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md) ジョブの結果情報をクエリできるようになりました。
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) は、データ品質が不適格なためにフィルタリングされたエラーデータ行を出力する機能をサポートし、`log_rejected_record_num` パラメータを使用して出力する最大データ行数を設定できます。
- [ランダムバケット分割（Random Bucketing）](../table_design/Data_distribution.md#设置分桶)機能がサポートされ、テーブル作成時にバケットキーを選択する必要がなくなりました。StarRocksはインポートデータをランダムに各バケットに分散させるようになりました。また、2.5.7以降のバージョンでサポートされている自動バケット数（`BUCKETS`）機能と組み合わせて使用することで、バケットの設定について心配する必要がなくなり、テーブル作成ステートメントが大幅に簡素化されます。ただし、大規模なデータや高性能を要求するシナリオでは、引き続きハッシュバケット方式を使用し、バケットプルーニングを活用してクエリを高速化することをお勧めします。
- [INSERT INTO](../loading/InsertInto.md) ステートメントで FILES() 関数を使用して、AWS S3またはHDFSから直接ParquetまたはORC形式のファイルのデータをインポートする機能がサポートされました。FILES() 関数は、テーブル構造（テーブルスキーマ）を自動的に推論し、事前にExternal Catalogや外部ファイルテーブルを作成する必要がなくなりました。
- [生成列（Generated Column）](../sql-reference/sql-statements/generated_columns.md)機能がサポートされ、生成式の値を自動的に計算して保存し、クエリ時に自動的に書き換えることで、クエリのパフォーマンスを向上させることができます。
- [Spark connector](../loading/Spark-connector-starrocks.md)を使用して、SparkデータをStarRocksにインポートする機能がサポートされました。[Spark Load](../loading/SparkLoad.md)と比較して、Spark connectorの機能がより充実しています。SparkジョブでデータをETL処理するためにSparkジョブをカスタマイズし、Spark connectorはSparkジョブ内のsinkとしてのみ使用します。
- [MAP](../sql-reference/sql-statements/data-types/Map.md)、[STRUCT](../sql-reference/sql-statements/data-types/STRUCT.md)型のフィールドにデータをインポートする機能がサポートされ、ARRAY、MAP、STRUCT型でFast Decimal型がサポートされました。

#### SQLステートメントと関数

- ストレージボリュームに関連するSQLステートメントが追加されました：[CREATE STORAGE VOLUME](../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md)、[ALTER STORAGE VOLUME](../sql-reference/sql-statements/Administration/ALTER_STORAGE_VOLUME.md)、[DROP STORAGE VOLUME](../sql-reference/sql-statements/Administration/DROP_STORAGE_VOLUME.md)、[SET DEFAULT STORAGE VOLUME](../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md)、[DESC STORAGE VOLUME](../sql-reference/sql-statements/Administration/DESC_STORAGE_VOLUME.md)、[SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW_STORAGE_VOLUMES.md)。

```
- [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を使用して、テーブルのコメントを変更することがサポートされました。[#21035](https://github.com/StarRocks/starrocks/pull/21035)

- 次の関数が追加されました：

  - Struct関数：[struct (row)](../sql-reference/sql-functions/struct-functions/row.md)、[named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)
  - Map関数：[str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md)、[map_concat](../sql-reference/sql-functions/map-functions/map_concat.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)、[element_at](../sql-reference/sql-functions/map-functions/element_at.md)、[distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md)、[cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)
  - Map高階関数：[map_filter](../sql-reference/sql-functions/map-functions/map_filter.md)、[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md)、[transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)
  - Array関数：[array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)は、`ORDER BY`をサポートしています、[array_generate](../sql-reference/sql-functions/array-functions/array_generate.md)、[element_at](../sql-reference/sql-functions/array-functions/element_at.md)、[cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)
  - Array高階関数：[all_match](../sql-reference/sql-functions/array-functions/all_match.md)、[any_match](../sql-reference/sql-functions/array-functions/any_match.md)
  - 集約関数：[min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md)、[percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)
  - テーブル関数：[FILES](../sql-reference/sql-functions/table-functions/files.md)、[generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)
  - 日付関数：[next_day](../sql-reference/sql-functions/date-time-functions/next_day.md)、[previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md)、[last_day](../sql-reference/sql-functions/date-time-functions/last_day.md)、[makedate](../sql-reference/sql-functions/date-time-functions/makedate.md)、[date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)
  - ビットマップ関数：[bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md)、[bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)
  - 数学関数：[cosine_similarity](../sql-reference/sql-functions/math-functions/cos_similarity.md)、[cosine_similarity_norm](../sql-reference/sql-functions/math-functions/cos_similarity_norm.md)

#### 権限とセキュリティ

ストレージボリュームに関連する[権限項目](../administration/privilege_item.md#ストレージボリュームの権限)と外部データディレクトリ（外部カタログ）に関連する[権限項目](../administration/privilege_item.md#データディレクトリの権限)が追加され、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)ステートメントを使用して関連する権限の付与と取り消しがサポートされるようになりました。

### 機能の最適化

#### ストレージと計算の分離アーキテクチャ

ストレージと計算の分離アーキテクチャのデータキャッシュ機能が最適化され、ホットデータの範囲を指定し、クエリのパフォーマンスに影響を与えることなくクールデータのクエリを防止することができます。

#### マテリアライズドビュー

- 非同期マテリアライズドビューの作成の最適化：
  - ランダムバケット（Random Bucketing）をサポートし、バケット化列（Bucketing Column）を指定しない場合はデフォルトでランダムバケット化が使用されます。
  - `ORDER BY`を使用してソートキーを指定できます。
  - `colocate_group`、`storage_medium`、`storage_cooldown_time`などのプロパティを使用できます。
  - セッション変数（Session Variable）を使用して、`properties("session.<variable_name>" = "<value>")`のように設定し、ビューのリフレッシュの実行戦略を柔軟に調整できます。
  - すべてのマテリアライズドビューのリフレッシュにSpill機能がデフォルトで有効になり、クエリのタイムアウト時間は1時間です。
  - ビュー（View）に基づいてマテリアライズドビューを作成することができます。データモデリングのシナリオの使いやすさを向上させるために、ビューとマテリアライズドビューを柔軟に使用できます。
- 非同期マテリアライズドビューのクエリの改変の最適化：
  - Stale Rewriteをサポートし、指定された時間内にリフレッシュされていないマテリアライズドビューをクエリの改変に直接使用できるようにします。基になるテーブルのデータが更新されているかどうかに関係なくです。マテリアライズドビューの作成時に`mv_rewrite_staleness_second`プロパティを使用して許容できる未リフレッシュ時間を設定できます。
  - Hiveカタログの外部テーブルから作成されたマテリアライズドビューは、View Delta Joinシナリオのクエリの改変をサポートします（主キーと外部キーの制約を定義する必要があります）。
  - Join派生の改変、Count Distinct、time_slice関数などのシナリオの改変をサポートし、Unionの改変を最適化します。
- 非同期マテリアライズドビューのリフレッシュの最適化：
  - Hiveカタログの外部テーブルのマテリアライズドビューのリフレッシュメカニズムを最適化し、StarRocksはパーティションレベルのデータ変更を検知し、データが変更されたパーティションのみを自動的にリフレッシュします。
  - `REFRESH MATERIALIZED VIEW WITH SYNC MODE`を使用して同期的にマテリアライズドビューのリフレッシュタスクを呼び出すことができます。
- 非同期マテリアライズドビューの使用の強化：
  - `ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}`ステートメントを使用してマテリアライズドビューを有効または無効にすることができます。無効（`INACTIVE`状態）のマテリアライズドビューはリフレッシュに使用されず、クエリの改変にも使用されませんが、直接クエリすることはできます。
  - `ALTER MATERIALIZED VIEW SWAP WITH`を使用してマテリアライズドビューを置換することができます。新しいマテリアライズドビューを作成し、元のビューとのアトミックな置換を行うことで、マテリアライズドビューのスキーマ変更を実現できます。
- 同期マテリアライズドビューの最適化：
  - SQL Hint `[_SYNC_MV_]`を使用して直接同期マテリアライズドビューをクエリすることができます。自動的に改変できないシナリオを回避します。
  - 同期マテリアライズドビューは、CASE-WHEN、CAST、数学演算などの式をサポートするようになり、使用シナリオが拡張されました。

#### データレイク分析

- Icebergのメタデータキャッシュとアクセスを最適化し、クエリのパフォーマンスを向上させました。
- データレイクのデータキャッシュ（Data Cache）機能を最適化し、データレイクのパフォーマンスをさらに向上させました。

#### ストレージ、インポート、およびクエリ

- [大算子落盤（Spill）](../administration/spill_to_disk.md)機能が正式にサポートされ、一部のブロッキング演算子の中間結果をディスクに保存することができます。大算子落盤機能を有効にすると、集約、ソート、または結合演算子を含むクエリの場合、これらの演算子の中間結果がディスクキャッシュに保存され、メモリ使用量を減らし、クエリの失敗を回避できます。
- カーディナリティを保持するJOINテーブルのトリミングをサポートします。多くのテーブルが含まれるスターモデル（SSBなど）やスノーフレークモデル（TPC-Hなど）のモデリングで、クエリが一部のテーブルのみを関連付ける場合、不要なテーブルをトリミングしてJOINのパフォーマンスを向上させることができます。
- [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)ステートメントを使用して主キーモデルのテーブルを部分的に更新する場合、列モードを有効にすることができます。これは、少数の列を更新するが行数が多いシナリオに適用され、更新パフォーマンスが10倍向上します。
- 統計情報の収集を最適化し、インポートに与える影響を軽減し、収集パフォーマンスを向上させました。
- 並列マージアルゴリズムを最適化し、全ソートシナリオで最大2倍のパフォーマンス向上が可能です。
- クエリのロジックを最適化し、DBロックに依存しないようにしました。
- ダイナミックパーティションで年単位のパーティション粒度をサポートしました。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQLステートメントと関数

- 条件関数case、coalesce、if、ifnull、nullifがARRAY、MAP、STRUCT、JSONの型をサポートします。
- 次のArray関数は、ネストされた構造型MAP、STRUCT、ARRAYをサポートします：
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice、array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 次のArray関数は、Fast Decimal型をサポートします：
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### 問題の修正

以下の問題が修正されました：

- Routine Loadの実行時に、Kafkaの再接続リクエストを正常に処理できない問題を修正しました。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- 複数のテーブルを含み、`WHERE`句があるSQLクエリの場合、これらのSQLクエリの意味は同じですが、指定されたテーブルの順序が異なる場合、一部のSQLクエリは関連するマテリアライズドビューの使用に変換できない問題を修正しました。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- `GROUP BY`句を含むクエリは、重複したデータ結果を返す問題を修正しました。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- lead()またはlag()関数の呼び出しにより、BEが予期しない終了をする可能性がある問題を修正しました。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- External Catalog外部テーブルに基づいて作成されたマテリアライズドビューの一部のパーティションクエリの再書き換えに失敗する問題を修正しました。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- SQLステートメントにバックスラッシュ（`\`）とセミコロン（`;`）が同時に含まれる場合に解析エラーが発生する問題を修正しました。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- マテリアライズドビューを削除した後、元のテーブルのデータがクリア（Truncate）されない問題を修正しました。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 動作の変更

- ストレージと計算の分離アーキテクチャでテーブルを削除する際の`storage_cache_ttl`パラメータは、キャッシュが一杯になった場合にLRUアルゴリズムで削除されるようになりました。
- BEの設定項目`disable_storage_page_cache`、`alter_tablet_worker_count`、およびFEの設定項目`lake_compaction_max_tasks`は、静的パラメータから動的パラメータに変更されました。
- BEの設定項目`block_cache_checksum_enable`のデフォルト値が`true`から`false`に変更されました。
- BEの設定項目`enable_new_load_on_memory_limit_exceeded`のデフォルト値が`false`から`true`に変更されました。
- FEの設定項目`max_running_txn_num_per_db`のデフォルト値が`100`から`1000`に変更されました。
- FEの設定項目`http_max_header_size`のデフォルト値が`8192`から`32768`に変更されました。
- FEの設定項目`tablet_create_timeout_second`のデフォルト値が`1`から`10`に変更されました。
- FEの設定項目`max_routine_load_task_num_per_be`のデフォルト値が`5`から`16`に変更されました。また、ルーチンロードタスクの数が多い場合、エラーが発生した場合にメッセージが表示されます。
- FEの設定項目`quorom_publish_wait_time_ms`は`quorum_publish_wait_time_ms`に名称が変更され、`async_load_task_pool_size`は`max_broker_load_job_concurrency`に名称が変更されました。
- CNの設定項目`thrift_port`は`be_port`に名称が変更されました。
- 廃止されたBEの設定項目`routine_load_thread_pool_size`は、FEの設定項目`max_routine_load_task_num_per_be`によって完全に制御されるようになりました。
- 廃止されたBEの設定項目`txn_commit_rpc_timeout_ms`とシステム変数`tx_visible_wait_timeout`。
- 廃止されたFEの設定項目`max_broker_concurrency`、`load_parallel_instance_num`。
- 廃止されたFEの設定項目`max_routine_load_job_num`は、`max_routine_load_task_num_per_be`を使用して各BEノードでサポートされるルーチンロードタスクの最大数を動的に判断し、タスクが失敗した場合にアドバイスを表示します。
- Routine Load ジョブに `task_consume_second` と `task_timeout_second` の2つの属性が追加され、個々の Routine Load インポートジョブ内のタスクに適用され、より柔軟になりました。これら2つの属性がジョブで設定されていない場合は、FE の設定項目 `routine_load_task_consume_second` と `routine_load_task_timeout_second` の設定を使用します。
- [リソースグループ](../administration/resource_group.md)機能がデフォルトで有効になり、そのためセッション変数 `enable_resource_group` は廃止されました。
- 以下の予約済みキーワードが追加されました：COMPACTION、TEXT。
