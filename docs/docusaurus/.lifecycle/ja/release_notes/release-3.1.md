---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 3.1

## 3.1.5

リリース日: 2023年11月28日

### 新機能

- StarRocks 共有データクラスターのCNノードは今後、データのエクスポートをサポートします。[#34018](https://github.com/StarRocks/starrocks/pull/34018)

### 改善点

- `INFORMATION_SCHEMA`データベースの[`COLUMNS`](../reference/information_schema/columns.md) ビューは、ARRAY、MAP、およびSTRUCTの列を表示できます。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- LZOを使用して圧縮され、[Hive](../data_source/catalog/hive_catalog.md)で保存されているParquet、ORC、およびCSV形式のファイルに対するクエリをサポートします。[#30923](https://github.com/StarRocks/starrocks/pull/30923)　[#30721](https://github.com/StarRocks/starrocks/pull/30721)
- 自動的にパーティションされたテーブルの指定されたパーティションに更新をサポートします。指定されたパーティションが存在しない場合は、エラーが返されます。[#34777](https://github.com/StarRocks/starrocks/pull/34777)
- スワップ、ドロップ、またはスキーマ変更操作がテーブルおよびビュー（これらのビューに関連する他のテーブルおよびマテリアライズドビューを含む）に対して実行されると、マテリアライズドビューの自動リフレッシュをサポートします。[#32829](https://github.com/StarRocks/starrocks/pull/32829)
- 一部のBitmap関連の操作のパフォーマンスを最適化しました：
  - ネストされたループ結合を最適化しました。[#340804](https://github.com/StarRocks/starrocks/pull/34804)　[#35003](https://github.com/StarRocks/starrocks/pull/35003)
  - `bitmap_xor` 関数を最適化しました。[#34069](https://github.com/StarRocks/starrocks/pull/34069)
  - Bitmapのパフォーマンスを最適化してメモリ消費を削減するために、Copy on Write をサポートします。[#34047](https://github.com/StarRocks/starrocks/pull/34047)

### バグ修正

以下の問題を修正しました：

- 特定の状況下でBroker Loadジョブでフィルタリング条件が指定されている場合、データのロード中にBEがクラッシュすることがあります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTSを実行すると不明なエラーが報告されます。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- 式ベースの自動パーティショニングを使用するテーブルにデータがロードされると、「エラー: Runtime error: failed to analyse partition value」というエラーが発生する場合があります。[#33513](https://github.com/StarRocks/starrocks/pull/33513)
- クエリで「get_applied_rowsets failed, tablet updates is in error state: tablet:18849 actual row size changed after compaction」というエラーが返されます。[#33246](https://github.com/StarRocks/starrocks/pull/33246)
- StarRocks 共有ノッシングクラスターでは、IcebergまたはHiveテーブルに対するクエリがBEをクラッシュさせることがあります。[#34682](https://github.com/StarRocks/starrocks/pull/34682)
- StarRocks 共有ノッシングクラスターでは、データが自動作成された複数のパーティションにロードされると、時々一致しないパーティションにデータが書き込まれることがあります。[#34731](https://github.com/StarRocks/starrocks/pull/34731)
- Persistent indexが有効な主キーテーブルに長時間頻繁にデータをロードすると、BEがクラッシュする場合があります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリで「Exception: java.lang.IllegalStateException: null」というエラーが返されます。[#33535](https://github.com/StarRocks/starrocks/pull/33535)
- `show proc '/current_queries';` を実行している間にクエリが開始されると、BEがクラッシュする場合があります。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- 大量のデータがPersistent indexが有効な主キーテーブルにロードされると、エラーがスローされることがあります。[#34352](https://github.com/StarRocks/starrocks/pull/34352)
- StarRocksがv2.4またはそれ以前のバージョンからアップグレードされると、コンパクションのスコアが予期しないほど上昇することがあります。[#34618](https://github.com/StarRocks/starrocks/pull/34618)
- MariaDB ODBCデータベースドライバを使用して`INFORMATION_SCHEMA`がクエリされると、`schemata` ビューで返される `CATALOG_NAME` 列が`null` の値しか保持しません。[#34627](https://github.com/StarRocks/starrocks/pull/34627)
- 異常なデータがロードされたため、FEが起動せずにクラッシュします。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- Stream Loadジョブが **PREPARD** 状態でスキーマ変更が実行されると、ジョブによってロードされるソースデータの一部が失われることがあります。[#34381](https://github.com/StarRocks/starrocks/pull/34381)
- HDFSストレージパスの末尾に2つ以上のスラッシュ（`/`）を含めると、HDFSからのデータのバックアップと復元に失敗することがあります。[#34601](https://github.com/StarRocks/starrocks/pull/34601)
- セッション変数 `enable_load_profile` を `true` に設定すると、Stream Loadジョブの失敗がより頻繁に発生する可能性があります。[#34544](https://github.com/StarRocks/starrocks/pull/34544)
- 主キーテーブルの列モードで部分的な更新を実行すると、テーブルの一部のタブレットにレプリカ間でデータの矛盾が表示されることがあります。[#34555](https://github.com/StarRocks/starrocks/pull/34555)
- `ALTER TABLE` ステートメントを使用して追加された `partition_live_number` プロパティが効果を発揮しません。[#34842](https://github.com/StarRocks/starrocks/pull/34842)
- FEが起動に失敗し、「failed to load journal type 118」というエラーが報告されます。[#34590](https://github.com/StarRocks/starrocks/pull/34590)
- FEパラメータ `recover_with_empty_tablet` を `true` に設定すると、FEがクラッシュする可能性があります。[#33071](https://github.com/StarRocks/starrocks/pull/33071)
- レプリカ操作の再生に失敗すると、FEがクラッシュする場合があります。[#32295](https://github.com/StarRocks/starrocks/pull/32295)

### 互換性の変更

#### パラメータ

- 統計クエリのプロファイルを生成するかどうかを制御するFE構成項目 [`enable_statistics_collect_profile`](../administration/Configuration.md#enable_statistics_collect_profile) を追加しました。デフォルト値は `false` です。[#33815](https://github.com/StarRocks/starrocks/pull/33815)
- FE構成項目 [`mysql_server_version`](../administration/Configuration.md#mysql_server_version) は今後変更可能となりました。新しい設定値は、FEを再起動することなく現在のセッションに対して即座に有効になります。[#34033](https://github.com/StarRocks/starrocks/pull/34033)
- Primary Keyテーブルのコンパクションに対してデータの最大のマージ比率を制御するBE/CN構成項目 [`update_compaction_ratio_threshold`](../administration/Configuration.md#update_compaction_ratio_threshold) を追加しました。デフォルト値は `0.5` です。シングルタブレットが過剰に大きくなった場合は、この値を縮小することをお勧めします。StarRocks共有ノッシングクラスターの場合、Primary Keyテーブルのコンパクションがどの程度のデータをマージできるかは引き続き自動的に調整されます。[#35129](https://github.com/StarRocks/starrocks/pull/35129)

#### システム変数

- DECIMAL型のデータをSTRING型に変換する際のCBOの動作を制御するセッション変数 `cbo_decimal_cast_string_strict` を追加しました。この変数が`true`に設定されている場合、v2.5.x以降のバージョンに組み込まれたロジックが有効となり、システムは厳密な変換（つまり、生成された文字列を切り詰めて、スケール長に基づいて0を埋めます）を実行します。この変数が`false`に設定されている場合、v2.5.xより前のバージョンに組み込まれたロジックが有効となり、システムはすべての有効な桁を処理して文字列を生成します。デフォルト値は `true` です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- DECIMAL型データとSTRING型データの比較に使用するデータ型を指定するセッション変数 `cbo_eq_base_type` を追加しました。デフォルト値は `VARCHAR` で、`DECIMAL` も有効な値です。[#34208](https://github.com/StarRocks/starrocks/pull/34208)
- セッション変数 [`enable_profile`](../reference/System_variable.md#enable_profile) が `false` に設定されておりかつクエリの実行にかかる時間が `big_query_profile_second_threshold` 変数で指定された閾値を超えると、そのクエリのためにプロファイルが生成されます。[#33825](https://github.com/StarRocks/starrocks/pull/33825)

## 3.1.4
リリース日: 2023年11月2日

### 新機能

- 共有データStarRocksクラスターで作成されたプライマリキーテーブルに対するソートキーをサポートします。
- 非同期マテリアライズドビューのパーティション式を指定するためにstr2date関数を使用できるようになりました。これにより、外部カタログに存在しSTRING型データをパーティション式として使用しているテーブルに作成された非同期マテリアライズドビューの増分更新とクエリ書き換えが容易になります。[#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- 新しいセッション変数`enable_query_tablet_affinity`が追加されました。この変数は、同じタブレットに対する複数のクエリを固定のレプリカに向けるかどうかを制御します。このセッション変数はデフォルトで`false`に設定されています。[#33049](https://github.com/StarRocks/starrocks/pull/33049)
- `is_role_in_session`というユーティリティ関数が追加されました。この関数は、現在のセッションで指定されたロールがアクティブ化されているかどうかを確認するために使用されます。この関数は、ユーザーに付与されたネストされたロールを確認することができます。[#32984](https://github.com/StarRocks/starrocks/pull/32984)
- リソースグループレベルのクエリキューの設定をサポートします。これはグローバル変数`enable_group_level_query_queue`（デフォルト値:`false`）によって制御されます。グローバルレベルまたはリソースグループレベルのリソース消費が事前に定義された閾値に達すると、新しいクエリはキューに配置され、両方のリソース消費が閾値以下になると実行されます。
  - ユーザーは、各リソースグループに対して`concurrency_limit`を設定して、各BEに許可される最大同時クエリの数を制限できます。
  - ユーザーは、各リソースグループに対して`max_cpu_cores`を設定して、各BEで許可される最大CPU 消費量を制限できます。
- リソースグループ分類子のために`plan_cpu_cost_range`と`plan_mem_cost_range`の2つのパラメータが追加されました。
  - `plan_cpu_cost_range`: システムによって推定されたCPU消費範囲。デフォルト値`NULL`は制限がないことを示します。
  - `plan_mem_cost_range`: システムによって推定されたメモリ消費範囲。デフォルト値`NULL`は制限がないことを示します。

### 改善

- Window関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPは、ORDER BY句とWindow句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- DECIMAL型データのクエリ中に桁あふれが発生した場合、エラーが返されるようになりました。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- クエリキューに許可される同時クエリの数は、各フォロワーFEによって管理されるようになりました。各フォロワーFEは、クエリの開始と終了時にリーダーFEに通知します。同時クエリの数がグローバルレベルまたはリソースグループレベルの`concurrency_limit`に達すると、新しいクエリは拒否されるかキューに配置されます。

### バグ修正

以下の問題が修正されました:

- 正確でないメモリ使用量統計により、SparkまたはFlinkがデータ読み取りエラーを報告する場合があります。[#30702](https://github.com/StarRocks/starrocks/pull/30702)  [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- メタデータキャッシュのメモリ使用量統計が不正確です。[#31978](https://github.com/StarRocks/starrocks/pull/31978)
- libcurlが呼び出されたとき、BEがクラッシュする問題が修正されました。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- Hiveビューに作成されたStarRocksマテリアライズドビューをリフレッシュすると、「java.lang.ClassCastException: com.starrocks.catalog.HiveView cannot be cast to com.starrocks.catalog.HiveMetaStoreTable」というエラーが返される問題が修正されました。[#31004](https://github.com/StarRocks/starrocks/pull/31004)
- ORDER BY句に集約関数が含まれている場合、「java.lang.IllegalStateException: null」というエラーが返される問題が修正されました。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 共有データStarRocksクラスターでは、`information_schema.COLUMNS`にテーブルキーの情報が記録されず、Flink Connectorを使用してデータをロードする際にDELETE操作を実行できない問題が修正されました。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- Flink Connectorを使用してデータをロードする際、非常に同時実行されるロードジョブがあり、HTTPスレッドの数とスキャンスレッドの数の両方が上限に達した場合、ロードジョブが予期せず中断される問題が修正されました。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- データの変更が完了する前に、少数のバイトのフィールドが追加されると、「error: invalid field name」というエラーが返される問題が修正されました。[#33243](https://github.com/StarRocks/starrocks/pull/33243)
- クエリキャッシュが有効になっている後に、クエリ結果が正しくない問題が修正されました。[#32781](https://github.com/StarRocks/starrocks/pull/32781)
- ハッシュ結合中にクエリが失敗し、BEがクラッシュする問題が修正されました。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- BINARYまたはVARBINARYデータ型の`DATA_TYPE`および`COLUMN_TYPE`が`information_schema.columns`ビューで`unknown`と表示される問題が修正されました。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 動作の変更

- v3.1.4以降、新しいStarRocksクラスターで作成されたプライマリキーテーブルに対して、デフォルトで永続インデックス設定が有効になりました（これは、既存のStarRocksクラスターがv3.1.4により以前のバージョンからアップグレードされた場合には適用されません）。[#33374](https://github.com/StarRocks/starrocks/pull/33374)
- デフォルトで`true`に設定される新しいFEパラメータ`enable_sync_publish`が追加されました。このパラメータが`true`に設定されている場合、プライマリキーテーブルへのデータロードの発行フェーズは、Applyタスクが完了した後に実行結果のみを返します。そのため、データが読み込まれた後はロードジョブが正常に終了した直後にクエリ対象となります。ただし、このパラメータを`true`に設定すると、プライマリキーテーブルへのデータロードに時間がかかる場合があります（このパラメータが追加される前は、ApplyタスクはPublishフェーズと非同期でした）。[#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

リリース日: 2023年9月25日

### 新機能

- 共有データStarRocksクラスターで作成されたプライマリキーテーブルが、共有しないStarRocksクラスターと同様に、ローカルディスクに対するインデックス永続化をサポートします。
- 集約関数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)がDISTINCTキーワードとORDER BY句をサポートするようになりました。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Kafka Connector](../loading/Kafka-connector-starrocks.md)、[Flink Connector](../loading/Flink-connector-starrocks.md)、および[Spark Connector](../loading/Spark-connector-starrocks.md)が、プライマリキーテーブルの列モードで部分更新をサポートするようになりました。[#28288](https://github.com/StarRocks/starrocks/pull/28288)
- パーティション内のデータが時間とともに自動的に冷却されるようになりました。（この機能は[list partitioning](../table_design/list_partitioning.md)にはサポートされていません。) [#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改善

無効なコメントを含むSQLコマンドを実行すると、MySQLと一貫した結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)

### バグ修正

以下の問題が修正されました:

- [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md)ステートメントのWHERE句で[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)または[HLL](../sql-reference/sql-statements/data-types/HLL.md)データ型が指定された場合、ステートメントが正しく実行されない問題が修正されました。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- フォロワーFEが再起動された後、CpuCoresの統計が最新でないため、クエリパフォーマンスが低下する問題が修正されました。[#28472](https://github.com/StarRocks/starrocks/pull/28472) [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- [to_bitmap()](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)関数の実行コストが誤って計算されていたため、マテリアライズドビューが書き換えられた後に関数のための適切な実行計画が選択されなかった問題が修正されました。[#29961](https://github.com/StarRocks/starrocks/pull/29961)
- 共有データアーキテクチャの特定のユースケースでは、フォロワーFEが再起動された後、フォロワーFEに送信されたクエリがエラーになり、"バックエンドノードが見つかりません。バックエンドノードがダウンしているかどうかをチェックしてください"というエラーが表示されることがあります。[#28615](https://github.com/StarRocks/starrocks/pull/28615)

- [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して変更中のテーブルに連続してデータをロードすると、「Tablet is in error state」というエラーが発生する可能性があります。[#29364](https://github.com/StarRocks/starrocks/pull/29364)

- `ADMIN SET FRONTEND CONFIG` コマンドを使用して FE 動的パラメータ `max_broker_load_job_concurrency` を変更しても効果がありません。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)

- [date_diff()](../sql-reference/sql-functions/date-time-functions/date_diff.md) 関数の時間単位が定数であるが、日付が定数でない場合、BEがクラッシュすることがあります。[#29937](https://github.com/StarRocks/starrocks/issues/29937)

- 共有データアーキテクチャでは、非同期ロードが有効になっている場合、自動分割が効果を発揮しません。[#29986](https://github.com/StarRocks/starrocks/issues/29986)

- ユーザーが [CREATE TABLE LIKE](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md) ステートメントを使用して主キーのテーブルを作成すると、「予期しない例外：未知のプロパティ：{persistent_index_type=LOCAL}」というエラーが発生します。[#30255](https://github.com/StarRocks/starrocks/pull/30255)

- BEが再起動されると、主キーテーブルのリストアによりメタデータの不整合が発生します。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

- ユーザーが truncate 操作およびクエリが同時に実行される主キーテーブルにデータをロードすると、特定のケースでは "java.lang.NullPointerException" というエラーが発生します。[#30573](https://github.com/StarRocks/starrocks/pull/30573)

- 材料化ビューの作成ステートメントで述語式が指定されている場合、その材料化ビューのリフレッシュ結果が正しくありません。[#29904](https://github.com/StarRocks/starrocks/pull/29904)

- ユーザーがStarRocksクラスタを v3.1.2 にアップグレードした後、アップグレード前に作成されたテーブルのストレージボリュームプロパティが `null` にリセットされます。[#30647](https://github.com/StarRocks/starrocks/pull/30647)

- tabletメタデータでチェックポイントとリストアが同時に実行される場合、一部のtabletレプリカが失われ、取得できなくなります。[#30603](https://github.com/StarRocks/starrocks/pull/30603)

- ユーザーがCloudCanalを使用して、デフォルト値が指定されていないが `NOT NULL` に設定されているテーブル列にデータをロードしようとすると、「Unsupported dataFormat value is: \N」というエラーが発生します。[#30799](https://github.com/StarRocks/starrocks/pull/30799)

### 動作の変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数を使用する場合、ユーザーは SEPARATOR キーワードを使用してセパレータを宣言する必要があります。

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数によって返される文字列のデフォルトの最大長を制御するセッション変数 [`group_concat_max_len`](../reference/System_variable.md#group_concat_max_len) のデフォルト値が、無制限から `1024` に変更されました。
- [Preview] [Paimon catalogs](../data_source/catalog/paimon_catalog.md) を使用して、Apache Paimon に格納されたストリーミングデータの分析を実行することをサポートしています。

#### ストレージエンジン、データ取り込み、およびクエリ

- [式パーティショニング](../table_design/expression_partitioning.md) への自動パーティショニングのアップグレード。ユーザーは、テーブルの作成時に単純なパーティション式（時間関数式または列式のいずれか）を使用するだけでよく、StarRocks がデータの特性とパーティション式で定義されたルールに基づいて自動的にパーティションを作成します。このパーティション作成方法は、ほとんどのシナリオに適しており、柔軟性が高くユーザーフレンドリーです。
- [リストパーティショニング](../table_design/list_partitioning.md) をサポート。特定の列の事前定義された値のリストに基づいてデータがパーティション分割され、クエリの高速化や明確にカテゴライズされたデータの効率的な管理を実現できます。
- `Information_schema` データベースに新しい `loads` という名前のテーブルを追加。ユーザーは、`loads` テーブルから [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) および [Insert](../sql-reference/sql-statements/data-manipulation/INSERT.md) ジョブの結果をクエリできます。
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および [Spark Load](../sql-reference/sql-statements/data-manipulation/SPARK_LOAD.md) ジョブでフィルタリングされた適格でないデータ行のログ記録をサポート。ユーザーは、ロードジョブで `log_rejected_record_num` パラメータを使用して、ログ記録できるデータ行の最大数を指定できます。
- [ランダムバケット化](../table_design/Data_distribution.md#how-to-choose-the-bucketing-columns) をサポート。この機能により、ユーザーはテーブルの作成時にバケティング列を構成する必要がなく、StarRocks はデータをランダムにバケットに分散させます。この機能を v2.5.7 以降提供しているバケツの数 (`BUCKETS`) を自動的に設定する能力と併せて使用することで、ユーザーはバケット設定を考慮する必要がなくなり、テーブルの作成ステートメントが大幅に簡略化されます。ただし、ビッグデータおよび高パフォーマンス要件のあるシナリオでは、バケットの構成を引き続き検討することをお勧めします。これにより、バケットのプルーニングを使用してクエリを高速化できます。
- [INSERT INTO](../loading/InsertInto.md) で表関数 FILES() を使用して、AWS S3 に格納された Parquet 形式または ORC 形式のデータファイルのデータを直接ロードすることをサポート。FILES() 関数は自動的にテーブルスキーマを推論することができ、これによりデータのロード前に外部カタログやファイル外部テーブルを作成する必要がなくなり、データのロードプロセスが大幅に簡略化されます。
- [生成列](../sql-reference/sql-statements/generated_columns.md) をサポート。生成列機能により、StarRocks は列式の値を自動的に生成して保存し、クエリパフォーマンスを向上させるためにクエリを自動的に書き換えることができます。
- [Spark connector](../loading/Spark-connector-starrocks.md) を使用して、Spark から StarRocks にデータをロードすることをサポート。[Spark Load](../loading/SparkLoad.md) と比較して、Spark connector はより包括的な機能を提供します。ユーザーは、Spark ジョブを定義してデータ上で ETL 操作を実行でき、Spark connector はそのスパークジョブでシンクとして機能します。
- [MAP](../sql-reference/sql-statements/data-types/Map.md) および [STRUCT](../sql-reference/sql-statements/data-types/STRUCT.md) データ型の列にデータをロードすることをサポートし、ARRAY、MAP、および STRUCT 内に Fast Decimal 値をネストすることをサポートします。

#### SQL リファレンス

- 次のストレージボリューム関連ステートメントを追加: [CREATE STORAGE VOLUME](../sql-reference/sql-statements/Administration/CREATE_STORAGE_VOLUME.md), [ALTER STORAGE VOLUME](../sql-reference/sql-statements/Administration/ALTER_STORAGE_VOLUME.md), [DROP STORAGE VOLUME](../sql-reference/sql-statements/Administration/DROP_STORAGE_VOLUME.md), [SET DEFAULT STORAGE VOLUME](../sql-reference/sql-statements/Administration/SET_DEFAULT_STORAGE_VOLUME.md), [DESC STORAGE VOLUME](../sql-reference/sql-statements/Administration/DESC_STORAGE_VOLUME.md), [SHOW STORAGE VOLUMES](../sql-reference/sql-statements/Administration/SHOW_STORAGE_VOLUMES.md)。
- [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) を使用して表のコメントを変更することをサポート。[#21035](https://github.com/StarRocks/starrocks/pull/21035)。
- 次の関数を追加: Struct functions: [struct (row)](../sql-reference/sql-functions/struct-functions/row.md), [named_struct](../sql-reference/sql-functions/struct-functions/named_struct.md)。Map functions: [str_to_map](../sql-reference/sql-functions/string-functions/str_to_map.md), [map_concat](../sql-reference/sql-functions/map-functions/map_concat.md), [map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md), [element_at](../sql-reference/sql-functions/map-functions/element_at.md), [distinct_map_keys](../sql-reference/sql-functions/map-functions/distinct_map_keys.md), [cardinality](../sql-reference/sql-functions/map-functions/cardinality.md)。Higher-order Map functions: [map_filter](../sql-reference/sql-functions/map-functions/map_filter.md), [map_apply](../sql-reference/sql-functions/map-functions/map_apply.md), [transform_keys](../sql-reference/sql-functions/map-functions/transform_keys.md), [transform_values](../sql-reference/sql-functions/map-functions/transform_values.md)。Array functions: [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) は `ORDER BY` をサポート、[array_generate](../sql-reference/sql-functions/array-functions/array_generate.md), [element_at](../sql-reference/sql-functions/array-functions/element_at.md), [cardinality](../sql-reference/sql-functions/array-functions/cardinality.md)。Higher-order Array functions: [all_match](../sql-reference/sql-functions/array-functions/all_match.md), [any_match](../sql-reference/sql-functions/array-functions/any_match.md)。Aggregate functions: [min_by](../sql-reference/sql-functions/aggregate-functions/min_by.md), [percentile_disc](../sql-reference/sql-functions/aggregate-functions/percentile_disc.md)。Table functions: [FILES](../sql-reference/sql-functions/table-functions/files.md), [generate_series](../sql-reference/sql-functions/table-functions/generate_series.md)。Date functions: [next_day](../sql-reference/sql-functions/date-time-functions/next_day.md), [previous_day](../sql-reference/sql-functions/date-time-functions/previous_day.md), [last_day](../sql-reference/sql-functions/date-time-functions/last_day.md), [makedate](../sql-reference/sql-functions/date-time-functions/makedate.md), [date_diff](../sql-reference/sql-functions/date-time-functions/date_diff.md)。Bitmap functions：[bitmap_subset_limit](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_limit.md), [bitmap_subset_in_range](../sql-reference/sql-functions/bitmap-functions/bitmap_subset_in_range.md)。Math functions: [cosine_similarity](../sql-reference/sql-functions/math-functions/cos_similarity.md), [cosine_similarity_norm](../sql-reference/sql-functions/math-functions/cos_similarity_norm.md)。

#### 権限とセキュリティ

ストレージボリュームに関連する[権限項目](../administration/privilege_item.md#storage-volume) および外部カタログに関連する[権限項目](../administration/privilege_item.md#catalog) を追加し、これらの権限を付与および取り消すために[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) および[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) の使用をサポートしました。

### 改善点

#### 共有データクラスタ

共有データ StarRocks クラスタのデータキャッシュを最適化しました。最適化されたデータキャッシュにより、ホットデータの範囲を指定できるほか、冷データへのクエリがローカルディスクキャッシュを占有することを防ぐことで、ホットデータへのクエリのパフォーマンスを確保できます。

#### マテリアライズドビュー

- 非同期型マテリアライズドビューの作成を最適化しました:
  - ランダムバケティングをサポート。ユーザーがバケット列を指定しない場合、StarRocks はデフォルトでランダムバケティングを採用します。
  - `ORDER BY` を使用してソートキーを指定することをサポート。
  - `colocate_group`、`storage_medium`、`storage_cooldown_time` などの属性を指定することをサポート。
  - セッション変数を使用することをサポート。ユーザーは、`properties("session.<variable_name>" = "<value>")` 構文を使用してこれらの変数を設定し、柔軟にビューの更新戦略を調整できます。
  - すべての非同期マテリアライズドビューにスピル機能を有効にし、デフォルトで 1 時間のクエリタイムアウトを実装します。
  - ビューに基づくマテリアライズドビューの作成をサポート。これにより、レイヤードモデリングを実装するために、ユーザーは必要に応じて柔軟にビューとマテリアライズドビューを使用できます。
- 非同期型マテリアライズドビューのクエリ書き換えを最適化しました:
  - 指定された時間間隔内に更新されないマテリアライズドビューが、マテリアライズビューの基本テーブルが更新されるかどうかに関係なく、クエリの書き換えに使用される Stale Rewrite をサポートします。ユーザーは、マテリアライズドビューの作成時に `mv_rewrite_staleness_second` プロパティを使用して時間間隔を指定できます。
  - Hive カタログテーブルで作成されるマテリアライズドビューに対して、View Delta Join クエリの書き換えをサポートします（プライマリキーと外部キーを定義する必要があります）。
  - Union 操作を含むクエリの書き換え機構を最適化し、JOIN や COUNT DISTINCT、time_slice などの関数を含むクエリの書き換えをサポートしました。
- 非同期型マテリアライズドビューの更新を最適化しました:
- Hiveカタログテーブルに作成されたマテリアライズドビューのリフレッシュメカニズムを最適化しました。 StarRocksは今、パーティションレベルのデータ変更を認識し、自動リフレッシュ中にデータ変更のあるパーティションのみをリフレッシュします。
- `REFRESH MATERIALIZED VIEW WITH SYNC MODE`構文を使用して、同期的にマテリアライズドビューのリフレッシュタスクを呼び出すことをサポートしています。
- 非同期マテリアライズドビューの利用を強化しました：
  - `ALTER MATERIALIZED VIEW {ACTIVE | INACTIVE}`を使用してマテリアライズドビューを有効または無効にすることをサポートしています。 無効になったマテリアライズドビュー（`INACTIVE`状態）は、リフレッシュできずクエリ書き換えにも使用できませんが、直接クエリを実行できます。
  - `ALTER MATERIALIZED VIEW SWAP WITH`を使用して2つのマテリアライズドビューを交換することをサポートしています。 ユーザーは新しいマテリアライズドビューを作成し、既存のマテリアライズドビューとアトミックスワップを実行して既存のマテリアライズドビューのスキーマ変更を実装できます。
- 同期マテリアライズドビューを最適化しました：
  - SQLヒント`[_SYNC_MV_]`を使用して同期マテリアライズドビューに対して直接クエリを実行し、一部のクエリが稀な状況で適切に書き換えられない問題を解決できるようにサポートしています。
  - `CASE-WHEN`、`CAST`、および数学演算などのより多くの式をサポートしており、マテリアライズドビューをさまざまなビジネスシナリオに適したものにしています。

#### データレイクアナリティクス

- Icebergデータクエリのパフォーマンス向上のためにIcebergのメタデータキャッシュとアクセスを最適化しました。
- データレイクアナリティクスのパフォーマンスをさらに向上させるためにデータキャッシュを最適化しました。

#### ストレージエンジン、データ取り込み、およびクエリ

- [スパイル](../administration/spill_to_disk.md)機能の一般提供を発表しました。この機能は、いくつかのブロッキングオペレータの中間計算結果をディスクにスパイリングすることをサポートしています。このスパイル機能を有効にすると、クエリに集約、ソート、または結合オペレータが含まれている場合、StarRocksはオペレータの中間計算結果をディスクにキャッシュしてメモリ消費量を削減し、メモリ制限によるクエリの失敗を最小限に抑えることができます。
- カーディナリティ保存結合のプルーニングをサポートしています。ユーザーが星型スキーマ（たとえばSSB）またはスノーフレークスキーマ（たとえばTPC-H）で組織された大量のテーブルを維持しており、これらのテーブルのうちわずかしかクエリしていない場合、この機能は必要のないテーブルを削除して結合のパフォーマンスを向上させます。
- 列モードで部分更新をサポートしています。主キーテーブルの部分的な更新を実行する場合、[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)ステートメントを使用して列モードを有効にできます。列モードは、列の数は少ないが行数が多い更新に適しており、更新のパフォーマンスを最大10倍向上させることができます。
- CBOの統計の収集を最適化しました。これにより、データ取り込みに対する統計の収集の影響が軽減され、統計の収集のパフォーマンスが向上します。
- マージアルゴリズムを最適化して、順列シナリオで全体のパフォーマンスを最大2倍向上させました。
- データベースのロックへの依存を減らすために、クエリロジックを最適化しました。
- 動的分割をさらにサポートして、分割ユニットが年であることをサポートします。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQLリファレンス

- 条件関数case、coalesce、if、ifnull、nullifがARRAY、MAP、STRUCT、JSONデータタイプをサポートしています。
- 次の配列関数はネストされたタイプMAP、STRUCT、ARRAYをサポートしています：
  - array_agg
  - array_contains、array_contains_all、array_contains_any
  - array_slice、array_concat
  - array_length、array_append、array_remove、array_position
  - reverse、array_distinct、array_intersect、arrays_overlap
  - array_sortby
- 次の配列関数はFast Decimalデータタイプをサポートしています：
  - array_agg
  - array_append、array_remove、array_position、array_contains
  - array_length
  - array_max、array_min、array_sum、array_avg
  - arrays_overlap、array_difference
  - array_slice、array_distinct、array_sort、reverse、array_intersect、array_concat
  - array_sortby、array_contains_all、array_contains_any

### バグ修正

次の問題を修正しました：

- ルーチンロードジョブのKafkaへの再接続要求が適切に処理されないことがあります。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- 複数のテーブルを含むSQLクエリに`WHERE`句が含まれる場合、これらのSQLクエリのセマンティクスが同じでも、各SQLクエリのテーブルの順序が異なる場合、関連するマテリアライズドビューから利益を得るためにこれらのSQLクエリの一部が失敗することがあります。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- `GROUP BY`句を含むクエリで重複したレコードが返されることがあります。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- lead()またはlag()関数を呼び出すと、BEがクラッシュすることがあります。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 外部カタログテーブルに作成されたマテリアライズドビューに基づく部分的なパーティションクエリの書き換えが失敗することがあります。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- 斜線（`/`）とセミコロン（`;`）の両方を含むSQL文が適切にパースされないことがあります。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- テーブルのマテリアライズドビューが削除された場合、テーブルを切り詰めることができません。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 動作の変更

- 共有データStarRocksクラスターのテーブル作成構文から`storage_cache_ttl`パラメータが削除されました。現在、ローカルキャッシュ内のデータはLRUアルゴリズムに基づいて退去されます。
- BE設定項目`disable_storage_page_cache`および`alter_tablet_worker_count`およびFE設定項目`lake_compaction_max_tasks`の不変パラメータが可変パラメータに変更されました。
- BE設定項目`block_cache_checksum_enable`のデフォルト値が`true`から`false`に変更されました。
- BE設定項目`enable_new_load_on_memory_limit_exceeded`のデフォルト値が`false`から`true`に変更されました。
- FE設定項目`max_running_txn_num_per_db`のデフォルト値が`100`から`1000`に変更されました。
- FE設定項目`http_max_header_size`のデフォルト値が`8192`から`32768`に変更されました。
- FE設定項目`tablet_create_timeout_second`のデフォルト値が`1`から`10`に変更されました。
- FE設定項目`max_routine_load_task_num_per_be`のデフォルト値が`5`から`16`に変更され、大量のRoutine Loadタスクが作成された場合、エラー情報が返されます。
- FE設定項目`quorom_publish_wait_time_ms`が`quorum_publish_wait_time_ms`に名前が変更され、FE設定項目`async_load_task_pool_size`が`max_broker_load_job_concurrency`に名前が変更されました。
- BE設定項目`routine_load_thread_pool_size`が廃止されました。現在、ルーチンロードスレッドプールサイズはFE設定項目`max_routine_load_task_num_per_be`によって制御されます。
- BE設定項目`txn_commit_rpc_timeout_ms`およびシステム変数`tx_visible_wait_timeout`が廃止されました。
- FE設定項目`max_broker_concurrency`および`load_parallel_instance_num`が廃止されました。
- FE設定項目`max_routine_load_job_num`が廃止されました。現在、StarRocksは`max_routine_load_task_num_per_be`パラメータに基づいて個々のBEノードでサポートされるRoutine Loadタスクの最大数を動的に推論し、タスクの失敗に対する検討事項を提供します。
- CN設定項目`thrift_port`が`be_port`に名前が変更されました。
- ルーチンロードジョブプロパティの新しい`task_consume_second`および`task_timeout_second`が追加され、ルーチンロードジョブ内の個々のロードタスクのデータを消費する最大時間とタイムアウト期間を制御するようになりました。ユーザーがルーチンロードジョブでこれら2つのプロパティを指定しない場合、FE設定項目`routine_load_task_consume_second`および`routine_load_task_timeout_second`が優先します。
- セッション変数`enable_resource_group`が廃止されました。なぜなら、v3.1.0以降、[リソースグループ](../administration/resource_group.md)機能がデフォルトで有効になっているからです。
- 新しい予約キーワードCOMPACTIONおよびTEXTが追加されました。