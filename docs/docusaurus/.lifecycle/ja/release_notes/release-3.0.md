```yaml
---
displayed_sidebar: "Japanese"
---

# StarRocksバージョン3.0

## 3.0.8

リリース日: 2023年11月17日

## 改良

- システムデータベース`INFORMATION_SCHEMA`の`COLUMNS`ビューにARRAY、MAP、STRUCT列を表示できるようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

## バグ修正

以下の問題が修正されました:

- `show proc '/current_queries';`を実行している最中にクエリが実行されると、BEがクラッシュする可能性があります。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- プライマリキーのテーブルに高頻度で指定されたソートキーで連続的にデータがロードされると、コンパクションの失敗が発生する可能性があります。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- ブローカーロードジョブでフィルタリング条件が指定されている場合、特定の状況でデータロード中にBEがクラッシュする可能性があります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTSを実行した際に未知のエラーが報告されます。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- `cast()`関数でターゲットデータ型が元のデータ型と同じ場合、特定のデータ型の場合にBEがクラッシュする可能性があります。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- `information_schema.columns`ビューでBINARYまたはVARBINARYデータ型の`DATA_TYPE`および`COLUMN_TYPE`が`unknown`と表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 持続的インデックスが有効なプライマリキーテーブルに長時間および頻繁なデータロードが行われた場合、BEがクラッシュする可能性があります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効になっている場合、クエリ結果が正しくありません。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- クラスターが再起動された後、復元されたテーブルのデータがバックアップされる前のテーブルのデータと一貫しない場合があります。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- RESTOREが実行され、同時にCompactionが行われると、BEがクラッシュする可能性があります。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

リリース日: 2023年10月18日

### 改良

- ウィンドウ関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPは、ORDER BY句とウィンドウ句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- データをプライマリキーテーブルに書き込むロードジョブのPublishフェーズを非同期モードから同期モードに変更しました。そのため、ロードジョブの完了後すぐにデータをクエリできるようになります。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- クエリ中にDECIMAL型データで10進オーバーフローが発生した場合、NULLではなくエラーが返されます。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドを実行した際、MySQLと一貫した結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 1つのパーティション列のみを使用するRANGEパーティションを使用するStarRocksテーブルの場合、パーティション列式を含むSQL述語をパーティションの剪定に使用できるようになりました。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### バグ修正

以下の問題が修正されました:

- データベースやテーブルを同時に作成および削除すると、一部の場合、テーブルが見つからず、それによりそのテーブルにデータロードが失敗する可能性があります。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 特定の状況でUDFの使用によりメモリリークが発生する可能性があります。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- ORDER BY句に集約関数が含まれている場合、"java.lang.IllegalStateException: null"というエラーが返されます。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- ユーザーがHiveカタログを使用してTencent COSに格納されたデータにクエリを実行すると、クエリ結果が正しくありません。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- ARRAY&lt;STRUCT&gt;タイプデータのSTRUCTの一部のサブフィールドが欠落している場合、クエリ中に欠落したサブフィールドにデフォルト値が埋められることにより、データ長が不正確になり、BEがクラッシュする可能性があります。
- Berkeley DB Java Editionのバージョンがアップグレードされ、セキュリティの脆弱性を回避するようになりました。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- truncate操作とクエリが同時に実行されるプライマリキーテーブルにデータをロードすると、特定の場合に"java.lang.NullPointerException"というエラーが発生します。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- スキーマ変更の実行時間が長すぎる場合、指定されたバージョンのタブレットがガーベージコレクションされるため、失敗する可能性があります。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- `NOT NULL`に設定されたテーブル列にCloudCanalを使用してデータをロードする場合、"Unsupported dataFormat value is : \N"というエラーが発生します。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- StarRocks共有データクラスターでは、`information_schema.COLUMNS`にテーブルキーの情報が記録されないため、Flink Connectorを使用してデータをロードする際にDELETE操作を実行できません。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- アップグレード中に特定の列の型もアップグレードされる場合（例: Decimal型からDecimal v3型に）、特定の特性を持つテーブルのコンパクションによりBEがクラッシュする可能性があります。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードする際、高度に同時実行されるロードジョブがあり、HTTPスレッドの数とスキャンスレッドの数が上限に達した場合、ロードジョブが予期せず中断されます。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlが呼び出されると、BEがクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- BITMAPタイプの列がプライマリキーテーブルに追加されると、エラーが発生します。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

リリース日: 2023年9月12日

### 挙動の変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)関数を使用する場合、区切り文字を宣言するためにSEPARATORキーワードを使用する必要があります。

### 新機能

- 集約関数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)がDISTINCTキーワードとORDER BY句をサポートするようになりました。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- パーティション内のデータが時間経過とともに自動的に冷却されるようになりました。（この機能は[list partitioning](../table_design/list_partitioning.md)には対応していません。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改良

- WHERE句のすべての複合述語およびすべての式に対して暗黙の変換をサポートするようになりました。[session変数](../reference/System_variable.md)`enable_strict_type`を使用して暗黙の変換を有効または無効にできます。このセッション変数のデフォルト値は`false`です。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 文字列を整数に変換する際のロジックをFEおよびBEで統一するようになりました。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### バグ修正

- `enable_orc_late_materialization`が`true`に設定されている場合、Hiveカタログを使用してORCファイル内のSTRUCT型データをクエリする際に予期しない結果が返されます。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- Hiveカタログを介してデータクエリを実行する際、WHERE句にパーティション列とOR演算子が指定されている場合、クエリ結果が正しくありません。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- RESTful APIアクション`show_data`がクラウドネイティブテーブルの値を正しく返しません。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
```
- [共有データクラスター](../deployment/shared_data/azure.md) がAzure Blob Storageにデータを保存し、テーブルが作成されている場合、クラスターがバージョン3.0にロールバックされた後、FEは起動に失敗します。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- Icebergカタログでテーブルに対する権限を付与されているにもかかわらず、ユーザーがクエリを実行する権限がありません。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) ステートメントによって返される[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) や[HLL](../sql-reference/sql-statements/data-types/HLL.md) データ型の列の `Default` フィールド値が正しくありません。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- `ADMIN SET FRONTEND CONFIG` コマンドを使用してFEダイナミックパラメータ `max_broker_load_job_concurrency` を変更しても効果がありません。
- マテリアライズドビューのリフレッシュ中にリフレッシュ戦略が変更されている場合、FEの起動に失敗する可能性があります。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- `select count(distinct(int+double)) from table_name` を実行した際に `unknown error` が返されます。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- プライマリキーテーブルが復元された後、BEが再起動された場合、メタデータエラーが発生し、メタデータの不整合が発生します。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

