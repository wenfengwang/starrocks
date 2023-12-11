---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 3.0

## 3.0.8

リリース日: 2023年11月17日

## 改善点

- システムデータベース`INFORMATION_SCHEMA`の`COLUMNS`ビューに、ARRAY、MAP、およびSTRUCTの列を表示できるようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

## バグ修正

以下の問題が修正されました:

- `show proc '/current_queries';`を実行している際にクエリが実行され、その間にBEがクラッシュすることがあります。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- プライマリキーのテーブルに頻繁に指定されたソートキーでデータが連続してロードされる場合、コンパクションの失敗が発生することがあります。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- Broker Load ジョブでフィルタリング条件が指定されている場合、特定の状況下でデータロード中にBEがクラッシュする可能性があります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTSを実行した際に不明なエラーが報告されます。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- `cast()`関数でターゲットデータタイプが元のデータタイプと同じ場合、特定のデータ型に関してBEがクラッシュする可能性があります。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- BINARYまたはVARBINARYデータ型の`DATA_TYPE`および`COLUMN_TYPE`が`information_schema.columns`ビューで`unknown`として表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- プライマリキーテーブルに持続的インデックスが有効な状態で長時間かつ頻繁にデータをロードすると、BEがクラッシュすることがあります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効な場合、クエリ結果が正しくありません。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- クラスターが再起動されると、復元されたテーブルのデータがバックアップされる前のテーブルのデータと一貫しない場合があります。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- RESTOREが実行されている際にCompactionが行われると、BEがクラッシュする可能性があります。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

リリース日: 2023年10月18日

### 改善点

- ウィンドウ関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPがORDER BY句とウィンドウ句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- プライマリキーのテーブルにデータを書き込むロードジョブのPublishフェーズが非同期モードから同期モードに変更されました。そのため、ロードジョブが終了した後すぐにロードされたデータをクエリできるようになりました。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- クエリ実行中にDECIMAL型のデータで桁あふれが発生した場合、NULLの代わりにエラーが返されます。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドを実行した場合、MySQLと一貫した結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 単一のパーティション列を使用するRANGEパーティションで使用されるStarRocksテーブルでは、パーティション列の式を含むSQL述語もパーティションの絞り込みに使用できるようになりました。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### バグ修正

以下の問題が修正されました:

- データベースやテーブルを同時に作成および削除すると、特定の場合にテーブルが見つからず、そのテーブルへのデータロードが失敗することがあります。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 特定の場合にUDFの使用がメモリリークを引き起こす可能性があります。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- ORDER BY句に集約関数が含まれていると、"java.lang.IllegalStateException: null"というエラーが返されます。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- Tencent COSに格納されたデータにクエリを実行する場合、クエリ結果が正しくありません。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- ARRAY&lt;STRUCT&gt;タイプのデータのSTRUCTの一部のサブフィールドが欠落している場合、欠落したサブフィールドにデフォルト値が入力されたクエリ中にデータの長さが誤ってしまい、BEがクラッシュすることがあります。
- Berkeley DB Java Editionのバージョンがアップグレードされ、セキュリティ脆弱性を回避します。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- truncate操作とクエリが同時に行われるプライマリキーテーブルにデータをロードする場合、特定のケースで"java.lang.NullPointerException"というエラーが発生します。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- スキーマ変更の実行時間が長すぎる場合、指定されたバージョンのタブレットがガベージ収集されるため失敗することがあります。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- `NOT NULL`に設定された列にデータをロードする場合、default値が指定されていない場合、"Unsupported dataFormat value is : \N"というエラーが発生します。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- StarRocks共有データクラスターでは、`information_schema.COLUMNS`にテーブルキーの情報が記録されず、Flink Connectorを使用してデータをロードする際にDELETE操作を実行できません。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- アップグレード中に特定の列の型もアップグレードされる（例: Decimal型からDecimal v3型に）場合、特定の特性を持つテーブルのコンパクションがBEがクラッシュする可能性があります。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードする際、高並行のロードジョブが存在し、HTTPスレッド数とスキャンスレッド数が上限に達した場合、ロードジョブが予期せず中断されることがあります。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlが呼び出されるとBEがクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- BITMAP型の列がプライマリキーテーブルに追加された場合、エラーが発生します。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

リリース日: 2023年9月12日

