---
displayed_sidebar: English
---

# StarRocks バージョン 3.0

## 3.0.8

リリース日: 2023年11月17日

## 改善点

- システムデータベース `INFORMATION_SCHEMA` の `COLUMNS` ビューは、ARRAY、MAP、およびSTRUCT列を表示できるようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

## バグ修正

以下の問題を修正しました:

- `show proc '/current_queries';` の実行中にクエリが開始されると、BEがクラッシュする可能性があります。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- ソートキーを指定して高頻度でプライマリキーテーブルにデータをロードすると、コンパクションの失敗が発生することがあります。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- Broker Loadジョブでフィルタリング条件を指定した場合、特定の状況でデータロード中にBEがクラッシュすることがあります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTSを実行すると未知のエラーが報告されます。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- `cast()` 関数で指定されたターゲットデータ型が元のデータ型と同じ場合、特定のデータ型でBEがクラッシュすることがあります。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- BINARYまたはVARBINARYデータ型の `DATA_TYPE` と `COLUMN_TYPE` が `information_schema.columns` ビューで `unknown` として表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 永続インデックスが有効なプライマリキーテーブルに長時間かつ頻繁にデータをロードすると、BEがクラッシュすることがあります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効な場合、クエリ結果が不正確になることがあります。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- クラスタを再起動した後、復元されたテーブルのデータがバックアップ前のデータと一致しないことがあります。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- RESTOREを実行中にコンパクションが発生すると、BEがクラッシュすることがあります。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

リリース日: 2023年10月18日

### 改善点

- ウィンドウ関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPがORDER BY句とウィンドウ句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- プライマリキーテーブルにデータを書き込むロードジョブのパブリッシュフェーズが非同期モードから同期モードに変更され、ロードジョブが終了するとすぐにデータをクエリできるようになりました。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- DECIMAL型データに対するクエリ中に10進数のオーバーフローが発生した場合、NULLではなくエラーが返されるようになりました。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドを実行すると、MySQLと一致する結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- RANGEパーティショニングを使用し、パーティショニング列が1つだけ、または式パーティショニングを使用するStarRocksテーブルでは、パーティション列の式を含むSQL述部をパーティションプルーニングに使用できるようになりました。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### バグ修正

以下の問題を修正しました:

- データベースとテーブルの同時作成と削除を行うと、場合によってはテーブルが見つからず、そのテーブルへのデータロードが失敗することがあります。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- UDFを使用すると、特定のケースでメモリリークが発生することがあります。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- ORDER BY句に集約関数が含まれている場合、エラー「java.lang.IllegalStateException: null」が返されます。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- ユーザーが複数レベルのHiveカタログを使用してTencent COSに格納されたデータに対してクエリを実行すると、クエリ結果が正しくないことがあります。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- ARRAY<STRUCT>型データのSTRUCT内の一部のサブフィールドが欠けている場合、クエリ中に欠けているサブフィールドにデフォルト値が埋められるとデータの長さが正しくなく、BEがクラッシュすることがあります。
- セキュリティ脆弱性を回避するためにBerkeley DB Java Editionのバージョンがアップグレードされました。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- ユーザーがトランケート操作とクエリが同時に実行されるプライマリキーテーブルにデータをロードすると、特定のケースで「java.lang.NullPointerException」エラーが発生することがあります。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- スキーマ変更の実行時間が長すぎると、指定されたバージョンのタブレットがガベージコレクションされ、失敗することがあります。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- ユーザーがCloudCanalを使用して`NOT NULL`でデフォルト値が指定されていないテーブル列にデータをロードすると、「Unsupported dataFormat value is : \N」というエラーが発生することがあります。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- StarRocks共有データクラスタでは、`information_schema.COLUMNS`にテーブルキーの情報が記録されていないため、Flink Connectorを使用してデータをロードする際にDELETE操作が実行できないことがあります。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- アップグレード中に特定の列の型がアップグレードされると（例えば、Decimal型からDecimal V3型へ）、特定の特性を持つテーブルでコンパクションを行うとBEがクラッシュすることがあります。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードする際、高度に並行するロードジョブがあり、HTTPスレッド数とスキャンスレッド数が上限に達した場合、ロードジョブが予期せず中断されることがあります。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlが呼び出された際にBEがクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- BITMAP型のカラムをプライマリキーテーブルに追加する際にエラーが発生することがあります。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