リリース日: 2023年8月16日

### 新機能

- 集計関数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、および [CORR](../sql-reference/sql-functions/aggregate-functions/corr.md) をサポートします。
- 次の[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)をサポートします: COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMP。

### 改善点

- エラーメッセージ `xxx too many versions xxx` に追加のプロンプトを追加しました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- ダイナミックパーティショニングは、さらにパーティショニングユニットが年であることをサポートします。[#28386](https://github.com/StarRocks/starrocks/pull/28386)
- 表作成時の式パーティショニングと[特定のパーティションにデータを上書きするには、INSERT OVERWRITEが使用される](../table_design/expression_partitioning.md#load-data-into-partitions)際、パーティショニングフィールドは大文字と小文字を区別しません。[#28309](https://github.com/StarRocks/starrocks/pull/28309)

### バグ修正

以下の問題が修正されました:

- FEにおける不正確なテーブルレベルのスキャン統計情報のために、テーブルクエリとロードのためのメトリクスが正確でない。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- パーティション化されたテーブルのソートキーが変更された場合、クエリ結果が安定しない。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- データの復元後、タブレットのバージョン番号がBEとFEで一貫性がない。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- ユーザーがコロケーションテーブルを作成する際にバケット数を指定しない場合、その数は0と推論され、新しいパーティションの追加に失敗します。[#27086](https://github.com/StarRocks/starrocks/pull/27086)
- INSERT INTO SELECTのSELECT結果セットが空の場合、SHOW LOADによって返されるロードジョブのステータスは `CANCELED` です。[#26913](https://github.com/StarRocks/starrocks/pull/26913)
- サブビットマップ関数の入力値がBITMAPタイプでない場合、BEがクラッシュすることがあります。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- AUTO_INCREMENT列が更新されている際、BEがクラッシュすることがあります。[#27199](https://github.com/StarRocks/starrocks/pull/27199)
- マテリアライズドビューの外部結合およびアンチ結合の再記述エラー。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 平均行サイズの不正確な推定により、プライマリキー部分更新が過剰なメモリを占有します。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 非アクティブなマテリアライズドビューを有効にすると、FEがクラッシュすることがあります。[#27959](https://github.com/StarRocks/starrocks/pull/27959)
- Hudiカタログ内の外部テーブルに基づいて作成されたマテリアライズドビューに対して、クエリをマテリアライズドビューに再書き込むことができません。[#28023](https://github.com/StarRocks/starrocks/pull/28023)
- Hiveテーブルのデータが削除された後もクエリ可能です。さらに、メタデータキャッシュを手動で更新してもクエリ可能です。[#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 非同期マテリアライズドビューを同期的に呼び出しても、複数のINSERT OVERWRITEレコードが `information_schema.task_runs` テーブルに結果として登録されることがあります。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- ブロックされたLabelCleanerスレッドによるFEメモリリークが発生します。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 3.0.4

リリース日: 2023年7月18日

### 新機能

クエリに異なる種類の結合が含まれる場合でも、クエリをマテリアライズドビューから直接取得するように書き直すことができます。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改善点

- 非同期マテリアライズドビューの手動リフレッシュを最適化しました。 `REFRESH MATERIALIZED VIEW WITH SYNC MODE` 構文を使用してマテリアライズドビューリフレッシュタスクを同期的に呼び出すことをサポートします。[#25910](https://github.com/StarRocks/starrocks/pull/25910)
- クエリに出力カラムに含まれないがマテリアライズドビューの述語に含まれるクエリは、それでもマテリアライズドビューの利点を得るために書き換えることができます。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- SQLダイアレクト(`sql_dialect`)が `trino` に設定された場合、テーブルのエイリアスは大文字と小文字を区別しません。[#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- `Information_schema` データベースの中の `be_tablets` テーブルの `table_id` フィールドに `table_id` フィールドを追加しました。これにより、タブレットが属するデータベースとテーブルの名前をクエリするために、`tables_config` テーブルを `be_tablets` テーブルに結合することができます。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### バグ修正

以下の問題が修正されました:

- 合計集計関数を含むクエリが1つのテーブルのマテリアライズドビューからクエリ結果を直接取得するように書き直された場合、型推論の問題により `sum()` フィールドの値が不正確になる場合があります。[#25512](https://github.com/StarRocks/starrocks/pull/25512)
- StarRocks共有データクラスターのタブレットに関する情報を表示するためにSHOW PROCを使用するとエラーが発生します。
- 挿入操作がCHARデータのSTRUCTの長さが最大長を超える場合に、挿入が停止します。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- FULL JOINを使用したINSERT INTO SELECTでは、クエリに失敗したデータ行が返されない場合があります。[#26603](https://github.com/StarRocks/starrocks/pull/26603)
- ALTER TABLEステートメントを使用してテーブルのプロパティ `default.storage_medium` を変更しようとした際にエラー `ERROR xxx: Unknown table property xxx` が発生します。[#25870](https://github.com/StarRocks/starrocks/issues/25870)
- 空のファイルを読み込むためにBroker Loadを使用するとエラーが発生します。[#26212](https://github.com/StarRocks/starrocks/pull/26212)
- BEのデコミッショニングが停止することがあります。[#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

リリース日: 2023年6月28日

### 改善点

- StarRocksの外部テーブルのメタデータ同期がデータのロード時に変更されました。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- INSERT OVERWRITEを実行する際に、自動的に作成されるテーブルのパーティションを指定することができます。詳細は[自動パーティショニング](https://docs.starrocks.io/en-us/3.0/table_design/dynamic_partitioning)を参照してください。 [#25005](https://github.com/StarRocks/starrocks/pull/25005)
- パーティションが非パーティションテーブルに追加された場合のエラーメッセージを最適化しました。 [#25266](https://github.com/StarRocks/starrocks/pull/25266)

### バグ修正

以下の問題を修正しました:
- Parquetファイルに複雑なデータ型が含まれている場合、min/maxフィルタは間違ったParquetフィールドを取得します。 [#23976](https://github.com/StarRocks/starrocks/pull/23976)
- 関連するデータベースまたはテーブルが削除された場合でも、ロードタスクはまだキューイング中です。 [#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FEの再起動が原因でBEがクラッシュする可能性が低いです。 [#25037](https://github.com/StarRocks/starrocks/pull/25037)
- 変数 `enable_profile` が `true` に設定されている場合、ロードおよびクエリジョブが凍結することがあります。 [#25060](https://github.com/StarRocks/starrocks/pull/25060)
- 3つ以下のアクティブなBEがあるクラスタでINSERT OVERWRITEが実行された場合、不正確なエラーメッセージが表示されます。 [#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

リリース日: 2023年6月13日

### 改善

- UNIONクエリ内の述語は、非同期マテリアライズドビューによって再作成された後に押し下げることができます。 [#23312](https://github.com/StarRocks/starrocks/pull/23312)
- テーブルの[自動タブレット分布ポリシー](../table_design/Data_distribution.md#determine-the-number-of-buckets)を最適化しました。 [#24543](https://github.com/StarRocks/starrocks/pull/24543)
- システムクロックが一貫していないために、NetworkTimeが不正確になる問題を修正するために、NetworkTimeのシステム時計に対する依存関係を削除しました。 [#24858](https://github.com/StarRocks/starrocks/pull/24858)

### バグ修正

以下の問題を修正しました:
- スキーマ変更が同時にデータロードが発生した場合、スキーマ変更が時々止まることがあります。 [#23456](https://github.com/StarRocks/starrocks/pull/23456)
- セッション変数 `pipeline_profile_level` が `0` に設定されている場合、クエリがエラーになります。 [#23873](https://github.com/StarRocks/starrocks/pull/23873)
- `cloud_native_storage_type` が `S3` に設定されている場合、CREATE TABLEがエラーになります。
- パスワードを使用しない場合でもLDAP認証が成功します。 [#24862](https://github.com/StarRocks/starrocks/pull/24862)
- LOADジョブに関与するテーブルが存在しない場合、CANCEL LOADが失敗します。[#24922](https://github.com/StarRocks/starrocks/pull/24922)

### アップグレード注意事項

システムに `starrocks` という名前のデータベースがある場合、アップグレード前にALTER DATABASE RENAMEを使用して別の名前に変更してください。これは、 `starrocks` が特権情報を格納するデフォルトのシステムデータベースの名前であるためです。

## 3.0.1

リリース日: 2023年6月1日

### 新機能

- [プレビュー] 大規模な演算子の中間計算結果をディスクにスパイルして大規模な演算子のメモリ消費を削減するサポート。詳細については[Spill to disk](../administration/spill_to_disk.md)を参照してください。
- [Routine Load](../loading/RoutineLoad.md#load-avro-format-data)は、Avroデータの読み込みをサポートします。
- [Microsoft Azure Storage](../integrations/authenticate_to_azure_storage.md)（Azure Blob StorageおよびAzure Data Lake Storageを含む）をサポートします。

### 改善

- 共有データクラスタは、StarRocks外部テーブルを使用して別のStarRocksクラスタとデータ同期することをサポートします。
- [Information Schema](../reference/information_schema/load_tracking_logs.md)に `load_tracking_logs` を追加し、最近のロードエラーを記録します。
- CREATE TABLEステートメントで特殊文字を無視します。[#23885](https://github.com/StarRocks/starrocks/pull/23885)

### バグ修正

以下の問題を修正しました:
- PRIMARY KEYテーブルのSHOW CREATE TABLEによって返される情報が不正確です。[#24237](https://github.com/StarRocks/starrocks/issues/24237)
- ルーチンロードジョブ中にBEがクラッシュすることがあります。[#20677](https://github.com/StarRocks/starrocks/issues/20677)
- パーティションテーブルの作成時にサポートされていないプロパティが指定された場合、Nullポインタ例外（NPE）が発生します。[#21374](https://github.com/StarRocks/starrocks/issues/21374)
- SHOW TABLE STATUSによって返される情報が不完全です。[#24279](https://github.com/StarRocks/starrocks/issues/24279)

### アップグレード注意事項

システムに `starrocks` という名前のデータベースがある場合、アップグレード前にALTER DATABASE RENAMEを使用して別の名前に変更してください。これは、 `starrocks` が特権情報を格納するデフォルトのシステムデータベースの名前であるためです。

## 3.0.0

リリース日: 2023年4月28日

### 新機能

#### システムアーキテクチャ

- **ストレージとコンピューティングを切り離す。**StarRocksは今やS3互換オブジェクトストレージへのデータ永続化をサポートし、リソースの分離を強化し、ストレージコストを削減し、コンピューティングリソースをより拡張可能にしました。クエリパフォーマンスは、ローカルディスクキャッシュがヒットした場合には、新しい共有データアーキテクチャのクエリパフォーマンスが古典的なアーキテクチャ（共有なし）とほぼ同等になります。詳細については、[共有データStarRocksのデプロイと使用](../deployment/shared_data/s3.md)を参照してください。

#### ストレージエンジンとデータ取り込み

- ユニークなグローバルIDを提供するために[AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)属性がサポートされ、データ管理が簡素化されます。
- [自動パーティショニングおよびパーティショニング式](../table_design/dynamic_partitioning.md)がサポートされるようになり、パーティションの作成がより使いやすく柔軟になりました。
- PRIMARY KEYテーブルはより完全な[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)および[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#primary-key-tables)構文をサポートし、CTEの使用や複数のテーブルへの参照を含むようになりました。
- ブローカーロードおよびINSERT INTOジョブのためにLoad Profileを追加しました。ロードプロファイルをクエリしてロードジョブの詳細を表示できます。使用方法は[クエリプロファイルの分析](../administration/query_profile.md)と同じです。

#### データレイク分析

- [プレビュー] Presto/Trino互換の方言をサポートします。Presto/TrinoのSQLは自動的にStarRocksのSQLパターンに書き直すことができます。詳細については、[システム変数](../reference/System_variable.md) `sql_dialect`を参照してください。
- [プレビュー] [JDBCカタログ](../data_source/catalog/jdbc_catalog.md)をサポートします。
- [SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションでカタログ間を手動で切り替えることができます。

#### 権限とセキュリティ

- フルのRBAC機能を持つ新しい権限システムを提供し、ロールの継承とデフォルトロールをサポートします。詳細については、[権限の概要](../administration/privilege_overview.md)を参照してください。
- より多くの権限管理オブジェクトとより細かい粒度の権限を提供します。詳細については、[StarRocksがサポートする権限](../administration/privilege_item.md)を参照してください。

#### クエリエンジン

<!-- - [プレビュー] 大きなクエリの**スピリング**サポートを行うことで、メモリが不足した場合にクエリの安定稼働を保証するためにディスクスペースを使用することができます。 -->
- JOINされたテーブルに対するクエリキャッシュにより、[クエリキャッシュ](../using_starrocks/query_cache.md)がもっと多くのクエリを利用できるようになりました。例えば、クエリキャッシュはBroadcast JoinとBucket Shuffle Joinをサポートします。
- [Global UDFs](../sql-reference/sql-functions/JAVA_UDF.md)をサポートします。
- 動的適応並列処理: StarRocksはクエリの並列実行性の `pipeline_dop` パラメータを自動的に調整することができます。

#### SQLリファレンス

- 次の権限に関連するSQLステートメントが追加されました: [SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md), [SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md), [SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md), [SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 次の半構造化データ解析関数が追加されました: [map_apply](../sql-reference/sql-functions/map-functions/map_apply.md), [map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)はORDER BYをサポートします。
- Window関数[lead](../sql-reference/sql-functions/Window_function.md#lead)と[lag](../sql-reference/sql-functions/Window_function.md#lag)はIGNORE NULLSをサポートします。
- 文字列関数[replace](../sql-reference/sql-functions/string-functions/replace.md), [hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md),[hex_decode_string()](../sql-reference/sql-functions/string-functions/hex_decode_string.md)が追加されました。
- 暗号化関数 [base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md) と [base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md) を追加しました。
- 数学関数 [sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md)、および[tanh](../sql-reference/sql-functions/math-functions/tanh.md) を追加しました。
- ユーティリティ関数 [current_role](../sql-reference/sql-functions/utility-functions/current_role.md) を追加しました。

### 改善点

#### デプロイ

- バージョン3.0のために、Dockerイメージと関連する[Dockerデプロイメントドキュメント](../quick_start/deploy_with_docker.md)を更新しました。[#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### ストレージエンジンおよびデータ取り込み

- [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、および[ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)など、データ取り込みのためのCSVパラメータをよりサポートしました。その中には SKIP_HEADER、TRIM_SPACE、ENCLOSE、およびESCAPEも含まれます。
- [主キーの表](../table_design/table_types/primary_key_table.md) で、主キーとソートキーを分離しました。テーブルを作成する際に`ORDER BY`でソートキーを別々に指定できます。
- 大量の取り込み、部分更新、および永続的な主キーインデックスなどのシナリオで、主キーの表へのデータ取り込みのメモリ使用量を最適化しました。
- 非同期のINSERTタスクの作成をサポートしました。詳細については、[INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert)および[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を参照してください。[#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### 材料化ビュー

- [材料化ビュー](../using_starrocks/Materialized_view.md) のリライト機能を最適化しました。
  - ビューデルタ結合、アウタージョイン、およびクロスジョインのリライトをサポートしました。
  - パーティションごとのUnionのSQLリライトを最適化しました。
- 材料化ビューの構築能力を向上させました。CTE、select *、およびUnionをサポートしました。
- [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)が返す情報を最適化しました。
- 材料化ビューの構築時のパーティション追加をバッチ処理でサポートしました。これにより材料化ビューの構築時のパーティション追加の効率が向上します。[#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### クエリエンジン

- すべての演算子をパイプラインエンジンでサポートしました。非パイプラインコードは今後のバージョンで削除されます。
- [ビッグクエリのポジショニング](../administration/monitor_manage_big_queries.md)を改善し、ビッグクエリログを追加しました。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) はCPUとメモリ情報の表示をサポートします。
- アウタージョインのリオーダーを最適化しました。
- SQL解析段階でのエラーメッセージを最適化し、より正確なエラーの位置づけとより明確なエラーメッセージを提供しました。

#### データレイクアナリティクス

- メタデータ統計の収集を最適化しました。
- 外部カタログで管理され、Apache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに格納されているテーブルの作成ステートメントを表示するために[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)の使用をサポートしました。

### バグ修正

- StarRocksのソースファイルのライセンスヘッダー内の一部のURLにアクセスできません。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- SELECTクエリ実行中に不明なエラーが返されます。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- SHOW/SET CHARACTER をサポートしました。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- StarRocksでサポートされているフィールド長を超えるデータがロードされると、正しいエラーメッセージが返されません。[#14](https://github.com/StarRocks/DataX/issues/14)
- `show full fields from 'table'`をサポートしました。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- パーティションの剪定が材料化ビューのリライトに失敗する場合があります。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- CREATE MATERIALIZED VIEW ステートメントに `count(distinct)` が含まれており、 `count(distinct)` が DISTRIBUTED BY カラムに適用されると、MVのリライトに失敗します。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- VARCHAR列が材料化ビューのパーティショニング列として使用されると、FEが起動に失敗します。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- ウィンドウ関数[LEAD](../sql-reference/sql-functions/Window_function.md#lead)、[LAG](../sql-reference/sql-functions/Window_function.md#lag)がIGNORE NULLSを不正確に処理します。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 一時的なパーティションの追加が自動的なパーティションの作成と競合します。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 動作の変更

- 新しいロールベースのアクセス制御（RBAC）システムは以前の権限とロールをサポートしています。ただし、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)や[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)などの関連するステートメントの構文が変更されています。
- [SHOW MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) を [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) に名前変更しました。
- 次の[予約語](../sql-reference/sql-statements/keywords.md)が追加されました：AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### アップグレードの注意事項

v2.5 から v3.0 にアップグレードするか、v3.0 から v2.5 にダウングレードすることができます。

> 理論上、v2.5 よりも古いバージョンからのアップグレードもサポートされています。システムの可用性を確保するために、まずクラスタをv2.5にアップグレードしてからv3.0にアップグレードすることをお勧めします。

v3.0 から v2.5 にダウングレードする際には、以下のポイントに注意してください。

#### BDBJE

StarRocksはv3.0でBDBライブラリをアップグレードしました。ただし、BDBJEはロールバックできません。ダウングレード後はv3.0のBDBライブラリを使用する必要があります。以下の手順を実行してください。

1. FEパッケージをv2.5のパッケージで置き換えた後、v3.0の`fe/lib/starrocks-bdb-je-18.3.13.jar`を`fe/lib`ディレクトリにコピーします。
2. `fe/lib/je-7.*.jar`を削除します。

#### 権限システム

新しいRBAC権限システムは、v3.0にアップグレード後にデフォルトで使用されます。v2.5にダウングレードすることができます。

ダウングレード後、[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成し、新しいイメージがすべてのフォロワーFEに同期されるのを待ちます。このコマンドは、2.5.3以降でサポートされています。

v2.5とv3.0の権限システムの違いについての詳細は、「[StarRocksがサポートする権限](../administration/privilege_item.md)」の「アップグレードの注意事項」を参照してください。