### 動作の変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) 関数を使用する際には、SEPARATORキーワードを使用して区切り文字を宣言する必要があります。

### 新機能

- 集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) がDISTINCTキーワードとORDER BY句をサポートするようになりました。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- パーティション内のデータを経過時間に応じて自動的に冷却することができるようになりました（この機能は [list partitioning](../table_design/list_partitioning.md) ではサポートされていません）。[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改善点

- WHERE句のすべての複合述語とすべての式に対して暗黙の変換をサポートするようになりました。[session variable](../reference/System_variable.md) `enable_strict_type` を使用して、暗黙の変換を有効または無効にすることができます。このセッション変数のデフォルト値は `false` です。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 文字列を整数に変換するときのFEとBEの間のロジックを統一しました。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### バグ修正

- `enable_orc_late_materialization` が`true`に設定されている場合、Hiveカタログを使用してORCファイル内のSTRUCT型データをクエリする際に予期しない結果が返されます。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- Hiveカタログを介してデータクエリを実行する際に、WHERE句でパーティション列とOR演算子が指定されている場合、クエリ結果が正しくありません。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- クラウドネイティブテーブルのRESTful APIアクション`show_data`が返す値が正しくありません。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- [共有データクラスター](../deployment/shared_data/azure.md)がAzure Blob Storageにデータを格納し、テーブルが作成されると、クラスターがバージョン3.0にロールバックされるとFEが起動しなくなります。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- ユーザーがIcebergカタログのテーブルをクエリする際に、そのテーブルに対する権限が付与されていても、ユーザーは権限を持っていません。 [#29173](https://github.com/StarRocks/starrocks/pull/29173)
- [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md)ステートメントによって返される[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)または[HLL](../sql-reference/sql-statements/data-types/HLL.md)データ型の列の`Default`フィールド値が不正確です。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- `ADMIN SET FRONTEND CONFIG`コマンドを使用してFEダイナミックパラメーター`max_broker_load_job_concurrency`を変更しても、効果がありません。
- マテリアライズドビューがリフレッシュされている間に、そのリフレッシュ戦略が変更されると、FEの起動に失敗する可能性があります。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- `select count(distinct(int+double)) from table_name`が実行されると、エラー`unknown error`が返されます。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- 主キーテーブルが復元された後、BEが再起動されると、メタデータエラーが発生し、メタデータの不整合が発生します。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

リリース日: 2023年8月16日

### 新機能

- 集約関数[COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、および[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md)がサポートされています。
- 次の[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md)がサポートされています: COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMP。

### 改善

- エラーメッセージ`xxx too many versions xxx`にさらにプロンプトが追加されました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 動的パーティション化はさらに、パーティション単位を年にサポートします。[#28386](https://github.com/StarRocks/starrocks/pull/28386)
- テーブル作成時に式パーティショニングが使用され、[INSERT OVERWRITEで特定のパーティションのデータを上書き](../table_design/expression_partitioning.md#load-data-into-partitions)する場合、パーティションフィールドは大文字と小文字を区別しません。[#28309](https://github.com/StarRocks/starrocks/pull/28309)

### バグ修正

次の問題が修正されました:

- FEでテーブルレベルのスキャン統計が不正確なため、テーブルクエリやローディングのメトリクスが正確でない。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- パーティションテーブルのソートキーが変更された場合、クエリ結果は安定しない。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- データが復元された後、タブレットのBEとFEでバージョン番号が一貫しない。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- Colocationテーブルを作成する際にバケット数が指定されていない場合、数値は0と推定され、新しいパーティションの追加に失敗します。[#27086](https://github.com/StarRocks/starrocks/pull/27086)
- INSERT INTO SELECTのSELECT結果が空の場合、SHOW LOADによって返されるロードジョブのステータスは`CANCELED`です。[#26913](https://github.com/StarRocks/starrocks/pull/26913)
- sub_bitmap関数の入力値がBITMAP型でない場合、BEがクラッシュする可能性があります。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- AUTO_INCREMENT列を更新している最中にBEがクラッシュする場合があります。[#27199](https://github.com/StarRocks/starrocks/pull/27199)
- マテリアライズドビューの外部結合およびアンチ結合の再記述エラー。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 平均行サイズの見積もりが不正確であり、プライマリキーの部分的な更新が過度に大きなメモリを占有してしまいます。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 非アクティブなマテリアライズドビューのアクティブ化によって、FEがクラッシュする場合があります。[#27959](https://github.com/StarRocks/starrocks/pull/27959)
- Hudiカタログの外部テーブルベースで作成されたマテリアライズドビューによってリライトされても、クエリをリライトすることはできません。[#28023](https://github.com/StarRocks/starrocks/pull/28023)
- Hiveテーブルのデータは、テーブルが削除され、メタデータキャッシュが手動で更新された後でもクエリできます。[#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 非同期マテリアライズドビューを手動で更新すると、複数のINSERT OVERWRITEレコードが`information_schema.task_runs`テーブルに挿入されます。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- ブロックされたLabelCleanerスレッドによって引き起こされるFEメモリリーク。[#28311](https://github.com/StarRocks/starrocks/pull/28311)

## 3.0.4

リリース日:2023年7月18日

### 新機能

クエリに異なる種類の結合が含まれている場合でも、クエリがマテリアライズドビューと異なる種類の結合に再書き換えられることができます。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 改善

- 非同期マテリアライズドビューの手動リフレッシュが最適化されました。REFRESH MATERIALIZED VIEW WITH SYNC MODE構文を使用して、マテリアライズドビューのリフレッシュタスクを同期的に呼び出すことがサポートされています。[#25910](https://github.com/StarRocks/starrocks/pull/25910)
- クエリによって取得されたフィールドが出力列に含まれていないが、マテリアライズドビューの述語に含まれている場合、クエリは依然としてマテリアライズドビューから利益を得るために再書き換えることができます。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- SQL方言(`sql_dialect`)が`trino`に設定されている場合、テーブルエイリアスは大文字と小文字を区別しません。[#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- `Information_schema.tables_config`テーブルに`table_id`という新しいフィールドが追加されました。データベース`Information_schema`のテーブル`be_tablets`と`tables_config`を列`table_id`で結合して、タブレットが属するデータベースとテーブルの名前をクエリすることができます。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### バグ修正

次の問題が解決されました:

- 含まれているテーブルが単一テーブルのマテリアライズドビューから直接クエリ結果を取得するように書き換えられる場合、sum()フィールドの値が型の推論の問題によって不正確になることがあります。[#25512](https://github.com/StarRocks/starrocks/pull/25512)
- StarRocks共有データクラスターのタブレットに関する情報を表示するためにSHOW PROCが使用された場合にエラーが発生します。
- 挿入されるCHARデータの長さが最大長を超える場合、INSERT操作が保留状態になります。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- FULL JOINを使用したINSERT INTO SELECTにおいて、一部のデータ行が返されない場合があります。[#26603](https://github.com/StarRocks/starrocks/pull/26603)
- ALTER TABLEステートメントが使用されてテーブルのプロパティ`default.storage_medium`を変更しようとすると、エラー`ERROR xxx: Unknown table property xxx`が発生します。[#25870](https://github.com/StarRocks/starrocks/issues/25870)
- 空のファイルをロードするためにブローカーロードを使用すると、エラーが発生します。[#26212](https://github.com/StarRocks/starrocks/pull/26212)
- BEの退役が時々保留状態になります。[#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

リリース日: 2023年6月28日

### 改善

- StarRocks外部テーブルのメタデータ同期がデータロード中に変更されました。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- ユーザーは、テーブルのINSERT OVERWRITEを実行する際に、パーティションを指定できます。パーティションは自動的に作成されます。詳細については、[自動パーティショニング](https://docs.starrocks.io/en-us/3.0/table_design/dynamic_partitioning)を参照してください。[#25005](https://github.com/StarRocks/starrocks/pull/25005)
- パーティションを非パーティションテーブルに追加する際のエラーメッセージを最適化しました。[#25266](https://github.com/StarRocks/starrocks/pull/25266)

### バグ修正

以下の問題を修正しました：

- Parquetファイルに複雑なデータ型が含まれる場合、min/maxフィルタが誤ったParquetフィールドを取得する問題を修正しました。[#23976](https://github.com/StarRocks/starrocks/pull/23976)
- 関連するデータベースまたはテーブルが削除された場合でも、ロードタスクがまだキューに入れられている問題を修正しました。[#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FEの再起動が原因でBEがクラッシュする可能性が低い場合があります。[#25037](https://github.com/StarRocks/starrocks/pull/25037)
- 変数`enable_profile`が`true`に設定されている場合、ロードおよびクエリジョブが凍結することがあります。[#25060](https://github.com/StarRocks/starrocks/pull/25060)
- 3つ未満の生きているBEを持つクラスタでINSERT OVERWRITEが実行された場合、不正確なエラーメッセージが表示される問題を修正しました。[#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

リリース日：2023年6月13日

### 改善点

- UNIONクエリの述語は、非同期マテリアライズドビューによってクエリが再構築された後にプッシュダウンされることができます。[#23312](https://github.com/StarRocks/starrocks/pull/23312)
- テーブルの[自動タブレット分配ポリシー](../table_design/Data_distribution.md#determine-the-number-of-buckets)を最適化しました。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- NetworkTimeがサーバー間で一貫性のないシステムクロックによって不正確になることを修正するため、NetworkTimeのシステムクロックに対する依存を削除しました。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### バグ修正

以下の問題を修正しました：

- スキーマ変更が同時にデータロードが発生した場合、スキーマ変更が一部場合にハングすることがあります。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- セッション変数`pipeline_profile_level`が`0`に設定されている場合、クエリがエラーに遭遇します。[#23873](https://github.com/StarRocks/starrocks/pull/23873)
- `cloud_native_storage_type`が`S3`に設定されている場合、CREATE TABLEがエラーに遭遇します。
- パスワードを使用しない場合でもLDAP認証が成功します。[#24862](https://github.com/StarRocks/starrocks/pull/24862)
- LOADジョブに関与するテーブルが存在しない場合、CANCEL LOADが失敗します。[#24922](https://github.com/StarRocks/starrocks/pull/24922)

### アップグレード注意事項

システムに`starrocks`という名前のデータベースがある場合、アップグレード前にALTER DATABASE RENAMEを使用して別の名前に変更してください。これは、`starrocks`が特権情報を保存するデフォルトのシステムデータベースの名前であるためです。

## 3.0.1

リリース日：2023年6月1日

### 新機能

- [プレビュー] 大規模な演算子の中間計算結果をディスクにスピルして大規模な演算子のメモリ消費を減らすことをサポートします。詳細については、[Spill to disk](../administration/spill_to_disk.md)を参照してください。
- [ルーチンロード](../loading/RoutineLoad.md#load-avro-format-data)は、Avroデータのロードをサポートします。
- [Microsoft Azure Storage](../integrations/authenticate_to_azure_storage.md)（Azure Blob StorageおよびAzure Data Lake Storageを含む）をサポートします。

### 改善点

- 共有データクラスタは、StarRocks外部テーブルを使用して他のStarRocksクラスタとデータを同期することをサポートします。
- [情報スキーマ](../reference/information_schema/load_tracking_logs.md)に`load_tracking_logs`を追加し、最近のロードエラーを記録します。
- CREATE TABLEステートメントの特殊文字を無視するようにしました。[#23885](https://github.com/StarRocks/starrocks/pull/23885)

### バグ修正

以下の問題を修正しました：

- Primary Keyテーブルに関するSHOW CREATE TABLEによって返される情報が正しくありません。[#24237](https://github.com/StarRocks/starrocks/issues/24237)
- ルーチンロードジョブ中にBEがクラッシュする可能性があります。[#20677](https://github.com/StarRocks/starrocks/issues/20677)
- パーティションを作成する際にサポートされていないプロパティを指定すると、Nullポインタ例外（NPE）が発生します。[#21374](https://github.com/StarRocks/starrocks/issues/21374)
- SHOW TABLE STATUSによって返される情報が不完全です。[#24279](https://github.com/StarRocks/starrocks/issues/24279)

### アップグレード注意事項

システムに`starrocks`という名前のデータベースがある場合、アップグレード前にALTER DATABASE RENAMEを使用して別の名前に変更してください。これは、`starrocks`が特権情報を保存するデフォルトのシステムデータベースの名前であるためです。

## 3.0.0

リリース日：2023年4月28日

### 新機能

#### システムアーキテクチャ

- **ストレージとコンピューティングの切り離し。** StarRocksは現在、データをS3互換のオブジェクトストレージに永続化サポートし、リソースの分離を強化し、ストレージコストを削減し、コンピューティングリソースをよりスケーラブルにします。クエリパフォーマンスは、ローカルディスクキャッシュがヒットした場合、新しい共有データアーキテクチャにおけるクエリパフォーマンスがクラシックアーキテクチャ（共有なし）と同等です。詳細については、[共有データStarRocksのデプロイと使用](../deployment/shared_data/s3.md)を参照してください。

#### ストレージエンジンおよびデータ取り込み

- [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md)属性をサポートして、グローバルに一意のIDを提供し、データ管理を簡素化します。
- [自動パーティションおよびパーティション式](../table_design/dynamic_partitioning.md)をサポートして、パーティションの作成をより使いやすく、柔軟にします。
- Primary Keyテーブルは、CTEの使用と複数のテーブルへの参照を含むより完全な[UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md)および[DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md#primary-key-tables)構文をサポートします。
- Broker LoadおよびINSERT INTOジョブ用のLoad Profileを追加しました。ロードプロファイルをクエリしてロードジョブの詳細を表示できます。使用方法は[クエリプロファイルを分析](../administration/query_profile.md)と同じです。

#### データレイクアナリティクス

- [プレビュー] Presto/Trino互換の方言をサポートします。Presto/TrinoのSQLを自動的にStarRocksのSQLパターンに書き直すことができます。詳細については、[システム変数](../reference/System_variable.md)`sql_dialect`を参照してください。
- [プレビュー] [JDBCカタログ](../data_source/catalog/jdbc_catalog.md)をサポートします。
- 現在のセッションでカタログを手動で切り替えるために[SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md)の使用をサポートします。

#### 特権およびセキュリティ

- フルのRBAC機能を持つ新しい特権システムを提供し、ロールの継承とデフォルトロールをサポートします。詳細については、[特権の概要](../administration/privilege_overview.md)を参照してください。
- より多くの特権管理オブジェクトとより細かい特権を提供します。詳細については、[StarRocksがサポートする特権](../administration/privilege_item.md)を参照してください。

#### クエリエンジン

<!-- - 大規模なクエリのためのオペレータ**スピリング**をサポートし、不足しているメモリの場合にクエリの安定した実行を保証するためにディスクスペースを使用できます。 -->
- より多くの結合テーブルのクエリが[クエリキャッシュ](../using_starrocks/query_cache.md)から利益を得ることができるようになりました。例えば、クエリキャッシュは今やBroadcast JoinやBucket Shuffle Joinをサポートします。
- [Global UDFs](../sql-reference/sql-functions/JAVA_UDF.md)をサポートします。
- 動的な適応並列処理：StarRocksはクエリ同時性のための`pipeline_dop`パラメータを自動的に調整できます。

#### SQLリファレンス

- 次の特権関連のSQLステートメントを追加しました：[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)、[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)、および[SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 次の半構造化データ分析関数を追加しました：[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)、[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md)がORDER BYをサポートします。
- Window関数[lead](../sql-reference/sql-functions/Window_function.md#lead)および[lag](../sql-reference/sql-functions/Window_function.md#lag)がIGNORE NULLSをサポートします。
- 文字列関数[replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md)、および[hex_decode_string()](../sql-reference/sql-functions/string-functions/hex_decode_string.md)を追加しました。
- 暗号化機能 [base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md) と [base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md) を追加しました。
- 数学関数 [sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md)、[tanh](../sql-reference/sql-functions/math-functions/tanh.md) を追加しました。
- ユーティリティ関数 [current_role](../sql-reference/sql-functions/utility-functions/current_role.md) を追加しました。

### 改善

#### デプロイ

- バージョン3.0のためのDockerイメージと関連する[Dockerデプロイメントドキュメント](../quick_start/deploy_with_docker.md)を更新しました。 [#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

#### ストレージエンジンとデータ取込

- SKIP_HEADER、TRIM_SPACE、ENCLOSE、ESCAPEを含むデータ取り込みのためのCSVパラメータをサポートしています。 [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。
- [Primary Keyテーブル](../table_design/table_types/primary_key_table.md)において主キーとソートキーを分離しました。テーブルを作成する際に`ORDER BY`でソートキーを別々に指定できます。
- 大量のデータ取り込み、部分更新、および永続的な主キーインデックスなどのシナリオでのPrimary Keyテーブルへのデータ取り込みのメモリ使用量を最適化しました。
- 非同期INSERTタスクの作成をサポートしています。詳細については、[INSERT](../loading/InsertInto.md#load-data-asynchronously-using-insert) および [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) を参照してください。[#20609](https://github.com/StarRocks/starrocks/issues/20609)

#### マテリアライズドビュー

- [マテリアライズドビュー](../using_starrocks/Materialized_view.md)のリライト機能を最適化しました。以下を含む:
  - View Delta Join、Outer Join、Cross Joinのリライトをサポートしています。
  - パーティションのあるUnionのSQLリライトを最適化しました。
- マテリアライズドビューの構築機能を改善しました。CTE、select *、およびUnionをサポートしています。
- [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) によって返される情報を最適化しました。
- マテリアライズドビューの構築中にパーティションを一括で追加することをサポートし、マテリアライズドビューの構築中のパーティション追加の効率を向上させました。[#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### クエリエンジン

- すべての演算子をパイプラインエンジンでサポートしています。ノンパイプラインコードは後のバージョンで削除されます。
- [Big Query Positioning](../administration/monitor_manage_big_queries.md)を改善し、big queryログを追加しました。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) はCPUおよびメモリ情報の表示をサポートします。
- Outer Join Reorderを最適化しました。
- SQL解析段階でのエラーメッセージを最適化し、より正確なエラー位置とより明確なエラーメッセージを提供します。

#### データレイクアナリティクス

- メタデータ統計の収集を最適化しました。
- 外部カタログによって管理されていてApache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに保存されているテーブルの作成ステートメントを表示するために[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を使用することをサポートしています。

### バグ修正

- StarRocksのソースファイルのライセンスヘッダ内の一部のURLにアクセスできません。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- SELECTクエリー中に未知のエラーが返されます。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- SHOW/SET CHARACTERをサポートしています。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- StarRocksがサポートするフィールド長を超えるデータがロードされた場合、返されるエラーメッセージが正しくありません。[#14](https://github.com/StarRocks/DataX/issues/14)
- `show full fields from 'table'` をサポートしています。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- パーティションプルーニングにより、MVリライトに失敗します。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- CREATE MATERIALIZED VIEWステートメントに`count(distinct)`が含まれ、`count(distinct)`がDISTRIBUTED BY列に適用されている場合、MVリライトが失敗します。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- FeがVARCHAR列をマテリアライズドビューのパーティショニング列として使用すると、FEが起動しなくなります。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- Window関数 [LEAD](../sql-reference/sql-functions/Window_function.md#lead) および [LAG](../sql-reference/sql-functions/Window_function.md#lag) は、IGNORE NULLSを誤って処理しています。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 一時的なパーティションを追加することが自動的なパーティション作成と競合しています。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 動作の変更

- 新しいロールベースのアクセス制御（RBAC）システムは以前の権限とロールをサポートしています。ただし、 [GRANT](../sql-reference/sql-statements/account-management/GRANT.md)、[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) などの関連ステートメントの構文が変更されています。
- [SHOW MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) を [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) に名前変更しました。
- 次の[予約語](../sql-reference/sql-statements/keywords.md)を追加しました: AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### アップグレードノート

v2.5 から v3.0 にアップグレードするか、v3.0 から v2.5 にダウングレードすることができます。

> 理論的には、v2.5より古いバージョンからのアップグレードもサポートされています。システムの可用性を確保するため、まずクラスターをv2.5にアップグレードし、その後v3.0にアップグレードすることをお勧めします。

v3.0 から v2.5 にダウングレードを行う際には、以下の点に注意してください。

#### BDBJE

StarRocksはv3.0でBDBライブラリをアップグレードしました。ただし、BDBJEはダウングレードできません。ダウングレード後はv3.0のBDBライブラリを使用する必要があります。以下の手順を実行してください:

1. FEパッケージをv2.5のパッケージに置き換えた後、`fe/lib/starrocks-bdb-je-18.3.13.jar`をv3.0の`fe/lib`ディレクトリにコピーします。

2. `fe/lib/je-7.*.jar`を削除します。

#### 権限システム

新しいRBAC権限システムは、v3.0にアップグレード後にデフォルトで使用されます。v2.5 にのみダウングレードできます。

ダウングレード後、[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成し、新しいイメージがすべてのフォロワーFEに同期されるのを待ちます。このコマンドは2.5.3以降でサポートされています。

v2.5 および v3.0の権限システムの違いについての詳細は、「Upgrade notes」の [StarRocksがサポートする権限](../administration/privilege_item.md)を参照してください。