リリース日: 2023年9月12日

### 挙動変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数を使用する際には、SEPARATORキーワードを使用して区切り文字を宣言する必要があります。

### 新機能

- 集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) がDISTINCTキーワードとORDER BY句をサポートするようになりました。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- パーティション内のデータは時間の経過と共に自動的にクールダウンされます。（この機能は[list partitioning](../table_design/list_partitioning.md)ではサポートされていません。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改善点

- すべての複合述語とWHERE句内のすべての式に対して暗黙の型変換をサポートします。[session variable](../reference/System_variable.md) `enable_strict_type` を使用して暗黙の型変換を有効または無効にすることができます。このセッション変数のデフォルト値は `false` です。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 文字列から整数への変換におけるFEとBEのロジックを統一しました。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### バグ修正

- `enable_orc_late_materialization` が `true` に設定されている場合、Hiveカタログを使用してORCファイル内のSTRUCT型データをクエリすると、予期しない結果が返されることがあります。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- Hiveカタログを介したデータクエリ中に、パーティション分割列とOR演算子がWHERE句で指定されている場合、クエリ結果が正しくないことがあります。[#28876](https://github.com/StarRocks/starrocks/pull/28876)

- クラウドネイティブテーブルのRESTful APIアクション `show_data` によって返される値が正しくありません。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- [共有データクラスター](../deployment/shared_data/azure.md)がAzure Blob Storageにデータを格納し、テーブルが作成された場合、クラスターがバージョン3.0にロールバックされた後、FEは起動に失敗します。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- ユーザーがIcebergカタログ内のテーブルをクエリする際、そのテーブルに対する権限が付与されていても、権限がないとされます。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) ステートメントによって返される `Default` フィールド値が、[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) または [HLL](../sql-reference/sql-statements/data-types/HLL.md) データタイプの列に対して誤っています。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- `ADMIN SET FRONTEND CONFIG` コマンドを使用してFE動的パラメータ `max_broker_load_job_concurrency` を変更しても、効果がありません。
- マテリアライズドビューがリフレッシュされている最中にそのリフレッシュ戦略が変更されると、FEが起動に失敗することがあります。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- `select count(distinct(int+double)) from table_name` が実行されると `unknown error` エラーが返されます。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- プライマリキーテーブルが復元された後、BEが再起動されるとメタデータエラーが発生し、メタデータの不整合が生じます。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

リリース日: 2023年8月16日

### 新機能

- 集計関数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、および [CORR](../sql-reference/sql-functions/aggregate-functions/corr.md) をサポートします。
- 以下の[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)をサポートします: COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMP。

### 改善点

