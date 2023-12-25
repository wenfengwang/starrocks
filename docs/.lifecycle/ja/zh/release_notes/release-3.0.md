---
displayed_sidebar: Chinese
---

# StarRocks バージョン 3.0

## 3.0.8

リリース日：2023年11月17日

### 機能最適化

- `INFORMATION_SCHEMA.COLUMNS` テーブルが ARRAY、MAP、STRUCT 型のフィールドを表示するようにサポートしました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

### 問題修正

以下の問題を修正しました：

- `show proc '/current_queries';` を実行する際、クエリが開始されたばかりの場合、BEがクラッシュする可能性があります。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- Sort Keyを持つ主キーモデルのテーブルに連続して高頻度でデータをインポートすると、Compactionが失敗する小さな確率があります。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- LDAP認証ユーザーのパスワードを変更した後、新しいパスワードでログインするとエラーが発生します。
- Broker Loadジョブにフィルタ条件が含まれている場合、データインポート中に特定の状況でBEがクラッシュすることがあります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- `SHOW GRANTS` 実行時に `unknown error` と報告されます。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- `cast()` 関数を使用する際にデータ型の変換前後で型が同じである場合、特定の型でBEがクラッシュすることがあります。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- BINARYまたはVARBINARY型が `information_schema.columns` ビューの `DATA_TYPE` および `COLUMN_TYPE` で `unknown` と表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 永続化インデックスが開かれている主キーモデルのテーブルに長時間高頻度でデータをインポートすると、BEがクラッシュする可能性があります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- Query Cacheを有効にした後、クエリ結果に誤りがあります。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- 再起動後にRESTOREされたテーブルのデータが、BACKUP前のテーブルのデータと一部の状況で一致しないことがあります。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- RESTOREを実行する際にCompactionが同時に行われていると、BEがクラッシュする可能性があります。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

リリース日：2023年10月18日

### 機能最適化

- ウィンドウ関数 COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMPが ORDER BY句とWindow句をサポートしました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- 主キーモデルのデータ書き込み時のpublishプロセスを非同期から同期に変更し、インポートジョブが成功した後にデータが即座に表示されるようにしました。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- DECIMAL型のデータクエリ結果がオーバーフローした場合、NULLではなくエラーを返します。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 不正なコメントを含むSQLコマンドを実行すると、結果がMySQLと一致するようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 単一のパーティション列を持つRangeパーティションまたは式パーティションのStarRocksテーブルに対して、SQLステートメントの述語に含まれるパーティション列の式をパーティションプルーニングに使用できるようになりました。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### 問題修正

以下の問題を修正しました：

- データベースとテーブルの作成と削除を並行して行うと、特定の状況でテーブルが見つからずデータ書き込みエラーが発生することがあります。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 特定の状況でUDFを使用するとメモリリークが発生することがあります。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- ORDER BY句に集約関数が含まれている場合に "java.lang.IllegalStateException: null" というエラーが発生します。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- Hive Catalogが複数レベルのディレクトリであり、データが腾讯云COSに保存されている場合、クエリ結果が正しくありません。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- ARRAY&lt;STRUCT&gt;型のデータでSTRUCTの一部のサブカラムが欠落している場合、デフォルトデータの長さが誤っているためデータを読み取る際にBEがクラッシュします。[#30263](https://github.com/StarRocks/starrocks/pull/30263)
- セキュリティ上の脆弱性を避けるためにBerkeley DB Java Editionのバージョンをアップグレードしました。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- 主キーモデルのテーブルにデータをインポートする際にTruncate操作とクエリが並行して行われると、特定の状況で "java.lang.NullPointerException" エラーが発生します。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- Schema Changeの実行時間が長すぎると、Tabletのバージョンがガベージコレクションによって削除されて失敗します。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- テーブルフィールドが `NOT NULL` でデフォルト値が設定されていない場合、CloudCanalを使用してデータをインポートすると "Unsupported dataFormat value is : \N" エラーが発生します。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- ストレージ分離モードで、テーブルのKey情報が `information_schema.COLUMNS` に記録されていないため、Flink Connectorを使用してデータをインポートする際にDELETE操作が実行できません。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- 一部の列の型がアップグレードされている場合（例：DecimalがDecimal v3にアップグレード）、特定の特性を持つテーブルでCompactionを行うとBEがクラッシュします。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをインポートする際に、並行性が高くHTTPとScanスレッド数が制限されていると、ハングアップすることがあります。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlを呼び出すとBEがクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- 主キーモデルのテーブルにBITMAP型の列を追加するとエラーが発生します。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

リリース日：2023年9月12日

### 行動変更

- 集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) の区切り文字は `SEPARATOR` キーワードを使用して宣言する必要があります。

