---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 3.1

## 3.1.4

リリース日: 2023年11月2日

### 新機能

- 共有データの StarRocks クラスタで作成された主キー付きテーブルに対して、ソートキーをサポートします。
- str2date 関数を使用して非同期マテリアライズドビューのパーティション式を指定することができるようになりました。これにより、外部カタログに存在するテーブルで、STRING 型のデータをパーティション式として使用している非同期マテリアライズドビューの増分更新とクエリの書き換えが容易になります。[#29923](https://github.com/StarRocks/starrocks/pull/29923) [#31964](https://github.com/StarRocks/starrocks/pull/31964)
- 新しいセッション変数 `enable_query_tablet_affinity` を追加しました。このセッション変数は、同じタブレットに対する複数のクエリを固定のレプリカに送信するかどうかを制御します。このセッション変数はデフォルトで `false` に設定されています。[#33049](https://github.com/StarRocks/starrocks/pull/33049)
- `is_role_in_session` という新しいセッション変数を追加しました。この変数は、現在のセッションでアクティブ化されている指定されたロールが付与されているかどうかをチェックするために使用されます。この変数は、ユーザーに付与されたネストされたロールをチェックすることもサポートしています。[#32984](https://github.com/StarRocks/starrocks/pull/32984)
- リソースグループレベルのクエリキューを設定する機能をサポートしました。この機能は、グローバル変数 `enable_group_level_query_queue`（デフォルト値: `false`）によって制御されます。グローバルレベルまたはリソースグループレベルのリソース消費が事前に定義されたしきい値に達すると、新しいクエリはキューに配置され、グローバルレベルのリソース消費とリソースグループレベルのリソース消費がそれぞれしきい値以下になると実行されます。
  - ユーザーは、各リソースグループに対して `concurrency_limit` を設定して、各 BE あたりの最大同時クエリ数を制限することができます。
  - ユーザーは、各リソースグループに対して `max_cpu_cores` を設定して、各 BE あたりの最大 CPU 消費量を制限することができます。
- リソースグループ分類子のための `plan_cpu_cost_range` と `plan_mem_cost_range` の 2 つのパラメータを追加しました。
  - `plan_cpu_cost_range`: システムによって推定される CPU 消費範囲。デフォルト値 `NULL` は制限がないことを示します。
  - `plan_mem_cost_range`: システムによって推定されるメモリ消費範囲。デフォルト値 `NULL` は制限がないことを示します。

### 改善点

- Window 関数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP が、ORDER BY 句と Window 句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- クエリの DECIMAL 型データにおいて、桁あふれが発生した場合に NULL ではなくエラーが返されるようになりました。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- クエリキュー内で許可される同時クエリ数の管理をリーダー FE で行うようになりました。各フォロワー FE は、クエリの開始と終了時にリーダー FE に通知します。同時クエリ数がグローバルレベルまたはリソースグループレベルの `concurrency_limit` に達すると、新しいクエリは拒否されるかキューに配置されます。

### バグ修正

以下の問題が修正されました:

- Spark や Flink がメモリ使用量の統計情報が不正確であるため、データ読み取りエラーを報告する場合があります。[#30702](https://github.com/StarRocks/starrocks/pull/30702)  [#30751](https://github.com/StarRocks/starrocks/pull/30751)
- メタデータキャッシュのメモリ使用量の統計情報が不正確です。[#31978](https://github.com/StarRocks/starrocks/pull/31978)
- libcurl が呼び出されると BE がクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- Hive ビュー上に作成された StarRocks マテリアライズドビューをリフレッシュすると、エラー "java.lang.ClassCastException: com.starrocks.catalog.HiveView cannot be cast to com.starrocks.catalog.HiveMetaStoreTable" が返されます。[#31004](https://github.com/StarRocks/starrocks/pull/31004)
- ORDER BY 句に集約関数が含まれている場合、エラー "java.lang.IllegalStateException: null" が返されます。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- 共有データの StarRocks クラスタでは、テーブルキーの情報が `information_schema.COLUMNS` に記録されません。そのため、Flink Connector を使用してデータをロードする場合に DELETE 操作を実行することができません。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- Flink Connector を使用してデータをロードする場合、非常に同時的なロードジョブが存在し、HTTP スレッドの数とスキャンスレッドの数の両方が上限に達した場合、ロードジョブが予期せず中断されることがあります。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- 数バイトしかないフィールドが追加された場合、データの変更が完了する前に SELECT COUNT(*) を実行すると "error: invalid field name" というエラーが返されます。[#33243](https://github.com/StarRocks/starrocks/pull/33243)
- クエリキャッシュが有効になっている場合、クエリの結果が正しくありません。[#32781](https://github.com/StarRocks/starrocks/pull/32781)
- ハッシュ結合中にクエリが失敗し、BE がクラッシュすることがあります。[#32219](https://github.com/StarRocks/starrocks/pull/32219)
- BINARY や VARBINARY データ型の `DATA_TYPE` と `COLUMN_TYPE` は、`information_schema.columns` ビューで `unknown` と表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### 動作の変更

- v3.1.4 以降、新しい StarRocks クラスタで作成された主キー付きテーブルでは、デフォルトで永続インデックスが有効になります（これは、以前のバージョンから v3.1.4 にアップグレードされた既存の StarRocks クラスタには適用されません）。[#33374](https://github.com/StarRocks/starrocks/pull/33374)
- 新しい FE パラメータ `enable_sync_publish` を追加しました。このパラメータはデフォルトで `true` に設定されています。このパラメータを `true` に設定すると、プライマリキーテーブルへのデータロードの Publish フェーズは、Apply タスクが完了した後にのみ実行結果を返します。そのため、ロードジョブが成功メッセージを返した後、データを即座にクエリできます。ただし、このパラメータを `true` に設定すると、プライマリキーテーブルへのデータロードにはより長い時間がかかる場合があります。（このパラメータが追加される前は、Apply タスクは Publish フェーズと非同期で実行されていました。）[#27055](https://github.com/StarRocks/starrocks/pull/27055)

## 3.1.3

リリース日: 2023年9月25日

### 動作の変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数を使用する場合、セパレータを宣言するために SEPARATOR キーワードを使用する必要があります。

### 新機能

- 共有データの StarRocks クラスタで作成された主キー付きテーブルは、共有しない StarRocks クラスタと同様に、ローカルディスクへのインデックス永続化をサポートします。
- 集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) は、DISTINCT キーワードと ORDER BY 句をサポートします。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[Kafka Connector](../loading/Kafka-connector-starrocks.md)、[Flink Connector](../loading/Flink-connector-starrocks.md)、[Spark Connector](../loading/Spark-connector-starrocks.md) は、主キー付きテーブルに対してカラムモードで部分更新をサポートします。[#28288](https://github.com/StarRocks/starrocks/pull/28288)
- パーティション内のデータを時間経過によって自動的に冷却することができます。（この機能は [リストパーティショニング](../table_design/list_partitioning.md) ではサポートされていません。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改善点

無効なコメントを含む SQL コマンドを実行した場合、MySQL と一貫した結果が返されるように改善しました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)

### バグ修正

以下の問題が修正されました:

- [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md) ステートメントの WHERE 句で [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) または [HLL](../sql-reference/sql-statements/data-types/HLL.md) データ型が指定された場合、ステートメントが正しく実行されません。[#28592](https://github.com/StarRocks/starrocks/pull/28592)
- フォロワー FE を再起動すると、CpuCores の統計情報が最新ではなくなり、クエリのパフォーマンスが低下します。[#28472](https://github.com/StarRocks/starrocks/pull/28472) [#30434](https://github.com/StarRocks/starrocks/pull/30434)
- [to_bitmap()](../sql-reference/sql-functions/bitmap-functions/to_bitmap.md) 関数の実行コストが誤って計算されるため、マテリアライズドビューが書き換えられた後に関数のための適切な実行計画が選択されません。[#29961](https://github.com/StarRocks/starrocks/pull/29961)
- 共有データアーキテクチャの特定の使用例では、フォロワー FE を再起動すると、フォロワー FE に送信されたクエリがエラー "Backend node not found. Check if any backend node is down" を返します。[#28615](https://github.com/StarRocks/starrocks/pull/28615)
- テーブルが [ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) ステートメントを使用して変更されている間にデータが連続的にロードされると、一部のパーティションでエラー "Tablet is in error state" がスローされる場合があります。[#29364](https://github.com/StarRocks/starrocks/pull/29364)
- `ADMIN SET FRONTEND CONFIG` コマンドを使用して FE の動的パラメータ `max_broker_load_job_concurrency` を変更しても効果がありません。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- [date_diff()](../sql-reference/sql-functions/date-time-functions/date_diff.md) 関数の時間単位が定数であり、日付が定数でない場合、BE がクラッシュします。[#29937](https://github.com/StarRocks/starrocks/issues/29937)
- 共有データアーキテクチャでは、非同期ロードが有効になっている場合に自動パーティショニングが機能しないことがあります。[#29986](https://github.com/StarRocks/starrocks/issues/29986)
- ユーザーが [CREATE TABLE LIKE](../sql-reference/sql-statements/data-definition/CREATE_TABLE_LIKE.md) ステートメントを使用して主キー付きテーブルを作成すると、エラー `Unexpected exception: Unknown properties: {persistent_index_type=LOCAL}` がスローされます。[#30255](https://github.com/StarRocks/starrocks/pull/30255)
- BE を再起動すると、主キー付きテーブルの復元によりメタデータの不整合が発生します。[#30135](https://github.com/StarRocks/starrocks/pull/30135)
- ユーザーがトランケート操作とクエリを同時に実行する主キー付きテーブルにデータをロードすると、一部のケースでエラー "java.lang.NullPointerException" がスローされる場合があります。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- マテリアライズドビュー作成ステートメントで述語式が指定されている場合、それらのマテリアライズドビューのリフレッシュ結果が正しくありません。[#29904](https://github.com/StarRocks/starrocks/pull/29904)
- StarRocks クラスタを v3.1.2 にアップグレードした後、アップグレード前に作成されたテーブルのストレージボリュームのプロパティが `null` にリセットされます。[#30647](https://github.com/StarRocks/starrocks/pull/30647)
- タブレットメタデータのチェックポイントと復元を同時に実行すると、一部のタブレットレプリカが失われ、取得できなくなります。[#30603](https://github.com/StarRocks/starrocks/pull/30603)
- ユーザーが CloudCanal を使用して `NOT NULL` に設定されたテーブル列にデータをロードしようとすると、エラー "Unsupported dataFormat value is : \N" がスローされます。[#30799](https://github.com/StarRocks/starrocks/pull/30799)

## 3.1.2

リリース日: 2023年8月25日

### バグ修正

以下の問題が修正されました:

- ユーザーがデフォルトで接続するデータベースを指定し、ユーザーがデータベースに対して権限を持たず、テーブルに対してのみ権限を持つ場合、データベースに対して権限がないというエラーがスローされます。[#29767](https://github.com/StarRocks/starrocks/pull/29767)
- クラウドネイティブテーブルの RESTful API アクション `show_data` が返す値が正しくありません。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- [array_agg()](../sql-reference/sql-functions/array-functions/array_agg.md) 関数の実行中にクエリがキャンセルされると、BE がクラッシュします。[#29400](https://github.com/StarRocks/starrocks/issues/29400)
- [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) ステートメントが [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) または [HLL](../sql-reference/sql-statements/data-types/HLL.md) データ型のカラムに対して返す "Default" フィールドの値が正しくありません。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- クエリに複数のテーブルを含む [array_map()](../sql-reference/sql-functions/array-functions/array_map.md) 関数が含まれる場合、プッシュダウン戦略の問題によりクエリが失敗します。[#29504](https://github.com/StarRocks/starrocks/pull/29504)
- ORC 形式のファイルに対するクエリが失敗するため、Apache ORC のバグ修正 ORC-1304 ([apache/orc#1299](https://github.com/apache/orc/pull/1299)) がマージされていません。[#29804](https://github.com/StarRocks/starrocks/pull/29804)

### 動作の変更

新しく展開された StarRocks v3.1 クラスタでは、[SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md) を実行してそのカタログに切り替える場合、宛先の外部カタログに対する USAGE 権限が必要です。必要な権限を付与するために [GRANT](../sql-reference/sql-statements/account-management/GRANT.md) を使用できます。

以前のバージョンから v3.1 クラスタにアップグレードされた場合、継承された権限で SET CATALOG を実行できます。

## 3.1.1

リリース日: 2023年8月18日

### 新機能

- 共有データクラスタで Azure Blob Storage をサポートしました。
- 共有データクラスタでリストパーティショニングをサポートしました。
- 集約関数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md) をサポートしました。
- [Window 関数](../sql-reference/sql-functions/Window_function.md) COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMP をサポートしました。

### 改善点

#### 共有データクラスタ

共有データの StarRocks クラスタのデータキャッシュを最適化しました。最適化されたデータキャッシュでは、ホットデータの範囲を指定することができます。また、コールドデータに対するクエリがローカルディスクキャッシュを占有することを防ぐことができるため、ホットデータに対するクエリのパフォーマンスを確保することができます。

#### マテリアライズドビュー

- 非同期マテリアライズドビューの作成を最適化しました:
  - バケット列を指定しない場合、StarRocks はデフォルトでランダムバケットを採用するようになりました。
  - `ORDER BY` を使用してソートキーを指定することができるようになりました。
  - `colocate_group`、`storage_medium`、`storage_cooldown_time` などの属性を指定することができるようになりました。
  - セッション変数を使用することができるようになりました。ユーザーは `properties("session.<variable_name>" = "<value>")` の構文を使用してこれらの変数を設定し、柔軟にビューのリフレッシュ戦略を調整することができます。
  - すべての非同期マテリアライズドビューにスピル機能を有効にし、デフォルトでクエリのタイムアウト時間を 1 時間に設定しました。
  - ビューを基にしたマテリアライズドビューの作成をサポートしました。これにより、ビューとマテリアライズドビューを柔軟に使用することができるため、データモデリングのシナリオでマテリアライズドビューをより使いやすくすることができます。
- 非同期マテリアライズドビューのクエリ書き換えを最適化しました:
  - ステールリライトをサポートしました。ステールリライトでは、指定された時間間隔内にリフレッシュされないマテリアライズドビューを、マテリアライズドビューの基になるテーブルが更新されているかどうかに関係なく、クエリ書き換えに使用することができます。ユーザーはマテリアライズドビュー作成時に `mv_rewrite_staleness_second` プロパティを使用して時間間隔を指定することができます。
  - 主キーや外部キーが定義されているマテリアライズドビューに対する View Delta Join クエリの書き換えをサポートしました。
  - UNION 操作を含むクエリの書き換えメカニズムを最適化し、COUNT DISTINCT や time_slice などの関数を含むクエリの書き換えをサポートしました。
- 非同期マテリアライズドビューのリフレッシュを最適化しました:
  - Hive カタログテーブル上に作成されたマテリアライズドビューのリフレッシュを最適化しました。StarRocks はパーティションレベルのデータ変更を感知し、自動リフレッシュ時にデータ変更のあるパーティションのみをリフレッシュします。
  - `REFRESH MATERIALIZED VIEW WITH SYNC MODE` 構文を使用してマテリアライズドビューのリフレッシュタスクを同期的に呼び出すことができるようになりました。
- 同期マテリアライズドビューを強化しました:
  - SQL ヒント `[_SYNC_MV_]` を使用して同期マテリアライズドビューに直接クエリを実行できるようになりました。これにより、一部のクエリが稀な状況で正しく書き換えられない問題を回避することができます。
  - `CASE-WHEN`、`CAST`、数学演算など、より多くの式をサポートするようになりました。これにより、マテリアライズドビューはさまざまなビジネスシナリオに適しています。

#### データレイクアナリティクス

- Iceberg のメタデータキャッシュとアクセスを最適化し、Iceberg データのクエリパフォーマンスを向上させました。
- データキャッシュを最適化して、データレイクアナリティクスのパフォーマンスをさらに向上させました。

#### ストレージエンジン、データ取り込み、クエリ

- いくつかのブロッキングオペレータの中間計算結果をディスクにスピルする [spill](../administration/spill_to_disk.md) 機能の一般提供を発表しました。スピル機能を有効にすると、クエリに集約、ソート、または結合演算子が含まれている場合、StarRocks は演算子の中間計算結果をディスクにキャッシュしてメモリ使用量を削減し、メモリ制限によるクエリの失敗を最小限に抑えることができます。
- カーディナリティを保存する結合に対するプルーニングをサポートしました。ユーザーがスターシェーマ（例: SSB）やスノーフレークスキーマ（例: TCP-H）で組織された大量のテーブルを保持しており、これらのテーブルのうちわずかな数のテーブルのみをクエリする場合、この機能を使用して不要なテーブルをプルーニングして結合のパフォーマンスを向上させることができます。
- カラムモードで部分更新をサポートしています。[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) ステートメントを使用して、主キーテーブルで部分更新を実行する際に、ユーザーはカラムモードを有効にすることができます。カラムモードは、少数のカラムを更新するが大量の行を更新する場合に適しており、更新パフォーマンスを最大で10倍向上させることができます。
- CBOの統計情報の収集を最適化しました。これにより、データの取り込みに対する統計情報の収集の影響が軽減され、統計情報の収集パフォーマンスが向上します。
- パーミュテーションシナリオでの全体的なパフォーマンスを最大で2倍向上させるために、マージアルゴリズムを最適化しました。
- データベースのロックに依存しないようにクエリロジックを最適化しました。
- ダイナミックパーティショニングは、パーティショニング単位を年にすることをさらにサポートします。[#28386](https://github.com/StarRocks/starrocks/pull/28386)

#### SQLリファレンス

- 条件付き関数 case、coalesce、if、ifnull、nullif は、ARRAY、MAP、STRUCT、JSON データ型をサポートしています。
- 次の配列関数は、ネストされた型 MAP、STRUCT、ARRAY をサポートしています：
  - array_agg
  - array_contains, array_contains_all, array_contains_any
  - array_slice, array_concat
  - array_length, array_append, array_remove, array_position
  - reverse, array_distinct, array_intersect, arrays_overlap
  - array_sortby
- 次の配列関数は、Fast Decimal データ型をサポートしています：
  - array_agg
  - array_append, array_remove, array_position, array_contains
  - array_length
  - array_max, array_min, array_sum, array_avg
  - arrays_overlap, array_difference
  - array_slice, array_distinct, array_sort, reverse, array_intersect, array_concat
  - array_sortby, array_contains_all, array_contains_any

### バグ修正

以下の問題が修正されました：

- ルーチンロードジョブの再接続要求を正しく処理できない問題が修正されました。[#23477](https://github.com/StarRocks/starrocks/issues/23477)
- 複数のテーブルを含むSQLクエリに`WHERE`句が含まれる場合、これらのSQLクエリが同じ意味を持ちながら各SQLクエリのテーブルの順序が異なる場合、一部のSQLクエリは関連するマテリアライズドビューの恩恵を受けるために再書き込みされない場合があります。[#22875](https://github.com/StarRocks/starrocks/issues/22875)
- `GROUP BY`句を含むクエリで重複したレコードが返される問題が修正されました。[#19640](https://github.com/StarRocks/starrocks/issues/19640)
- lead()またはlag()関数を呼び出すと、BEがクラッシュする可能性があります。[#22945](https://github.com/StarRocks/starrocks/issues/22945)
- 外部カタログテーブル上に作成されたマテリアライズドビューに基づいて部分パーティションクエリを再書き込みすることができない問題が修正されました。[#19011](https://github.com/StarRocks/starrocks/issues/19011)
- バックスラッシュ(`\`)とセミコロン(`;`)の両方を含むSQLステートメントを正しく解析することができない問題が修正されました。[#16552](https://github.com/StarRocks/starrocks/issues/16552)
- テーブルがトランケートされない場合、テーブルから作成されたマテリアライズドビューが削除された場合にトランケートできない問題が修正されました。[#19802](https://github.com/StarRocks/starrocks/issues/19802)

### 動作の変更

- 共有データのStarRocksクラスタのテーブル作成構文から`storage_cache_ttl`パラメータが削除されました。現在、ローカルキャッシュのデータはLRUアルゴリズムに基づいて削除されます。
- BEの設定項目`disable_storage_page_cache`と`alter_tablet_worker_count`、およびFEの設定項目`lake_compaction_max_tasks`は、変更可能なパラメータから変更不可能なパラメータに変更されました。
- BEの設定項目`block_cache_checksum_enable`のデフォルト値が`true`から`false`に変更されました。
- BEの設定項目`enable_new_load_on_memory_limit_exceeded`のデフォルト値が`false`から`true`に変更されました。
- FEの設定項目`max_running_txn_num_per_db`のデフォルト値が`100`から`1000`に変更されました。
- FEの設定項目`http_max_header_size`のデフォルト値が`8192`から`32768`に変更されました。
- FEの設定項目`tablet_create_timeout_second`のデフォルト値が`1`から`10`に変更されました。
- FEの設定項目`max_routine_load_task_num_per_be`のデフォルト値が`5`から`16`に変更され、大量のルーチンロードタスクが作成された場合にエラー情報が返されるようになりました。
- FEの設定項目`quorom_publish_wait_time_ms`が`quorum_publish_wait_time_ms`に名前が変更され、FEの設定項目`async_load_task_pool_size`が`max_broker_load_job_concurrency`に名前が変更されました。
- BEの設定項目`routine_load_thread_pool_size`は非推奨となりました。現在、ルーチンロードスレッドプールサイズは、FEの設定項目`max_routine_load_task_num_per_be`によって制御されるようになりました。
- BEの設定項目`txn_commit_rpc_timeout_ms`とシステム変数`tx_visible_wait_timeout`は非推奨となりました。トランザクションのタイムアウト期間を指定するために`time_out`パラメータが使用されるようになりました。
- FEの設定項目`max_broker_concurrency`と`load_parallel_instance_num`は非推奨となりました。
- FEの設定項目`max_routine_load_job_num`は非推奨となりました。現在、StarRocksは`max_routine_load_task_num_per_be`パラメータに基づいて各個別のBEノードがサポートするルーチンロードタスクの最大数を動的に推定し、タスクの失敗についての提案を行います。
- CNの設定項目`thrift_port`が`be_port`に名前が変更されました。
- ルーチンロードジョブの最大データ消費時間とタイムアウト時間を制御するための2つの新しいプロパティ、`task_consume_second`と`task_timeout_second`が追加されました。これにより、ジョブの調整がより柔軟に行えます。ユーザーがこれらの2つのプロパティをルーチンロードジョブで指定しない場合、FEの設定項目`routine_load_task_consume_second`と`routine_load_task_timeout_second`が優先されます。
- セッション変数`enable_resource_group`は非推奨となりました。なぜなら、[リソースグループ](../administration/resource_group.md)機能はv3.1.0以降デフォルトで有効になっているためです。
- COMPACTIONとTEXTという2つの新しい予約キーワードが追加されました。
