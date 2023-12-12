---
displayed_sidebar: "日本語"
---

# StarRocks バージョン 3.1

## 3.1.5

リリース日: 2023年11月28日

### 新機能

- StarRocks 共有データクラスターの CN ノードは、データのエクスポートをサポートします。[#34018](https://github.com/StarRocks/starrocks/pull/34018)

### 改善

- システムデータベース `INFORMATION_SCHEMA` の [`COLUMNS`](../reference/information_schema/columns.md) ビューは、ARRAY、MAP、および STRUCT カラムを表示できます。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- LZO を使用して圧縮され、[Hive](../data_source/catalog/hive_catalog.md) に保存されている Parquet、ORC、および CSV 形式のファイルに対するクエリをサポートします。[#30923](https://github.com/StarRocks/starrocks/pull/30923)  [#30721](https://github.com/StarRocks/starrocks/pull/30721)
- 自動的にパーティションされたテーブルの指定されたパーティションへの更新をサポートします。指定されたパーティションが存在しない場合、エラーが返されます。[#34777](https://github.com/StarRocks/starrocks/pull/34777)
- 特定のテーブルとビュー（これらのマテリアライズドビューが作成されたテーブルやビュー、およびこれらのマテリアライズドビューと関連する他のテーブルやマテリアライズドビューを含む）に対して Schema Change 操作、および Swap、Drop 操作が実行される場合、マテリアライズドビューの自動リフレッシュをサポートします。[#32829](https://github.com/StarRocks/starrocks/pull/32829)
- ビットマップ関連の一部の操作のパフォーマンスを最適化しました。具体的には以下の点を実施しました:
  - ネストされたループ結合を最適化しました。[#340804](https://github.com/StarRocks/starrocks/pull/34804)  [#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor` 関数を最適化しました。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - ビットマップパフォーマンスを最適化し、メモリ消費を削減するために Copy on Write をサポートします。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### バグ修正

以下の問題を修正しました:

- Broker Load ジョブでフィルタリング条件が指定されている場合に、特定の状況下でデータロード中に BE がクラッシュする問題を修正しました。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTS を実行すると不明なエラーが報告される問題を修正しました。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 式ベースの自動パーティショニングを使用するテーブルにデータをロードすると、"Error: The row create partition failed since Runtime error: failed to analyse partition value" エラーが発生する問題を修正しました。[#33513](https://github.com/StarRocks/starrocks/pull/33513)
- クエリの実行時に "get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction" エラーが返される問題を修正しました。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- StarRocks 共有データクラスターで Iceberg や Hive テーブルに対するクエリを実行すると、BE がクラッシュする問題を修正しました。[#34682](https://github.com/StarRocks/starrocks/pull/34682)
- データロード中に複数のパーティションが自動的に作成される場合、ロードされたデータが一致しないパーティションに書き込まれることがある問題を修正しました。[#34731](https://github.com/StarRocks/starrocks/pull/34731)
- プライマリキー テーブルに永続的インデックスが有効になっている状態で長い時間、頻繁なデータロードを行うと、BE がクラッシュする問題を修正しました。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリに対して "Exception: java.lang.IllegalStateException: null" エラーが返される問題を修正しました。[#33535](https://github.com/StarRocks/starrocks/pull/33535)
- `show proc '/current_queries';` を実行している最中にクエリの実行が開始されると、BE がクラッシュする問題を修正しました。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 大量のデータをプライマリキーテーブルにロードするとエラーが発生する問題を修正しました。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- StarRocks を v2.4 以前から後のバージョンにアップグレードすると、コンパクションスコアが予期せず上昇する問題を修正しました。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- `INFORMATION_SCHEMA` を MariaDB ODBC ドライバを使用してクエリすると、`schemata` ビューで返される `CATALOG_NAME` 列が `null` 値のみを保持する問題を修正しました。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- 異常なデータがロードされたことにより FE がクラッシュし、再起動できない問題を修正しました。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- Stream Load ジョブが **PREPARD** 状態でスキーマ変更が実行されると、そのジョブによってロードされるソースデータの一部が失われる問題を修正しました。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- HDFS ストレージパスの末尾に2つ以上のスラッシュ (`/`) が含まれていると、HDFS からデータのバックアップとリストアが失敗する問題を修正しました。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- セッション変数 `enable_load_profile` を `true` に設定すると、Stream Load ジョブが失敗しやすくなる問題を修正しました。[#34544](https://github.com/StarRocks/starrocks/pull/34544)
- プライマリキーテーブルに列モードで部分的な更新を実行すると、テーブルの一部のタブレットでその複製間にデータの不整合が表示される問題を修正しました。[#34555](https://github.com/StarRocks/starrocks/pull/34555)
- ALTER TABLE ステートメントを使用して追加された `partition_live_number` プロパティが有効にならない問題を修正しました。[#34842](https://github.com/StarRocks/starrocks/pull/34842)
- FE が起動に失敗し、エラー "failed to load journal type 118" を報告する問題を修正しました。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- FE パラメータ `recover_with_empty_tablet` を `true` に設定すると、FE がクラッシュする問題を修正しました。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- レプリカ操作の再生に失敗すると、FE がクラッシュする問題を修正しました。[#32295](https://github.com/StarRocks/starrocks/pull/32295)

### 互換性変更

#### パラメータ

- FE 構成項目 [`enable_statistics_collect_profile`](../administration/Configuration.md#enable_statistics_collect_profile) を追加しました。これは、統計クエリのプロファイルを生成するかどうかを制御します。デフォルト値は `false` です。[#33815](https://github.com/StarRocks/starrocks/pull/33815)
- FE 構成項目 [`mysql_server_version`](../administration/Configuration.md#mysql_server_version) は変更可能になりました。新しい設定は、FE の再起動を必要とせずに現在のセッションで有効になります。[#34033](https://github.com/StarRocks/starrocks/pull/34033)
- StarRocks 共有データクラスターのプライマリキーテーブルのコンパクションがマージできるデータの最大比率を制御する BE/CN 構成項目 [`update_compaction_ratio_threshold`](../administration/Configuration.md#update_compaction_ratio_threshold) を追加しました。デフォルト値は `0.5` です。単一のタブレットが過度に大きくなる場合は、この値を縮小することを推奨します。StarRocks 共有ノードクラスターの場合、プライマリキーテーブルのコンパクションがマージできるデータの比率は引き続き自動的に調整されます。[#35129](https://github.com/StarRocks/starrocks/pull/35129)

#### システム変数

- DECIMAL 型から STRING 型へのデータ変換を CBO がどのように行うかを制御するセッション変数 `cbo_decimal_cast_string_strict` を追加しました。この変数が `true` に設定されている場合、v2.5.x およびそれ以降のバージョンで組み込まれたロジックが優先され、システムは厳密な変換を実装します（つまり、システムは生成された文字列を切り捨て、スケール長に基づいて 0 を埋めます）。この変数が `false` に設定されている場合、v2.5.x より前のバージョンで組み込まれたロジックが優先され、システムはすべての有効な桁を生成するための文字列を処理します。デフォルト値は `true` です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- DECIMAL 型データと STRING 型データのデータ比較に使用されるデータ型を指定するセッション変数 `cbo_eq_base_type` を追加しました。デフォルト値は `VARCHAR` ですが、`DECIMAL` も有効な値です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- セッション変数 `big_query_profile_second_threshold` を追加しました。セッション変数 [`enable_profile`](../reference/System_variable.md#enable_profile) が `false` に設定されており、クエリに要する時間が `big_query_profile_second_threshold` 変数で指定された閾値を超えると、そのクエリのプロファイルが生成されます。[#33825](https://github.com/StarRocks/starrocks/pull/33825)

## 3.1.4
リリース日: 2023年11月2日

### 新機能

- 共有データStarRocksクラスタで作成されたプライマリキーテーブルにソートキーのサポートを追加しました。
- 非同期マテリアライズドビューのパーティション式を指定するためにstr2date関数を使用するサポートを追加しました。これにより、外部カタログに存在するテーブルで作成された非同期マテリアライズドビューの増分更新とクエリの書き換えを促進します。[#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- `enable_query_tablet_affinity`という新しいセッション変数を追加しました。これは、複数のクエリを同じタブレットに固定のレプリカに向けるかどうかを制御します。このセッション変数はデフォルトで`false`に設定されています。[#33049](https://github.com/StarRocks/starrocks/pull/33049)
- `is_role_in_session`というユーティリティ関数を追加しました。これは、指定されたロールが現在のセッションでアクティブ化されているかどうかをチェックするために使用されます。ユーザーに付与されたネストされたロールをチェックするのをサポートしています。[#32984](https://github.com/StarRocks/starrocks/pull/32984)
- リソースグループレベルのクエリキュー設定をサポートしました。これは、グローバル変数`enable_group_level_query_queue`（デフォルト値:`false`）によって制御されます。グローバルレベルまたはリソースグループレベルのリソース消費が事前に定義されたしきい値に達すると、新しいクエリはキューに配置され、グローバルレベルのリソース消費とリソースグループレベルのリソース消費の両方がしきい値以下になると実行されます。
  - ユーザーは、各リソースグループごとに`concurrency_limit`を設定して、各BEで許可される最大同時クエリ数を制限できます。
  - ユーザーは、各リソースグループごとに`max_cpu_cores`を設定して、各BEで許可される最大CPU消費を制限できます。
- リソースグループ分類子のために`plan_cpu_cost_range`と`plan_mem_cost_range`の2つのパラメータを追加しました。
  - `plan_cpu_cost_range`: システムによって推定されるCPU消費範囲。デフォルト値`NULL`は制限がないことを示します。
  - `plan_mem_cost_range`: システムによって推定されるメモリ消費範囲。デフォルト値`NULL`は制限がないことを示します。

### 改善

- Window関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMPがORDER BY句およびWindow句をサポートしました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- クエリ中にDECIMALタイプのデータの計算がオーバーフローした場合、NULLの代わりにエラーが返されるように修正されました。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- クエリキュー内で許可される同時クエリの数は、リーダーFEによって管理されるようになりました。各フォロワーFEは、クエリの開始と終了時にリーダーFEに通知します。同時クエリの数がグローバルレベルまたはリソースグループレベルの`concurrency_limit`に達すると、新しいクエリは拒否されるか、キューに配置されます。

### バグの修正

次の問題が修正されました：

- SparkまたはFlinkは、不正確なメモリ使用量の統計情報により、データ読み取りエラーを報告する場合があります。[#30702](https://github.com/StarRocks/starrocks/pull/30702)  [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- メタデータキャッシュのメモリ使用量の統計情報が不正確です。[#31978](https://github.com/StarRocks/starrocks/pull/31978)
- libcurlが呼び出された場合、BEがクラッシュする問題を修正しました。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- Hiveビューに作成されたStarRocksマテリアライズドビューを更新する際に、「java.lang.ClassCastException: com.starrocks.catalog.HiveView cannot be cast to com.starrocks.catalog.HiveMetaStoreTable」というエラーが返される問題を修正しました。[#31004](https://github.com/StarRocks/starrocks/pull/31004)
- ORDER BY句に集約関数が含まれる場合、「java.lang.IllegalStateException: null」というエラーが返される問題を修正しました。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 共有データStarRocksクラスタでは、`information_schema.COLUMNS`にテーブルキーの情報が記録されず、Flink Connectorを使用してデータをロードする際にDELETE操作が実行できない問題を修正しました。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- Flink Connectorを使用してデータをロードする場合、高並行のロードジョブが存在し、HTTPスレッドの数とスキャンスレッドの数の両方が上限に達した場合、ロードジョブが予期せず中断される問題を修正しました。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 数バイトしか含まれないフィールドが追加された場合、データ変更が完了する前にSELECT COUNT(*)を実行すると、「error: invalid field name」というエラーが返される問題を修正しました。[#33243](https://github.com/StarRocks/starrocks/pull/33243)
- クエリキャッシュが有効になった後にクエリの結果が正しくない問題を修正しました。[#32781](https://github.com/StarRocks/starrocks/pull/32781)
- ハッシュ結合中にクエリが失敗し、BEがクラッシュする問題を修正しました。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- BINARYまたはVARBINARYデータ型の`DATA_TYPE`および`COLUMN_TYPE`が、`information_schema.columns`ビューで`unknown`として表示される問題を修正しました。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 挙動の変更

- v3.1.4から、新しいStarRocksクラスタで作成されたプライマリキーテーブルに対して永続的なインデックス作成がデフォルトで有効になりました（これは、既存のStarRocksクラスタが以前のバージョンからv3.1.4にアップグレードされた場合には適用されません）。[#33374](https://github.com/StarRocks/starrocks/pull/33374)
- 新しいFEパラメータ`enable_sync_publish`が追加されました。これはデフォルトで`true`に設定されています。このパラメータが`true`に設定されていると、プライマリキーテーブルへのデータロードのPublishフェーズは、Applyタスクが終了した後に実行結果のみを返します。そのため、ロードジョブの成功メッセージが返された後、ロードされたデータをすぐにクエリできます。ただし、このパラメータを`true`に設定すると、プライマリキーテーブルへのデータロードにはより長い時間がかかる可能性があります。（このパラメータが追加される前は、ApplyタスクはPublishフェーズと非同期でした。.）[#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

リリース日: 2023年9月25日

### 新機能

- 共有データStarRocksクラスタで作成されたプライマリキーテーブルが、共有nothing StarRocksクラスタと同様にローカルディスクにインデックスを永続化するサポートを追加しました。
- 集約関数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)がDISTINCTキーワードとORDER BY句をサポートしました。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Kafka Connector](../loading/Kafka-connector-starrocks.md)、[Flink Connector](../loading/Flink-connector-starrocks.md)、および[Spark Connector](../loading/Spark-connector-starrocks.md)は、プライマリキーテーブルでカラムモードの部分更新をサポートしました。[#28288](https://github.com/StarRocks/starrocks/pull/28288)
- パーティションのデータは時間経過で自動的にクールダウンできます。（この機能は[list partitioning](../table_design/list_partitioning.md)ではサポートされていません。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改善

無効なコメントを持つSQLコマンドを実行すると、MySQLと一貫性のある結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)

### バグの修正

次の問題が修正されました：

- [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)または[HLL](../sql-reference/sql-statements/data-types/HLL.md)データ型が[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md)ステートメントのWHERE句で指定された場合、ステートメントを正しく実行できない問題が修正されました。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- フォロワーFEが再起動された後、CpuCoresの統計情報が最新ではなく、クエリのパフォーマンスが低下する問題が修正されました。[#28472](https://github.com/StarRocks/starrocks/pull/28472) [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- [to_bitmap()](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数の実行コストが誤って計算されていたため、マテリアライズドビューのリライト後に不適切な実行計画が選択される問題が修正されました。[#29961](https://github.com/StarRocks/starrocks/pull/29961)
- 共有データアーキテクチャの特定のユースケースでは、フォロワーFEが再起動された後、フォロワーFEに送信されたクエリは「バックエンドノードが見つかりません。バックエンドノードがダウンしているかどうかを確認してください」というエラーが返されます。 [#28615](https://github.com/StarRocks/starrocks/pull/28615)
- [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して変更されているテーブルに連続してデータがロードされている場合、「Tablet is in error state」エラーがスローされる可能性があります。 [#29364](https://github.com/StarRocks/starrocks/pull/29364)
- `ADMIN SET FRONTEND CONFIG` コマンドを使用してFE動的パラメータ `max_broker_load_job_concurrency` を変更しても効果がありません。 [#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- [date_diff()](../sql-reference/sql-functions/date-time-functions/date_diff.md) 関数の時間単位が定数であるが日付が定数でない場合、BEがクラッシュします。 [#29937](https://github.com/StarRocks/starrocks/issues/29937)
- 共有データアーキテクチャでは、非同期ロードを有効にした後、自動パーティショニングが効果を持ちません。 [#29986](https://github.com/StarRocks/starrocks/issues/29986)
- ユーザーが[CREATE TABLE LIKE](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md) ステートメントを使用してプライマリキー・テーブルを作成すると、「予期しない例外：不明なプロパティ：{persistent_index_type=LOCAL}」エラーがスローされます。 [#30255](https://github.com/StarRocks/starrocks/pull/30255)
- BEが再起動された後にプライマリキー・テーブルを復元すると、メタデータの不一致が発生します。 [#30135](https://github.com/StarRocks/starrocks/pull/30135)
- プライマリキー・テーブルにデータをロードし、切り捨て操作とクエリが同時に実行される場合、特定のケースで「java.lang.NullPointerException」というエラーがスローされます。 [#30573](https://github.com/StarRocks/starrocks/pull/30573)
- 材料化ビューの作成ステートメントで述語式が指定されている場合、その材料化ビューのリフレッシュ結果が正しくありません。 [#29904](https://github.com/StarRocks/starrocks/pull/29904)
- ユーザーがStarRocksクラスタをv3.1.2にアップグレードした後、アップグレード前に作成されたテーブルのストレージボリュームのプロパティが「null」にリセットされます。 [#30647](https://github.com/StarRocks/starrocks/pull/30647)
- タブレットメタデータでチェックポイントとリストアを同時に実行すると、一部のタブレットレプリカが紛失し、取得できなくなります。 [#30603](https://github.com/StarRocks/starrocks/pull/30603)
- CloudCanalを使用してデータをロードし、`NOT NULL` に設定されたがデフォルト値が指定されていないテーブル列にデータをロードした場合、「Unsupported dataFormat value is : \N」エラーがスローされます。 [#30799](https://github.com/StarRocks/starrocks/pull/30799)

### 動作の変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数を使用する場合、ユーザーはセパレータを宣言するために SEPARATOR キーワードを使用しなければなりません。
- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数によって返される文字列のデフォルト最大長を制御するセッション変数 [`group_concat_max_len`](../reference/System_variable.md#group_concat_max_len) のデフォルト値が、無制限から `1024` に変更されました。
- [プレビュー] [Paimonカタログ](../data_source/catalog/paimon_catalog.md)を使用してApache Paimonに格納されたストリーミングデータに対する分析を行うことができます。

#### ストレージエンジン、データ取り込み、およびクエリ

- [式パーティショニング](../table_design/expression_partitioning.md)への自動パーティショニングをアップグレードしました。ユーザーは単純なパーティション式（時間関数式または列式のいずれか）を使用してテーブル作成時にパーティショニング方法を指定するだけで、StarRocksはデータの特性とパーティション式で定義されたルールに基づいて自動的にパーティションを作成します。このパーティション作成方法はほとんどのシナリオに適しており、柔軟で利用しやすいです。
- [リストパーティショニング](../table_design/list_partitioning.md)をサポートしています。特定の列の事前に定義された値のリストに基づいてデータをパーティション分割し、クエリを加速し、明確にカテゴリ分類されたデータをより効率的に管理できます。
- `Information_schema`データベースに新しい`loads`というテーブルを追加しました。ユーザーは`loads`テーブルから[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)および[Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md)ジョブの結果をクエリできます。
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md)ジョブでフィルタリングされた不適格なデータ行を記録することをサポートしています。ユーザーはロードジョブで`log_rejected_record_num`パラメータを使用して記録できるデータ行の最大数を指定できます。
- [ランダムバケット分割](../table_design/Data_distribution.md#how-to-choose-the-bucketing-columns)をサポートしています。この機能により、ユーザーはテーブル作成時にバケット化列を構成する必要がなく、StarRocksはデータをランダムにバケットに割り当てます。StarRocks v2.5.7以降で提供されているバケットの数（`BUCKETS`）を自動的に設定する機能と併用することで、バケットの構成を考慮する必要がなくなり、テーブル作成文が大幅に簡略化されます。ただし、ビッグデータや高パフォーマンスを要求するシナリオでは引き続きハッシュバケットを使用することを推奨しており、これによりバケットのプルーニングを使用してクエリを加速できます。
- [INSERT INTO](../loading/InsertInto.md)でテーブル関数FILES()を使用して、AWS S3に格納されているParquet形式またはORC形式のデータファイルのデータを直接ロードすることをサポートしています。FILES()関数は、テーブルスキーマを自動的に推測できるため、データのロード前に外部カタログまたはファイル外部テーブルを作成する必要がなく、データのロードプロセスが大幅に簡素化されます。
- [生成列](../sql-reference/sql-statements/generated_columns.md)をサポートしています。生成列機能では、StarRocksが列式の値を自動的に生成および格納し、クエリの書き換えを自動的に行ってクエリのパフォーマンスを向上させることができます。
- [Spark connector](../loading/Spark-connector-starrocks.md)を使用して、SparkからStarRocksにデータをロードすることをサポートしています。[Spark Load](../loading/SparkLoad.md)と比較して、Spark connectorはより包括的な機能を提供しています。ユーザーはSparkジョブを定義してデータのETL操作を実行し、Spark connectorがSparkジョブ内のシンクとして機能します。
- [MAP](../sql-reference/sql-statements/data-types/Map.md)および[STRUCT](../sql-reference/sql-statements/data-types/STRUCT.md)データ型の列にデータをロードすることをサポートしており、ARRAY、MAP、およびSTRUCTでFast Decimal値をネストさせることをサポートしています。

#### SQLリファレンス

- 以下のストレージボリュームに関連するステートメントを追加しました: [CREATE STORAGE VOLUME](../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md), [ALTER STORAGE VOLUME](../sql-reference/sql-statements/Administration/ALTER_STORAGE_VOLUME.md), [DROP STORAGE VOLUME](../sql-reference/sql-statements/Administration/DROP_STORAGE_VOLUME.md), [SET DEFAULT STORAGE VOLUME](../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md), [DESC STORAGE VOLUME](../sql-reference/sql-statements/Administration/DESC_STORAGE_VOLUME.md), [SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW_STORAGE_VOLUMES.md).
- [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)を使用してテーブルコメントを変更することをサポートしています。[#21035](https://github.com/StarRocks/starrocks/pull/21035)
- 以下の関数を追加しました:
  - 構造体関数: [struct (row)](../sql-reference/sql-functions/struct-functions/row.md), [named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)
  - マップ関数: [str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md), [map_concat](../sql-reference/sql-functions/map-functions/map_concat.md), [map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md), [element_at](../sql-reference/sql-functions/map-functions/element_at.md), [distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md), [cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)
  - ハイアーオーダーマップ関数: [map_filter](../sql-reference/sql-functions/map-functions/map_filter.md), [map_apply](../sql-reference/sql-functions/map-functions/map_apply.md), [transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md), [transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)
  - 配列関数: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)で`ORDER BY`をサポート, [array_generate](../sql-reference/sql-functions/array-functions/array_generate.md), [element_at](../sql-reference/sql-functions/array-functions/element_at.md), [cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)
  - ハイアーオーダー配列関数: [all_match](../sql-reference/sql-functions/array-functions/all_match.md), [any_match](../sql-reference/sql-functions/array-functions/any_match.md)
  - 集約関数: [min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md), [percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)
  - テーブル関数: [FILES](../sql-reference/sql-functions/table-functions/files.md), [generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)
  - 日付関数: [next_day](../sql-reference/sql-functions/date-time-functions/next_day.md), [previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md), [last_day](../sql-reference/sql-functions/date-time-functions/last_day.md), [makedate](../sql-reference/sql-functions/date-time-functions/makedate.md), [date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)
  - ビットマップ関数: [bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md), [bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)
  - 数学関数: [cosine_similarity](../sql-reference/sql-functions/math-functions/cos_similarity.md), [cosine_similarity_norm](../sql-reference/sql-functions/math-functions/cos_similarity_norm.md)

#### 権限とセキュリティ

ストレージボリュームに関連する[権限項目](../administration/privilege_item.md#storage-volume)と外部カタログに関連する[権限項目](../administration/privilege_item.md#catalog)を追加し、これらの権限を付与および取り消すために[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)および[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)を使用することをサポートしています。

### 改善

#### 共有データクラスター

共有データStarRocksクラスター内のデータキャッシュを最適化しました。最適化されたデータキャッシュにより、ホットデータの範囲を指定できます。また、冷たいデータに対するクエリがローカルディスクキャッシュを占有することを防ぎ、ホットデータに対するクエリのパフォーマンスを確保できます。

#### マテリアライズドビュー

- 非同期マテリアライズドビューの作成を最適化しました:
  - ランダムバケット分割をサポートしています。ユーザーがバケット化列を指定しない場合、StarRocksはデフォルトでランダムバケット分割を採用します。
  - `ORDER BY`を使用してソートキーを指定することをサポートしています。
  - `colocate_group`、`storage_medium`、`storage_cooldown_time`などの属性を指定できます。
  - セッション変数を使用できます。ユーザーは`properties("session.<variable_name>" = "<value>")`構文を使用してこれらの変数を構成し、柔軟にビューリフレッシュ戦略を調整できます。
  - すべての非同期マテリアライズドビューにスピル機能を有効にし、デフォルトでクエリのタイムアウト期間を1時間に設定します。
  - ビューに基づいてマテリアライズドビューを作成することをサポートしています。これにより、データモデリングシナリオでマテリアライズドビューを柔軟に使用できるため、ユーザーは必要に応じてビューとマテリアライズドビューを使用して階層モデリングを実装できます。
- 非同期マテリアライズドビューによるクエリの書き換えを最適化しました:
  - ステールリライトをサポートしており、一定の時間間隔内にリフレッシュされないマテリアライズドビューを、マテリアライズドビューのベーステーブルが更新されたかどうかに関係なくクエリの書き換えに使用できます。ユーザーは、マテリアライズドビュー作成時に`mv_rewrite_staleness_second`プロパティを使用して時間間隔を指定できます。
  - Hiveカタログテーブルで作成されたマテリアライズドビューに対するView Delta Joinクエリの書き換えをサポートしており、主キーと外部キーの定義が必要です。
  - union操作を含むクエリの書き換えメカニズムを最適化し、joinやCOUNT DISTINCT、time_sliceなどの関数を含むクエリの書き換えをサポートしています。
- 非同期マテリアライズドビューのリフレッシュを最適化しました:
  - Hiveカタログテーブルに作成されたマテリアライズドビューを更新するメカニズムを最適化しました。StarRocksは現在、パーティションレベルのデータ変更を認識できるようになり、自動更新時にデータ変更があるパーティションのみを更新します。
  - `REFRESH MATERIALIZED VIEW WITH SYNC MODE`構文を使用して同期的にマテリアライズドビューの更新タスクを呼び出すことをサポートしています。
- 非同期マテリアライズドビューの利用を強化しました:
  - `ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}`を使用してマテリアライズドビューを有効化または無効化することをサポートしています。 (「INACTIVE」状態の) 無効化されたマテリアライズドビューは更新されたり、クエリの書き換えに使用されたりすることはできませんが、直接クエリを行うことはできます。
  - `ALTER MATERIALIZED VIEW SWAP WITH`を使用して2つのマテリアライズドビューを交換することをサポートしています。ユーザーは新しいマテリアライズドビューを作成し、既存のマテリアライズドビューと原子的に交換することで、既存のマテリアライズドビューのスキーマ変更を実装することができます。
- 同期的なマテリアライズドビューを最適化しました:
  - SQLヒント`[_SYNC_MV_]`を使用して同期的なマテリアライズドビューに直接クエリを行うことをサポートし、一部のクエリがまれな状況で適切に書き換えられない問題を回避できるようにしました。
  - `CASE-WHEN`、`CAST`、数学演算などの式をさらにサポートし、マテリアライズドビューをさまざまなビジネスシナリオに適したものにしました。

#### データレイク解析

- Icebergのメタデータキャッシュとアクセスを最適化し、Icebergデータのクエリパフォーマンスを向上させました。
- データキャッシュをさらに最適化し、データレイク解析のパフォーマンスを向上させました。

#### ストレージエンジン、データ取り込み、およびクエリ

- [spill](../administration/spill_to_disk.md)機能の一般提供を発表しました。この機能では、一部のブロック演算子の中間計算結果をディスクにスパイルすることがサポートされます。スパイル機能を有効にすると、クエリに集約、ソート、または結合演算子が含まれている場合、StarRocksは演算子の中間計算結果をディスクにキャッシュし、メモリ消費を減らし、メモリ制限によるクエリの失敗を最小限に抑えることができます。
- カーディナリティ保存結合でのプルーニングをサポートしました。ユーザーがスターシェーマ（たとえばSSB）やスノーフレークシェーマ（たとえばTCP-H）で組織された大量のテーブルを維持しており、これらのテーブルのうちわずかしかクエリしない場合、この機能は不要なテーブルを削除して結合のパフォーマンスを向上させます。
- カラムモードでの部分更新をサポートしました。主キーテーブルの部分更新を行う際に、[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)文を使用してカラムモードを有効にすることができます。カラムモードは、列の数は少ないが行の数は多い更新に適しており、最大10倍の更新パフォーマンス向上が見込まれます。
- CBOの統計情報の収集を最適化しました。これにより、データ取り込みに対する統計情報収集の影響が軽減され、統計情報の収集パフォーマンスが向上します。
- 全体のパフォーマンスを最大2倍向上させるために、マージアルゴリズムを最適化しました。
- クエリロジックを最適化し、データベースロックへの依存を削減しました。
- 動的パーティショニングは、さらにパーティショニング単位を年に対応します。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQLリファレンス

- 条件関数(case、coalesce、if、ifnull、nullif)がARRAY、MAP、STRUCT、JSONデータ型をサポートしています。
- 次の配列関数がネストされた型MAP、STRUCT、ARRAYをサポートしています:
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice、array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 次の配列関数がFast Decimalデータ型をサポートしています:
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### バグ修正

以下の問題が修正されました:

- ルーチンロードジョブのKafkaへの再接続要求を正しく処理できません。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- 複数のテーブルを含むSQLクエリに`WHERE`句が含まれている場合、これらのSQLクエリが同じセマンティクスを持ち、各SQLクエリのテーブルの順序が異なる場合、関連するマテリアライズドビューからの利益を得るためにこれらのSQLクエリの一部が正しく書き換えられないことがあります。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- `GROUP BY`句を含むクエリに重複したレコードが返されます。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- lead()またはlag()関数の呼び出しはBEのクラッシュを引き起こす場合があります。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 外部カタログテーブルに作成されたマテリアライズドビューに基づく部分パーティションクエリの書き換えが失敗する場合があります。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- バックスラッシュ(`\`)とセミコロン(`;`)を含むSQL文が正しく解析されない場合があります。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- マテリアライズドビューを削除すると、そのテーブルは切り捨てることができません。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 挙動の変更

- 共有データStarRocksクラスタのテーブル作成構文から`storage_cache_ttl`パラメータが削除されました。現在は、ローカルキャッシュのデータはLRUアルゴリズムに基づいて削除されます。
- BE構成項目`disable_storage_page_cache`および`alter_tablet_worker_count`、FE構成項目`lake_compaction_max_tasks`のデフォルト値が変更可能なパラメータから変更不可能なパラメータに変更されました。
- BE構成項目`block_cache_checksum_enable`のデフォルト値は`true`から`false`に変更されました。
- BE構成項目`enable_new_load_on_memory_limit_exceeded`のデフォルト値は`false`から`true`に変更されました。
- FE構成項目`max_running_txn_num_per_db`のデフォルト値は`100`から`1000`に変更されました。
- FE構成項目`http_max_header_size`のデフォルト値は`8192`から`32768`に変更されました。
- FE構成項目`tablet_create_timeout_second`のデフォルト値は`1`から`10`に変更されました。
- FE構成項目`max_routine_load_task_num_per_be`のデフォルト値は`5`から`16`に変更され、大量のルーチンロードタスクが作成された場合、エラー情報が返されるようになりました。
- FE構成項目`quorom_publish_wait_time_ms`は`quorum_publish_wait_time_ms`に、FE構成項目`async_load_task_pool_size`は`max_broker_load_job_concurrency`に名称が変更されました。
- BE構成項目`routine_load_thread_pool_size`は廃止されました。現在、ルーチンロードスレッドプールサイズはFE構成項目`max_routine_load_task_num_per_be`によって制御されます。
- BE構成項目`txn_commit_rpc_timeout_ms`およびシステム変数`tx_visible_wait_timeout`は廃止されました。
- FE構成項目`max_broker_concurrency`および`load_parallel_instance_num`は廃止されました。
- FE構成項目`max_routine_load_job_num`は廃止されました。現在、StarRocksは`max_routine_load_task_num_per_be`パラメータに基づいて各個別のBEノードがサポートするルーチンロードタスクの最大数を動的に推論し、タスクの失敗に関する提案を行います。
- CN構成項目`thrift_port`は`be_port`に名称が変更されました。
- ルーチンロードジョブの新しいプロパティである`task_consume_second`および`task_timeout_second`が追加され、ルーチンロードジョブ内の個々のロードタスクのデータを消費する時間の最大量とタイムアウト期間を制御するようになりました。ユーザーがこれら2つのプロパティをルーチンロードジョブで指定しない場合、FE構成項目`routine_load_task_consume_second`および`routine_load_task_timeout_second`が優先します。
- セッション変数`enable_resource_group`は廃止されました。なぜならば[v3.1.0](../administration/resource_group.md)以降、[リソースグループ](../administration/resource_group.md)機能がデフォルトで有効化されているからです。
- COMPACTIONおよびTEXTという2つの新しい予約キーワードが追加されました。