- エラーメッセージ `xxx too many versions xxx` により詳細なプロンプトを追加しました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 動的パーティショニングは、パーティション単位として年をサポートするようにさらに拡張されました。[#28386](https://github.com/StarRocks/starrocks/pull/28386)
- テーブル作成時に式パーティショニングを使用し、[INSERT OVERWRITEを使用して特定のパーティション内のデータを上書きする場合](../table_design/expression_partitioning.md#load-data-into-partitions)、パーティションフィールドは大文字小文字を区別しません。[#28309](https://github.com/StarRocks/starrocks/pull/28309)

### バグ修正

以下の問題を修正しました:

- FEにおけるテーブルレベルのスキャン統計が不正確で、テーブルクエリとロードのメトリクスが不正確になります。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- パーティションテーブルのソートキーを変更すると、クエリ結果が不安定になります。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- データ復元後、タブレットのバージョン番号がBEとFEで不一致になります。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- ユーザーがコロケーションテーブルを作成する際にバケット数を指定しないと、0と推定され、新しいパーティションの追加に失敗します。[#27086](https://github.com/StarRocks/starrocks/pull/27086)
- INSERT INTO SELECTのSELECT結果セットが空の場合、SHOW LOADによって返されるロードジョブのステータスは `CANCELED` になります。[#26913](https://github.com/StarRocks/starrocks/pull/26913)
- sub_bitmap関数の入力値がBITMAPタイプでない場合、BEがクラッシュする可能性があります。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- AUTO_INCREMENT列が更新されている際にBEがクラッシュすることがあります。[#27199](https://github.com/StarRocks/starrocks/pull/27199)
- マテリアライズドビューの外部結合とアンチ結合の書き換えエラー。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 平均行サイズの推定が不正確で、プライマリキーの部分更新が過度に大きなメモリを占有することがあります。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 非アクティブなマテリアライズドビューをアクティブ化するとFEがクラッシュすることがあります。[#27959](https://github.com/StarRocks/starrocks/pull/27959)
- Hudiカタログの外部テーブルに基づいて作成されたマテリアライズドビューにクエリを書き換えることはできません。[#28023](https://github.com/StarRocks/starrocks/pull/28023)
- Hiveテーブルが削除され、メタデータキャッシュが手動で更新された後でも、そのテーブルのデータがクエリ可能である問題。[#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 非同期マテリアライズドビューを同期呼び出しで手動でリフレッシュすると、`information_schema.task_runs` テーブルに複数のINSERT OVERWRITEレコードが生成される問題。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- LabelCleanerスレッドがブロックされることによるFEのメモリリーク。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 3.0.4

リリース日: 2023年7月18日

### 新機能

クエリがマテリアライズドビューとは異なるタイプの結合を含む場合でも、クエリの書き換えが可能です。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改善点

- 非同期マテリアライズドビューの手動リフレッシュを最適化しました。`REFRESH MATERIALIZED VIEW WITH SYNC MODE` 構文を使用して、マテリアライズドビューのリフレッシュタスクを同期的に呼び出すことをサポートします。[#25910](https://github.com/StarRocks/starrocks/pull/25910)
- クエリされたフィールドがマテリアライズドビューの出力列に含まれていないが、マテリアライズドビューの述語に含まれている場合、マテリアライズドビューの恩恵を受けるようにクエリを書き換えることができます。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- [SQLダイアレクト (`sql_dialect`) が `trino` に設定されている場合](../reference/System_variable.md)、テーブルエイリアスは大文字小文字を区別しません。[#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- `Information_schema.tables_config` テーブルに新しいフィールド `table_id` を追加しました。`Information_schema` データベース内の `tables_config` テーブルを `be_tablets` テーブルの `table_id` 列と結合して、タブレットが属するデータベースとテーブルの名前を照会できます。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### バグ修正

以下の問題を修正しました:

- sum集計関数を含むクエリが単一テーブルマテリアライズドビューからクエリ結果を直接取得するように書き換えられた場合、型推論の問題によりsum()フィールドの値が不正確になる可能性があります。[#25512](https://github.com/StarRocks/starrocks/pull/25512)
- SHOW PROCを使用してStarRocks共有データクラスター内のタブレットに関する情報を表示する際にエラーが発生します。
- STRUCT内のCHARデータの長さが最大長を超えると、INSERT操作がハングする問題。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- FULL JOINを使用したINSERT INTO SELECTでクエリされた一部のデータ行が返されない問題。[#26603](https://github.com/StarRocks/starrocks/pull/26603)
- ALTER TABLEステートメントを使用してテーブルのプロパティ `default.storage_medium` を変更する際に `ERROR xxx: Unknown table property xxx` エラーが発生する問題。[#25870](https://github.com/StarRocks/starrocks/issues/25870)
- Broker Loadを使用して空のファイルをロードする際にエラーが発生する問題。[#26212](https://github.com/StarRocks/starrocks/pull/26212)
- BEの廃止が時々ハングする問題。[#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

リリース日: 2023年6月28日

### 改善点


- StarRocksの外部テーブルのメタデータ同期は、データロード時に行われるように変更されました。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- ユーザーは、パーティションが自動的に作成されるテーブルに対してINSERT OVERWRITEを実行する際に、パーティションを指定することができます。詳細は[自動パーティショニング](https://docs.starrocks.io/en-us/3.0/table_design/dynamic_partitioning)を参照してください。[#25005](https://github.com/StarRocks/starrocks/pull/25005)
- パーティション化されていないテーブルにパーティションを追加する際のエラーメッセージが最適化されました。[#25266](https://github.com/StarRocks/starrocks/pull/25266)

### バグ修正

以下の問題を修正しました：

- Parquetファイルに複雑なデータ型が含まれている場合、min/maxフィルターが誤ったParquetフィールドを取得する問題。[#23976](https://github.com/StarRocks/starrocks/pull/23976)
- 関連するデータベースやテーブルが削除された後も、ロードタスクがキューに残る問題。[#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FEの再起動がBEのクラッシュを引き起こす可能性が低い問題。[#25037](https://github.com/StarRocks/starrocks/pull/25037)
- `enable_profile`変数が`true`に設定されている場合、ロードおよびクエリジョブが時々フリーズする問題。[#25060](https://github.com/StarRocks/starrocks/pull/25060)
- 3つ未満の稼働中BEがあるクラスターでINSERT OVERWRITEを実行する際に不正確なエラーメッセージが表示される問題。[#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

リリース日：2023年6月13日

### 改善点

- UNIONクエリの述語は、非同期マテリアライズドビューによってクエリが書き換えられた後、プッシュダウンされるようになりました。[#23312](https://github.com/StarRocks/starrocks/pull/23312)
- テーブルの自動タブレット配布ポリシーが最適化されました。詳細は[データ配布](../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- NetworkTimeのシステムクロックへの依存が削除され、サーバー間でのシステムクロックの不一致による誤ったNetworkTimeが修正されました。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### バグ修正

以下の問題を修正しました：

- スキーマ変更がデータロードと同時に行われると、スキーマ変更がハングすることがある問題。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- セッション変数`pipeline_profile_level`が`0`に設定されているとクエリでエラーが発生する問題。[#23873](https://github.com/StarRocks/starrocks/pull/23873)
- `cloud_native_storage_type`が`S3`に設定されている場合にCREATE TABLEでエラーが発生する問題。
- パスワードを使用せずにLDAP認証が成功する問題。[#24862](https://github.com/StarRocks/starrocks/pull/24862)
- ロードジョブに関連するテーブルが存在しない場合にCANCEL LOADが失敗する問題。[#24922](https://github.com/StarRocks/starrocks/pull/24922)

### アップグレードノート

システムに`starrocks`という名前のデータベースがある場合は、アップグレード前にALTER DATABASE RENAMEを使用して名前を変更してください。これは`starrocks`が特権情報を格納するデフォルトのシステムデータベースの名前だからです。

## 3.0.1

リリース日：2023年6月1日

### 新機能

- [プレビュー] 大規模なオペレータの中間計算結果をディスクにスピルして、大規模なオペレータのメモリ消費を削減します。詳細は[ディスクへのスピル](../administration/spill_to_disk.md)を参照してください。
- [ルーチンロード](../loading/RoutineLoad.md#load-avro-format-data)はAvroデータのロードをサポートします。
- [Microsoft Azure Storage](../integrations/authenticate_to_azure_storage.md)（Azure Blob StorageおよびAzure Data Lake Storageを含む）をサポートします。

### 改善点

- 共有データクラスタは、StarRocks外部テーブルを使用して別のStarRocksクラスタとデータを同期することをサポートします。
- 最近のロードエラーを記録する`load_tracking_logs`を[情報スキーマ](../reference/information_schema/load_tracking_logs.md)に追加しました。
- CREATE TABLEステートメントでの特殊文字を無視するようになりました。[#23885](https://github.com/StarRocks/starrocks/pull/23885)

### バグ修正

以下の問題を修正しました：

- SHOW CREATE TABLEで返される情報がプライマリキーテーブルに対して不正確な問題。[#24237](https://github.com/StarRocks/starrocks/issues/24237)
- ルーチンロードジョブ中にBEがクラッシュする問題。[#20677](https://github.com/StarRocks/starrocks/issues/20677)
- パーティションテーブルの作成時にサポートされていないプロパティを指定すると発生するNullポインタ例外(NPE)の問題。[#21374](https://github.com/StarRocks/starrocks/issues/21374)
- SHOW TABLE STATUSで返される情報が不完全な問題。[#24279](https://github.com/StarRocks/starrocks/issues/24279)

### アップグレードノート

システムに`starrocks`という名前のデータベースがある場合は、アップグレード前にALTER DATABASE RENAMEを使用して名前を変更してください。これは`starrocks`が特権情報を格納するデフォルトのシステムデータベースの名前だからです。

## 3.0.0

リリース日：2023年4月28日

### 新機能

#### システムアーキテクチャ

- **ストレージとコンピュートの分離。** StarRocksは、S3互換のオブジェクトストレージへのデータ永続化をサポートし、リソースの分離を強化し、ストレージコストを削減し、コンピュートリソースをよりスケーラブルにしました。ローカルディスクは、クエリパフォーマンスを向上させるためのホットデータキャッシュとして使用されます。新しい共有データアーキテクチャのクエリパフォーマンスは、ローカルディスクキャッシュがヒットした場合のクラシックアーキテクチャ（シェアードナッシング）に匹敵します。詳細は[共有データStarRocksのデプロイと使用](../deployment/shared_data/s3.md)を参照してください。

#### ストレージエンジンとデータ取り込み

- [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)属性がサポートされ、グローバルに一意のIDを提供し、データ管理を簡素化しました。
- [自動パーティショニングとパーティション式](../table_design/dynamic_partitioning.md)がサポートされ、パーティションの作成がより使いやすく、柔軟になりました。
- プライマリキーテーブルは、CTEの使用や複数のテーブルへの参照を含む、より完全な[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)および[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#primary-key-tables)構文をサポートします。
- Broker LoadおよびINSERT INTOジョブのロードプロファイルが追加されました。ロードジョブの詳細をロードプロファイルを照会することで確認できます。使用方法は[クエリプロファイルの分析](../administration/query_profile.md)と同じです。

#### データレイクアナリティクス

- [プレビュー] Presto/Trino互換のSQL方言をサポートします。Presto/TrinoのSQLは、StarRocksのSQLパターンに自動的に書き換えられます。詳細は[システム変数](../reference/System_variable.md)の`sql_dialect`を参照してください。
- [プレビュー] [JDBCカタログ](../data_source/catalog/jdbc_catalog.md)をサポートします。
- [SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでカタログ間を手動で切り替えることをサポートします。

#### 権限とセキュリティ

- 完全なRBAC機能を備えた新しい権限システムを提供し、ロールの継承とデフォルトロールをサポートします。詳細は[権限の概要](../administration/privilege_overview.md)を参照してください。
- より多くの権限管理オブジェクトと、より細かい権限を提供します。詳細は[StarRocksがサポートする権限](../administration/privilege_item.md)を参照してください。

#### クエリエンジン

<!-- - [プレビュー] Supports operator **spilling** for large queries, which can use disk space to ensure stable running of queries in case of insufficient memory. -->
- 結合されたテーブルに対するクエリが[クエリキャッシュ](../using_starrocks/query_cache.md)の恩恵をより多く受けられるようになりました。例えば、クエリキャッシュはBroadcast JoinやBucket Shuffle Joinをサポートするようになりました。
- [Global UDFs](../sql-reference/sql-functions/JAVA_UDF.md)をサポートします。
- 動的適応並列処理: StarRocksはクエリの同時実行性に合わせて`pipeline_dop`パラメータを自動的に調整できます。

#### SQLリファレンス

- 権限関連のSQLステートメントとして[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)、[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)、[SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)を追加しました。
- 半構造化データ分析関数として[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)を追加しました。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)はORDER BYをサポートします。
- ウィンドウ関数[lead](../sql-reference/sql-functions/Window_function.md#lead)と[lag](../sql-reference/sql-functions/Window_function.md#lag)はIGNORE NULLSをサポートします。
- 文字列関数として[replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md)、[hex_decode_string()](../sql-reference/sql-functions/string-functions/hex_decode_string.md)を追加しました。
- 暗号化関数として[base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md)と[base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md)を追加しました。
- 数学関数として[sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md)、[tanh](../sql-reference/sql-functions/math-functions/tanh.md)を追加しました。
- ユーティリティ関数[current_role](../sql-reference/sql-functions/utility-functions/current_role.md)を追加しました。

### 改善点

#### デプロイメント

- バージョン3.0のDockerイメージと関連する[Dockerデプロイメントドキュメント](../quick_start/deploy_with_docker.md)を更新しました。[#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### ストレージエンジンとデータ取り込み

- データ取り込み用のCSVパラメーターを拡張し、SKIP_HEADER、TRIM_SPACE、ENCLOSE、ESCAPEをサポートしました。[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。
- [Primary Keyテーブル](../table_design/table_types/primary_key_table.md)では、プライマリキーとソートキーが分離されています。テーブル作成時に`ORDER BY`でソートキーを別途指定できます。
- 大量データ取り込み、部分更新、永続プライマリインデックスなどのシナリオで、プライマリキーテーブルへのデータ取り込みのメモリ使用量を最適化しました。
- 非同期INSERTタスクの作成をサポートします。詳細は[INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert)および[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を参照してください。[#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### マテリアライズドビュー

- [マテリアライズドビュー](../using_starrocks/Materialized_view.md)の書き換え機能を最適化しました。以下をサポートします：
  - View Delta Join、Outer Join、Cross Joinの書き換え。
  - パーティションを使用したUnionのSQL書き換えの最適化。
- マテリアライズドビューの構築機能を改善し、CTE、select *、Unionをサポートしました。
- [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)によって返される情報を最適化しました。
- MVパーティションのバッチ追加をサポートし、マテリアライズドビュー構築時のパーティション追加の効率を向上させました。[#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### クエリエンジン

- すべてのオペレーターがパイプラインエンジンでサポートされています。パイプライン以外のコードは将来のバージョンで削除されます。
- [ビッグクエリの位置特定](../administration/monitor_manage_big_queries.md)を改善し、ビッグクエリログを追加しました。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)はCPUとメモリ情報の表示をサポートします。
- Outer Joinの順序変更を最適化しました。
- SQL解析ステージでのエラーメッセージを最適化し、より正確なエラー位置とより明確なエラーメッセージを提供しました。

#### データレイクアナリティクス

- メタデータ統計の収集を最適化しました。
- [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を使用して、外部カタログによって管理され、Apache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに格納されているテーブルの作成ステートメントを表示することをサポートします。

### バグ修正

- StarRocksのソースファイルのライセンスヘッダーにある一部のURLにアクセスできませんでした。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- SELECTクエリ中に不明なエラーが返されることがありました。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- SHOW/SET CHARACTERをサポートしました。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- 読み込まれたデータがStarRocksがサポートするフィールド長を超えた場合、返されるエラーメッセージが正しくありませんでした。[#14](https://github.com/StarRocks/DataX/issues/14)
- `show full fields from 'table'`をサポートしました。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- パーティションプルーニングがMVの書き換えに失敗する原因となりました。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- CREATE MATERIALIZED VIEWステートメントに`count(distinct)`が含まれ、それがDISTRIBUTED BY列に適用された場合、MVの書き換えが失敗しました。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- VARCHARカラムがマテリアライズドビューのパーティション列として使用された場合、FEの起動に失敗しました。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- ウィンドウ関数[LEAD](../sql-reference/sql-functions/Window_function.md#lead)と[LAG](../sql-reference/sql-functions/Window_function.md#lag)がIGNORE NULLSを正しく処理していませんでした。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 一時パーティションの追加が自動パーティション作成と競合しました。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 挙動の変更

- 新しいロールベースのアクセス制御（RBAC）システムは、以前の権限とロールをサポートしますが、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)や[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)などの関連ステートメントの構文が変更されました。
- SHOW MATERIALIZED VIEWを[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)に名称変更しました。
- 以下の[予約語](../sql-reference/sql-statements/keywords.md)を追加しました：AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### アップグレードノート

v2.5からv3.0へのアップグレード、またはv3.0からv2.5へのダウングレードが可能です。

> 理論上は、v2.5より前のバージョンからのアップグレードもサポートされています。システムの可用性を確保するために、まずクラスタをv2.5にアップグレードしてからv3.0にアップグレードすることを推奨します。

v3.0からv2.5へダウングレードする際には、以下の点に注意してください。

#### BDBJE

StarRocksはv3.0でBDBライブラリをアップグレードしました。しかし、BDBJEはロールバックできません。ダウングレード後もv3.0のBDBライブラリを使用する必要があります。以下の手順を実行してください：

1. FEパッケージをv2.5のものに置き換えた後、v3.0の`fe/lib/starrocks-bdb-je-18.3.13.jar`をv2.5の`fe/lib`ディレクトリにコピーします。

2. `fe/lib/je-7.*.jar`を削除します。

#### 権限システム

v3.0へのアップグレード後はデフォルトで新しいRBAC権限システムが使用されます。ダウングレードできるのはv2.5のみです。

ダウングレード後は、[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) コマンドを実行して新しいイメージを作成し、その新しいイメージが全てのフォロワー FE に同期されるまで待つ必要があります。このコマンドを実行しない場合、ダウングレード操作の一部が失敗する可能性があります。このコマンドはバージョン 2.5.3 以降でサポートされています。

v2.5 と v3.0 の権限システムの違いについては、[StarRocks がサポートする権限](../administration/privilege_item.md)の「アップグレードノート」を参照してください。