### 新機能

- 集約関数 [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md) が DISTINCT キーワードと ORDER BY 句をサポートしました。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- パーティション内のデータが時間の経過とともに自動的にコールドダウンされるようになりました（[List パーティション方式](../table_design/list_partitioning.md)はまだサポートされていません）。[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 機能最適化

- すべての複合述語およびWHERE句の式に対して暗黙の型変換をサポートし、[セッション変数](../reference/System_variable.md) `enable_strict_type` で暗黙の型変換を有効にするかどうかを制御できます（デフォルトは `false`）。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- FEとBEでSTRINGからINTへの変換ロジックを統一しました。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### 問題修正

以下の問題を修正しました：

- `enable_orc_late_materialization` が `true` に設定されている場合、Hive Catalogを使用してORCファイル内のSTRUCT型のデータをクエリすると結果が異常になります。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- Hive Catalogを使用してクエリする際に、WHERE句にパーティション列が含まれておりOR条件がある場合、クエリ結果が正しくありません。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- RESTful API `show_data` がクラウドネイティブテーブルの情報を正しく返さない問題があります。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- クラスタが[ストレージ分離アーキテクチャ](https://docs.starrocks.io/zh/docs/3.0/deployment/deploy_shared_data/)であり、データがAzure Blob Storageに保存されていて、テーブルが作成されている場合、3.0にロールバックするとFEが起動できない問題があります。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- ユーザーにIceberg Catalog下の特定のテーブルの権限を付与した後、そのユーザーがそのテーブルをクエリすると権限がないと表示される問題があります。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- [BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md) および [HLL](../sql-reference/sql-statements/data-types/HLL.md) 型の列が [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md) のクエリ結果で返される `Default` フィールドの値が正しくありません。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- FEの動的パラメータ `max_broker_load_job_concurrency` をオンラインで変更しても効果がありません。[#29964](https://github.com/StarRocks/starrocks/pull/29964) [#29720](https://github.com/StarRocks/starrocks/pull/29720)
- 物化ビューをリフレッシュする際に、物化ビューのリフレッシュポリシーを同時に変更すると、FEが起動できなくなる可能性があります。[#29691](https://github.com/StarRocks/starrocks/pull/29691)
- `select count(distinct(int+double)) from table_name` を実行すると `unknown error` エラーが発生します。[#30054](https://github.com/StarRocks/starrocks/pull/30054)
- 主キーモデルのテーブルをRestoreした後、BEを再起動するとメタデータにエラーが発生し、メタデータが一致しなくなります。[#30135](https://github.com/StarRocks/starrocks/pull/30135)

## 3.0.5

リリース日：2023年8月16日

### 新機能

- 集約関数 [COVAR_SAMP](../sql-reference/sql-functions/aggregate-functions/covar_samp.md)、[COVAR_POP](../sql-reference/sql-functions/aggregate-functions/covar_pop.md)、[CORR](../sql-reference/sql-functions/aggregate-functions/corr.md) をサポートしました。
- [ウィンドウ関数](../sql-reference/sql-functions/Window_function.md) COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、STDDEV_SAMPをサポートしました。

### 機能最適化


- エラーメッセージ `xxx too many versions xxx` に対処方法の説明を追加しました。[#28397](https://github.com/StarRocks/starrocks/pull/28397)
- 動的パーティションに年単位の粒度をサポートする機能が追加されました。[#28386](https://github.com/StarRocks/starrocks/pull/28386)
- [INSERT OVERWRITE で式ベースのパーティションを使用する場合](../table_design/expression_partitioning.md#%E5%AF%BC%E5%85%A5%E6%95%B0%E6%8D%AE%E8%87%B3%E5%88%86%E5%8C%BA)、パーティションフィールドは大文字と小文字を区別しません。[#28309](https://github.com/StarRocks/starrocks/pull/28309)

### 問題修正

以下の問題を修正しました：

- FEでのテーブルレベルのスキャン統計情報が誤っており、テーブルのクエリとインポートのメトリクス情報が正しくなかった。[#27779](https://github.com/StarRocks/starrocks/pull/27779)
- パーティションテーブルでsort key列を変更した後、クエリ結果が不安定になる問題。[#27850](https://github.com/StarRocks/starrocks/pull/27850)
- Restore後に同じtabletがBEとFEでversionが一致しない問題。[#26518](https://github.com/StarRocks/starrocks/pull/26518/files)
- Colocationテーブルを作成する際にbucketsの数を指定しないと、bucketsの数が自動的に0と推定され、後でパーティションを追加すると失敗する。[#27086](https://github.com/StarRocks/starrocks/pull/27086)
- INSERT INTO SELECTのSELECT結果セットが空の場合、SHOW LOADがインポートタスクのステータスを `CANCELED` と表示する。[#26913](https://github.com/StarRocks/starrocks/pull/26913)
- sub_bitmap関数の入力値がBITMAP型でない場合、BEがクラッシュする。[#27982](https://github.com/StarRocks/starrocks/pull/27982)
- AUTO_INCREMENT列を更新するとBEがクラッシュする。[#27199](https://github.com/StarRocks/starrocks/pull/27199)
- 物化ビューのOuter joinとAnti joinの書き換えが誤っていた。[#28028](https://github.com/StarRocks/starrocks/pull/28028)
- 主キーモデルで一部の列を更新すると、平均row sizeの推定が不正確でメモリ使用量が過剰になる。[#27485](https://github.com/StarRocks/starrocks/pull/27485)
- 無効な物化ビューをアクティブ化するとFEがクラッシュする可能性がある。[#27959](https://github.com/StarRocks/starrocks/pull/27959)
- Hudi Catalogをベースに作成された外部テーブルの物化ビューにクエリを書き換えることができない。[#28023](https://github.com/StarRocks/starrocks/pull/28023)
- Hiveテーブルを削除した後、メタデータキャッシュを手動で更新しても、Hiveテーブルのデータをクエリできる。[#28223](https://github.com/StarRocks/starrocks/pull/28223)
- 非同期物化ビューのリフレッシュポリシーが手動で、リフレッシュタスクを同期的に呼び出す（SYNC MODE）場合、手動リフレッシュ後に`information_schema.task_runs`テーブルにINSERT OVERWRITEのレコードが複数存在する。[#28060](https://github.com/StarRocks/starrocks/pull/28060)
- LabelCleanerスレッドがハングアップし、FEのメモリリークを引き起こす。[#28311](https://github.com/StarRocks/starrocks/pull/28311) [#28636](https://github.com/StarRocks/starrocks/pull/28636)

## 3.0.4

リリース日：2023年7月18日

### 新機能

- クエリと物化ビューのJoinタイプが異なる場合でも、クエリの書き換えをサポートします。[#25099](https://github.com/StarRocks/starrocks/pull/25099)

### 機能最適化

- 非同期物化ビューの手動リフレッシュポリシーを最適化。REFRESH MATERIALIZED VIEW WITH SYNC MODEを使用して物化ビューのリフレッシュタスクを同期的に呼び出すことをサポートします。[#25904](https://github.com/StarRocks/starrocks/pull/25904)
- クエリのフィールドが物化ビューのoutput列に含まれていなくても、その述語条件に含まれていれば、その物化ビューを使用してクエリの書き換えが可能です。[#23028](https://github.com/StarRocks/starrocks/issues/23028)
- [Trino構文に切り替え](../reference/System_variable.md) `set sql_dialect = 'trino';`すると、クエリ時のテーブルエイリアスは大文字と小文字を区別しません。[#26094](https://github.com/StarRocks/starrocks/pull/26094) [#25282](https://github.com/StarRocks/starrocks/pull/25282)
- `Information_schema.tables_config`テーブルに`table_id`フィールドを追加しました。`table_id`フィールドを使用して、データベース`Information_schema`の`tables_config`テーブルと`be_tablets`を関連付けて、tabletが属するデータベースとテーブル名をクエリできます。[#24061](https://github.com/StarRocks/starrocks/pull/24061)

### 問題修正

以下の問題を修正しました：

- sum集計関数のクエリが単一テーブルの物化ビューに書き換えられた際、型推論の問題によりsumのクエリ結果が誤っていた。[#25512](https://github.com/StarRocks/starrocks/pull/25512)
- ストレージ分離モードで、SHOW PROCを使用してtablet情報を表示する際にエラーが発生していました。
- データの長さがSTRUCTで定義されたCHARの長さを超えると、挿入が応答しなくなる。[#25942](https://github.com/StarRocks/starrocks/pull/25942)
- INSERT INTO SELECTにFULL JOINが存在する場合、返される結果に欠落がある。[#26603](https://github.com/StarRocks/starrocks/pull/26603)
- ALTER TABLEコマンドを使用してテーブルの`default.storage_medium`属性を変更する際に`ERROR xxx: Unknown table property xxx`というエラーが発生していました。[#25870](https://github.com/StarRocks/starrocks/issues/25870)
- Broker Loadで空のファイルをインポートするとエラーが発生する。[#26212](https://github.com/StarRocks/starrocks/pull/26212)
- BEのオフラインが時々ハングアップする。[#26509](https://github.com/StarRocks/starrocks/pull/26509)

## 3.0.3

リリース日：2023年6月28日

### 機能最適化

- StarRocksの外部テーブルのメタデータ同期は、データロード時に行われるようになりました。[#24739](https://github.com/StarRocks/starrocks/pull/24739)
- [式ベースのパーティション](../table_design/expression_partitioning.md)を使用するテーブルに対して、INSERT OVERWRITEが特定のパーティションを指定することをサポートします。[#25005](https://github.com/StarRocks/starrocks/pull/25005)
- 非パーティションテーブルにパーティションを追加する際のエラーメッセージを最適化しました。[#25266](https://github.com/StarRocks/starrocks/pull/25266)

### 問題修正

以下の問題を修正しました：

- Parquetファイルに複雑な型が含まれている場合、最大最小フィルタリングで列の取得が誤っていた。[#23976](https://github.com/StarRocks/starrocks/pull/23976)
- データベースやテーブルがDropされているにも関わらず、書き込みタスクがキューに残っている。[#24801](https://github.com/StarRocks/starrocks/pull/24801)
- FEの再起動時にBEがクラッシュする小さな確率がある。[#25037](https://github.com/StarRocks/starrocks/pull/25037)
- "enable_profile = true"が設定されていると、インポートとクエリが時々ハングアップする。[#25060](https://github.com/StarRocks/starrocks/pull/25060)
- クラスターに3つのAlive BEがない場合、INSERT OVERWRITEのエラーメッセージが不正確である。[#25314](https://github.com/StarRocks/starrocks/pull/25314)

## 3.0.2

リリース日：2023年6月13日

### 機能最適化

- Unionクエリが物化ビューによって書き換えられた後、述語もプッシュダウンできるようになりました。[#23312](https://github.com/StarRocks/starrocks/pull/23312)
- テーブルの[自動バケット戦略](../table_design/Data_distribution.md#确定分桶数量)を最適化しました。[#24543](https://github.com/StarRocks/starrocks/pull/24543)
- NetworkTimeのシステムクロックへの依存を解消し、システムクロックの誤差がExchangeネットワークの時間推定に異常を引き起こす問題を解決しました。[#24858](https://github.com/StarRocks/starrocks/pull/24858)

### 問題修正

以下の問題を修正しました：

- Schema changeとデータインポートが同時に行われると、Schema changeが時々ハングアップする。[#23456](https://github.com/StarRocks/starrocks/pull/23456)
- `pipeline_profile_level = 0`の設定でクエリがエラーになる。[#23873](https://github.com/StarRocks/starrocks/pull/23873)
- `cloud_native_storage_type`の設定がS3の場合、テーブル作成時にエラーが発生する。
- LDAPアカウントがパスワードなしでログインできる。[#24862](https://github.com/StarRocks/starrocks/pull/24862)
- CANCEL LOADがテーブルが存在しない場合に失敗する。[#24922](https://github.com/StarRocks/starrocks/pull/24922)

### アップグレード注意事項

- システムに`starrocks`という名前のデータベースがある場合は、ALTER DATABASE RENAMEを使用して名前を変更した後にアップグレードしてください。

## 3.0.1

リリース日：2023年6月1日

### 新機能

- [パブリックテスト中] 大規模クエリのオペレータをディスクにスピルする([Spill to disk](../administration/spill_to_disk.md))機能をサポートし、中間結果をディスクに書き出すことで大規模クエリのメモリ消費を削減します。
- [Routine Load](../loading/RoutineLoad.md#导入-avro-数据)はAvro形式のデータのインポートをサポートします。
- [Microsoft Azure Storage](../integrations/authenticate_to_azure_storage.md)（Azure Blob StorageおよびAzure Data Lake Storageを含む）をサポートします。

### 機能最適化

- ストレージ分離クラスター(shared-data)は、StarRocksの外部テーブルを介して他のStarRocksクラスターのデータを同期することをサポートします。
- [Information Schema](../reference/information_schema/load_tracking_logs.md)に`load_tracking_logs`を追加し、最近のインポートエラー情報を記録します。
- テーブル作成ステートメント内の中国語のスペースを無視します。[#23885](https://github.com/StarRocks/starrocks/pull/23885)

### 問題修正

以下の問題を修正しました：

- SHOW CREATE TABLEが返す**主キーモデルテーブル**のテーブル作成情報が誤っていた。[#24237](https://github.com/StarRocks/starrocks/issues/24237)
- Routine Loadのプロセス中にBEがクラッシュする。[#20677](https://github.com/StarRocks/starrocks/issues/20677)
- サポートされていないPropertiesを指定してパーティションテーブルを作成するとNPEが発生する。[#21374](https://github.com/StarRocks/starrocks/issues/21374)
- SHOW TABLE STATUSの結果が不完全である。[#24279](https://github.com/StarRocks/starrocks/issues/24279)

### アップグレード注意事項

- システムに`starrocks`という名前のデータベースがある場合は、ALTER DATABASE RENAMEを使用して名前を変更した後にアップグレードしてください。

## 3.0.0

リリース日：2023年4月28日

### 新機能

**システムアーキテクチャ**


- 支持存算分离アーキテクチャ。FEの設定ファイルで有効化でき、有効にするとデータはリモートオブジェクトストレージ/HDFSに永続化され、ローカルディスクはキャッシュとして使用されます。これにより、ノードの追加や削除が柔軟に行え、テーブルレベルでのキャッシュライフサイクル管理がサポートされます。ローカルキャッシュがヒットした場合、非存算分離バージョンと同等のパフォーマンスを実現できます。詳細は[StarRocksの存算分離クラスターのデプロイと使用](https://docs.starrocks.io/zh/docs/3.0/deployment/deploy_shared_data/)を参照してください。

**ストレージとインポート**

- 自動増分列属性 [AUTO_INCREMENT](../sql-reference/sql-statements/auto_increment.md) をサポートし、テーブル内でグローバルにユニークなIDを提供し、データ管理を簡素化します。
- [インポート時に自動的にパーティションを作成し、パーティション式を使用してパーティションルールを定義する](https://docs.starrocks.io/zh-cn/3.0/table_design/automatic_partitioning)ことをサポートし、パーティション作成の使いやすさと柔軟性を向上させました。
- Primary Keyモデルのテーブルは、より豊富な [UPDATE](../sql-reference/sql-statements/data-manipulation/UPDATE.md) と [DELETE](../sql-reference/sql-statements/data-manipulation/DELETE.md) の構文をサポートし、CTEや複数テーブルへの参照を使用できます。
- Broker LoadとINSERT INTOにLoad Profileを追加し、profileを通じてインポートジョブの詳細を確認し分析することができます。使用方法は [Query Profileの確認と分析](../administration/query_profile.md) と同じです。

**データレイク分析**

- [プレビュー] Presto/Trino互換モードをサポートし、Presto/TrinoのSQLを自動的に書き換えることができます。詳細は[システム変数](../reference/System_variable.md) の `sql_dialect` を参照してください。
- [プレビュー] [JDBC Catalog](../data_source/catalog/jdbc_catalog.md) をサポートします。
- [SET CATALOG](../sql-reference/sql-statements/data-definition/SET_CATALOG.md) コマンドを使用して手動でCatalogを選択することをサポートします。

**権限とセキュリティ**

- 新しい完全なRBAC機能を提供し、ロールの継承とデフォルトロールをサポートします。詳細は[権限システムの概要](../administration/privilege_overview.md)を参照してください。
- より多くの権限管理オブジェクトとより細かい権限を提供します。詳細は[権限項目](../administration/privilege_item.md)を参照してください。

**クエリ**

<!-- - [プレビュー] 大規模クエリのオペレーターをディスクに落とすことをサポートし、メモリ不足時にディスクスペースを利用してクエリの安定した実行を保証できます。
- [Query Cache](../using_starrocks/query_cache.md) は、Broadcast JoinやBucket Shuffle Joinなど、さまざまなシナリオでの使用をサポートします。-->
- [Global UDF](../sql-reference/sql-functions/JAVA_UDF.md) をサポートします。
- 動的に自動調整される並列度をサポートし、クエリの並行性に応じて `pipeline_dop` を調整できます。

**SQLステートメントと関数**

- 権限に関連する以下のSQLステートメントを新たに追加しました：[SET DEFAULT ROLE](../sql-reference/sql-statements/account-management/SET_DEFAULT_ROLE.md)、[SET ROLE](../sql-reference/sql-statements/account-management/SET_ROLE.md)、[SHOW ROLES](../sql-reference/sql-statements/account-management/SHOW_ROLES.md)、[SHOW USERS](../sql-reference/sql-statements/account-management/SHOW_USERS.md)。
- 半構造化データ分析に関連する関数を新たに追加しました：[map_from_arrays](../sql-reference/sql-functions/map-functions/map_from_arrays.md)、[map_apply](../sql-reference/sql-functions/map-functions/map_apply.md)。
- [array_agg](../sql-reference/sql-functions/array-functions/array_agg.md) は ORDER BY をサポートします。
- ウィンドウ関数 [lead](../sql-reference/sql-functions/Window_function.md#lead-ウィンドウ関数の使用) と [lag](../sql-reference/sql-functions/Window_function.md#lag-ウィンドウ関数の使用) は IGNORE NULLS をサポートします。
- [BINARY/VARBINARY データ型](../sql-reference/sql-statements/data-types/BINARY.md)を新たに追加し、[to_binary](../sql-reference/sql-functions/binary-functions/to_binary.md)、[from_binary](../sql-reference/sql-functions/binary-functions/from_binary.md) 関数を新たに追加しました。
- 文字列関数 [replace](../sql-reference/sql-functions/string-functions/replace.md)、[hex_decode_binary](../sql-reference/sql-functions/string-functions/hex_decode_binary.md)、[hex_decode_string](../sql-reference/sql-functions/string-functions/hex_decode_string.md) を新たに追加しました。
- 暗号化関数 [base64_decode_binary](../sql-reference/sql-functions/crytographic-functions/base64_decode_binary.md)、[base64_decode_string](../sql-reference/sql-functions/crytographic-functions/base64_decode_string.md) を新たに追加しました。
- 数学関数 [sinh](../sql-reference/sql-functions/math-functions/sinh.md)、[cosh](../sql-reference/sql-functions/math-functions/cosh.md)、[tanh](../sql-reference/sql-functions/math-functions/tanh.md) を新たに追加しました。
- ユーティリティ関数 [current_role](../sql-reference/sql-functions/utility-functions/current_role.md) を新たに追加しました。

### 機能の最適化

**デプロイ**

- 3.0バージョンのDockerイメージと関連する[デプロイドキュメント](../quick_start/deploy_with_docker.md)を更新しました。[#20623](https://github.com/StarRocks/starrocks/pull/20623) [#21021](https://github.com/StarRocks/starrocks/pull/21021)

**ストレージとインポート**

- データインポートには、`skip_header`、`trim_space`、`enclose`、`escape`など、より豊富なCSVフォーマットパラメータが提供されます。詳細は [STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)、[ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md) を参照してください。
- [Primary Keyモデルのテーブル](../table_design/table_types/primary_key_table.md)は、主キーとソートキーを分離し、`ORDER BY`を使用してソートキーを個別に指定できるようになりました。
- 大規模データインポート、部分的な列の更新、永続化インデックスの有効化などのシナリオで、Primary Keyモデルのテーブルのメモリ使用量を最適化しました。[#12068](https://github.com/StarRocks/starrocks/pull/12068) [#14187](https://github.com/StarRocks/starrocks/pull/14187) [#15729](https://github.com/StarRocks/starrocks/pull/15729)
- 非同期ETLコマンドインターフェースを提供し、非同期INSERTタスクを作成できます。詳細は[INSERT](../loading/InsertInto.md) および [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) を参照してください。([#20609](https://github.com/StarRocks/starrocks/issues/20609))

**マテリアライズドビュー**

- [マテリアライズドビュー](../using_starrocks/Materialized_view.md)の改写能力を最適化しました：
  - view delta joinをサポートし、改写可能です。
  - Outer JoinとCross Joinの改写をサポートします。
  - パーティションがある場合のUNIONのSQLリライトを最適化しました。
- マテリアライズドビューの構築能力を完善し、CTE、SELECT *、UNIONをサポートします。
- [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) コマンドの返り値情報を最適化しました。
- マテリアライズドビューの構築時のパーティション作成効率を向上させました。([#21167](https://github.com/StarRocks/starrocks/pull/21167))

**クエリ**

- オペレーターがPipelineを完全にサポートするようになりました。
- [大規模クエリの管理](../administration/monitor_manage_big_queries.md)を完善しました。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md) はCPUとメモリ情報を表示でき、大規模クエリログを追加しました。
- Outer Join Reorderの能力を最適化しました。
- SQL解析フェーズのエラーメッセージを最適化し、クエリのエラー位置がより明確で情報がより明瞭になりました。

**データレイク分析**

- メタデータの統計情報収集を最適化しました。
- Hive Catalog、Iceberg Catalog、Hudi Catalog、Delta Lake Catalogは [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) を使用して外部テーブルの作成情報を表示できるようになりました。

### 問題修正

以下の問題を修正しました：

- StarRocksのソースファイルのライセンスヘッダーにある一部のURLがアクセスできない問題。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- SELECTクエリでunknown errorが発生する問題。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- SHOW/SET CHARACTERをサポートするようになりました。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- インポートする内容がStarRocksのフィールド長を超えた場合、ErrorURLのプロンプト情報が不正確である問題。ソースフィールドが空であると表示されるのではなく、内容が長すぎると表示されるべきです。[#14](https://github.com/StarRocks/DataX/issues/14)
- show full fields from 'table'をサポートするようになりました。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- パーティションのプルーニングによりMVの改写が失敗する問題。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- MVの作成ステートメントにcount(distinct)が含まれ、count(distinct)が分布列に適用される場合、MVの改写が失敗する問題。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- VARCHARをマテリアライズドビューのパーティション列として使用するとFEが正常に起動できない問題。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- ウィンドウ関数 [lead](../sql-reference/sql-functions/Window_function.md#lead-ウィンドウ関数の使用) と [lag](../sql-reference/sql-functions/Window_function.md#lag-ウィンドウ関数の使用) がIGNORE NULLSを正しく処理していない問題。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 一時パーティションの挿入と自動パーティションの作成が衝突する問題。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 行動変更

- RBACのアップグレード後、以前のユーザーと権限との互換性が保たれますが、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md) および [REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md) などの関連する構文に大幅な変更があります。
- SHOW MATERIALIZED VIEWが [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) に名前変更されました。
- 以下の[予約済みキーワード](../sql-reference/sql-statements/keywords.md)を新たに追加しました：AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### アップグレードに関する注意事項

**2.5バージョンから3.0へのアップグレードが必要です。そうでなければロールバックできません（理論上は2.5以下のバージョンから3.0へのアップグレードもサポートされていますが、リスクを低減するために、2.5からのアップグレードを統一してください。）**

#### BDBJEのアップグレード

3.0バージョンではBDBライブラリがアップグレードされましたが、BDBはロールバックをサポートしていないため、ロールバック後も3.0のBDBライブラリを使用する必要があります。

1. FEパッケージを古いバージョンに置き換えた後、3.0パッケージ内の`fe/lib/starrocks-bdb-je-18.3.13.jar`を2.5バージョンの`fe/lib`ディレクトリに移動します。
2. `fe/lib/je-7.*.jar`を削除します。

#### 権限システムのアップグレード

3.0 へのアップグレードは、デフォルトで新しい RBAC 権限管理システムを採用します。アップグレード後は、2.5 へのロールバックのみをサポートします。

ロールバック後には、手動で [ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md) コマンドを実行し、新しい image の作成をトリガーし、その image がすべての follower ノードに同期されるのを待つ必要があります。このコマンドを実行しない場合、ロールバックが部分的に失敗する可能性があります。ALTER SYSTEM CREATE IMAGE は、バージョン 2.5.3 以降でサポートされています。
