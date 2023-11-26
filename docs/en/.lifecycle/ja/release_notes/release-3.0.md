# StarRocksバージョン3.0

## 3.0.8

リリース日: 2023年11月17日

## 改善点

- システムデータベース`INFORMATION_SCHEMA`の`COLUMNS`ビューでは、ARRAY、MAP、およびSTRUCTの列を表示できるようになりました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)

## バグ修正

以下の問題が修正されました:

- `show proc '/current_queries';`が実行されている間にクエリが開始されると、BEがクラッシュする場合があります。[#34316](https://github.com/StarRocks/starrocks/pull/34316)
- ソートキーが指定されたプライマリキーテーブルに連続してデータが高頻度でロードされると、コンパクションの失敗が発生する場合があります。[#26486](https://github.com/StarRocks/starrocks/pull/26486)
- ブローカーロードジョブでフィルタリング条件が指定されている場合、データロード中にBEがクラッシュする場合があります。[#29832](https://github.com/StarRocks/starrocks/pull/29832)
- SHOW GRANTSが実行されると不明なエラーが報告されます。[#30100](https://github.com/StarRocks/starrocks/pull/30100)
- `cast()`関数で指定されたターゲットデータ型が元のデータ型と同じ場合、特定のデータ型の場合にBEがクラッシュする場合があります。[#31465](https://github.com/StarRocks/starrocks/pull/31465)
- BINARYまたはVARBINARYデータ型の`DATA_TYPE`および`COLUMN_TYPE`は、`information_schema.columns`ビューで`unknown`として表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)
- 永続インデックスが有効なプライマリキーテーブルに長時間、頻繁にデータをロードすると、BEがクラッシュする場合があります。[#33220](https://github.com/StarRocks/starrocks/pull/33220)
- クエリキャッシュが有効な場合、クエリ結果が正しくありません。[#32778](https://github.com/StarRocks/starrocks/pull/32778)
- クラスタが再起動された後、復元されたテーブルのデータがバックアップ前のデータと一致しない場合があります。[#33567](https://github.com/StarRocks/starrocks/pull/33567)
- RESTOREが実行され、同時にCompactionが行われると、BEがクラッシュする場合があります。[#32902](https://github.com/StarRocks/starrocks/pull/32902)

## 3.0.7

リリース日: 2023年10月18日

### 改善点

- Window関数COVAR_SAMP、COVAR_POP、CORR、VARIANCE、VAR_SAMP、STD、およびSTDDEV_SAMPは、ORDER BY句とWindow句をサポートするようになりました。[#30786](https://github.com/StarRocks/starrocks/pull/30786)
- プライマリキーテーブルにデータを書き込むロードジョブのPublishフェーズは、非同期モードから同期モードに変更されました。そのため、ロードジョブが終了した後すぐにロードされたデータをクエリできるようになりました。[#27055](https://github.com/StarRocks/starrocks/pull/27055)
- DECIMAL型のデータのクエリ中にDECIMALオーバーフローが発生した場合、NULLではなくエラーが返されます。[#30419](https://github.com/StarRocks/starrocks/pull/30419)
- 無効なコメントを含むSQLコマンドを実行すると、MySQLと一貫した結果が返されるようになりました。[#30210](https://github.com/StarRocks/starrocks/pull/30210)
- 単一のパーティショニング列または式パーティショニングを使用するStarRocksテーブルでは、パーティション列式を含むSQL述語もパーティションプルーニングに使用できるようになりました。[#30421](https://github.com/StarRocks/starrocks/pull/30421)

### バグ修正

以下の問題が修正されました:

- データベースとテーブルの作成と削除を同時に行うと、テーブルが見つからず、そのテーブルへのデータロードの失敗につながる場合があります。[#28985](https://github.com/StarRocks/starrocks/pull/28985)
- 特定のケースでUDFの使用によりメモリリークが発生する場合があります。[#29467](https://github.com/StarRocks/starrocks/pull/29467) [#29465](https://github.com/StarRocks/starrocks/pull/29465)
- ORDER BY句に集計関数が含まれている場合、エラー「java.lang.IllegalStateException: null」が返されます。[#30108](https://github.com/StarRocks/starrocks/pull/30108)
- ユーザーがHiveカタログを使用してTencent COSに格納されたデータをクエリする場合、クエリ結果が正しくありません。[#30363](https://github.com/StarRocks/starrocks/pull/30363)
- ARRAY<STRUCT>型データのSTRUCTの一部のサブフィールドが欠落している場合、クエリ中に欠落したサブフィールドにデフォルト値が埋め込まれる際にデータの長さが正しくなりません。これにより、BEがクラッシュします。
- Berkeley DB Java Editionのバージョンがアップグレードされ、セキュリティの脆弱性を回避します。[#30029](https://github.com/StarRocks/starrocks/pull/30029)
- プライマリキーテーブルにデータをロードし、同時にTRUNCATE操作とクエリが実行される場合、特定のケースでエラー「java.lang.NullPointerException」がスローされます。[#30573](https://github.com/StarRocks/starrocks/pull/30573)
- スキーマ変更の実行時間が長すぎる場合、指定したバージョンのタブレットがガベージコレクションされるため、スキーマ変更が失敗する場合があります。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- CloudCanalを使用して`NOT NULL`に設定されたテーブル列にデータをロードする場合、エラー「Unsupported dataFormat value is : \N」がスローされます。[#30799](https://github.com/StarRocks/starrocks/pull/30799)
- StarRocks共有データクラスタでは、`information_schema.COLUMNS`にテーブルキーの情報が記録されないため、Flink Connectorを使用してデータをロードする場合にDELETE操作を実行できません。[#31458](https://github.com/StarRocks/starrocks/pull/31458)
- アップグレード中に特定の列の型もアップグレードされる場合（例: Decimal型からDecimal v3型に変更される場合）、特定の特性を持つテーブルのコンパクションがBEのクラッシュを引き起こす場合があります。[#31626](https://github.com/StarRocks/starrocks/pull/31626)
- Flink Connectorを使用してデータをロードする場合、高並行のロードジョブが存在し、HTTPスレッドの数とスキャンスレッドの数が上限に達した場合、ロードジョブが予期せず中断されます。[#32251](https://github.com/StarRocks/starrocks/pull/32251)
- libcurlが呼び出されるとBEがクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- BITMAP型の列がプライマリキーテーブルに追加されるとエラーが発生します。[#31763](https://github.com/StarRocks/starrocks/pull/31763)

## 3.0.6

リリース日: 2023年9月12日

### 動作変更

- [group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)関数を使用する場合、セパレータを宣言するためにSEPARATORキーワードを使用する必要があります。

### 新機能

- 集約関数[group_concat](../sql-reference/sql-functions/string-functions/group_concat.md)は、DISTINCTキーワードとORDER BY句をサポートします。[#28778](https://github.com/StarRocks/starrocks/pull/28778)
- パーティション内のデータは時間の経過とともに自動的に冷却されるようになりました。（この機能は[list partitioning](../table_design/list_partitioning.md)ではサポートされていません。）[#29335](https://github.com/StarRocks/starrocks/pull/29335) [#29393](https://github.com/StarRocks/starrocks/pull/29393)

### 改善点

- すべての複合述語とWHERE句のすべての式に対して暗黙の型変換をサポートするようになりました。[セッション変数](../reference/System_variable.md)`enable_strict_type`を使用して、暗黙の型変換を有効または無効にすることができます。このセッション変数のデフォルト値は`false`です。[#21870](https://github.com/StarRocks/starrocks/pull/21870)
- 文字列を整数に変換する際のFEとBEのロジックを統一しました。[#29969](https://github.com/StarRocks/starrocks/pull/29969)

### バグ修正

以下の問題が修正されました:

- `enable_orc_late_materialization`が`true`に設定されている場合、Hiveカタログを使用してORCファイル内のSTRUCT型データをクエリすると予期しない結果が返されます。[#27971](https://github.com/StarRocks/starrocks/pull/27971)
- Hiveカタログを介してデータクエリを実行する際、WHERE句でパーティショニング列とOR演算子が指定されている場合、クエリ結果が正しくありません。[#28876](https://github.com/StarRocks/starrocks/pull/28876)
- クラウドネイティブテーブルのRESTful APIアクション`show_data`が返す値が正しくありません。[#29473](https://github.com/StarRocks/starrocks/pull/29473)
- [共有データクラスタ](../deployment/shared_data/azure.md)がAzure Blob Storageにデータを格納し、テーブルが作成された場合、クラスタがバージョン3.0にロールバックされた後、FEの起動に失敗します。[#29433](https://github.com/StarRocks/starrocks/pull/29433)
- Icebergカタログのテーブルをクエリする場合、ユーザーにはそのテーブルに対して権限が付与されていても権限がありません。[#29173](https://github.com/StarRocks/starrocks/pull/29173)
- [SHOW FULL COLUMNS](../sql-reference/sql-statements/Administration/SHOW_FULL_COLUMNS.md)ステートメントによって返される[BITMAP](../sql-reference/sql-statements/data-types/BITMAP.md)または[HLL](../sql-reference/sql-statements/data-types/HLL.md)データ型の列の`Default`フィールドの値が正しくありません。[#29510](https://github.com/StarRocks/starrocks/pull/29510)
- `ADMIN SET FRONTEND CONFIG`コマンドを使用してFEの動的パラメータ`max_broker_load_job_concurrency`を変更しても効果がありません。
- [マテリアライズドビュー](../using_starrocks/Materialized_view.md)の書き換え機能を最適化しました。以下をサポートしています：
  - View Delta Join、Outer Join、および Cross Join の書き換えをサポートしました。
  - パーティションを持つ Union のSQLの書き換えを最適化しました。
- マテリアライズドビューのビルド機能を改善しました：CTE、select *、および Union をサポートしています。
- [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)で返される情報を最適化しました。
- マテリアライズドビューのビルド時のパーティション追加の効率を向上させるため、バッチでMVパーティションの追加をサポートしました。[#21167](https://github.com/StarRocks/starrocks/pull/21167)

#### クエリエンジン

- パイプラインエンジンで全てのオペレータをサポートしています。非パイプラインコードは後のバージョンで削除されます。
- [ビッグクエリの位置づけ](../administration/monitor_manage_big_queries.md)を改善し、ビッグクエリログを追加しました。[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)ではCPUとメモリの情報を表示することができます。
- Outer Join の最適化を行いました。
- SQLパース段階でのエラーメッセージを最適化し、より正確なエラー位置と明確なエラーメッセージを提供します。

#### データレイクアナリティクス

- メタデータの統計情報収集を最適化しました。
- 外部カタログで管理され、Apache Hive™、Apache Iceberg、Apache Hudi、またはDelta Lakeに保存されているテーブルの作成文を表示するために[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を使用することができるようになりました。

### バグ修正

- StarRocksのソースファイルのライセンスヘッダ内の一部のURLにアクセスできません。[#2224](https://github.com/StarRocks/starrocks/issues/2224)
- SELECTクエリ中に不明なエラーが返されます。[#19731](https://github.com/StarRocks/starrocks/issues/19731)
- SHOW/SET CHARACTER をサポートしました。[#17480](https://github.com/StarRocks/starrocks/issues/17480)
- StarRocksがサポートするフィールド長を超えるデータがロードされた場合、返されるエラーメッセージが正しくありません。[#14](https://github.com/StarRocks/DataX/issues/14)
- `show full fields from 'table'` をサポートしました。[#17233](https://github.com/StarRocks/starrocks/issues/17233)
- パーティションのプルーニングがMVの書き換えに失敗します。[#14641](https://github.com/StarRocks/starrocks/issues/14641)
- CREATE MATERIALIZED VIEW ステートメントに `count(distinct)` が含まれ、`count(distinct)` が DISTRIBUTED BY カラムに適用される場合、MVの書き換えに失敗します。[#16558](https://github.com/StarRocks/starrocks/issues/16558)
- VARCHARカラムがマテリアライズドビューのパーティショニングカラムとして使用される場合、FEが起動に失敗します。[#19366](https://github.com/StarRocks/starrocks/issues/19366)
- Window関数 [LEAD](../sql-reference/sql-functions/Window_function.md#lead) と [LAG](../sql-reference/sql-functions/Window_function.md#lag) が IGNORE NULLS を正しく処理しません。[#21001](https://github.com/StarRocks/starrocks/pull/21001)
- 一時的なパーティションの追加が自動パーティション作成と競合します。[#21222](https://github.com/StarRocks/starrocks/issues/21222)

### 動作の変更

- 新しいロールベースのアクセス制御（RBAC）システムは、以前の権限とロールをサポートしています。ただし、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)や[REVOKE](../sql-reference/sql-statements/account-management/REVOKE.md)などの関連するステートメントの構文が変更されています。
- [SHOW MATERIALIZED VIEW](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)に名前を変更しました。
- 以下の[予約語](../sql-reference/sql-statements/keywords.md)を追加しました：AUTO_INCREMENT、CURRENT_ROLE、DEFERRED、ENCLOSE、ESCAPE、IMMEDIATE、PRIVILEGES、SKIP_HEADER、TRIM_SPACE、VARBINARY。

### アップグレードの注意事項

v2.5からv3.0にアップグレードするか、v3.0からv2.5にダウングレードすることができます。

> 理論的には、v2.5より前のバージョンからのアップグレードもサポートされています。システムの可用性を確保するために、まずクラスタをv2.5にアップグレードし、その後v3.0にアップグレードすることをお勧めします。

v3.0からv2.5にダウングレードする際には、以下のポイントに注意してください。

#### BDBJE

StarRocksはv3.0でBDBライブラリをアップグレードしますが、BDBJEはロールバックできません。ダウングレード後はv3.0のBDBライブラリを使用する必要があります。以下の手順を実行してください：

1. v2.5のパッケージでFEパッケージを置き換えた後、v3.0の`fe/lib/starrocks-bdb-je-18.3.13.jar`をv2.5の`fe/lib`ディレクトリにコピーします。

2. `fe/lib/je-7.*.jar`を削除します。

#### 権限システム

新しいRBAC権限システムは、v3.0にアップグレード後にデフォルトで使用されます。v2.5にダウングレードすることしかできません。

ダウングレード後、[ALTER SYSTEM CREATE IMAGE](../sql-reference/sql-statements/Administration/ALTER_SYSTEM.md)を実行して新しいイメージを作成し、新しいイメージがすべてのフォロワーFEに同期されるのを待ちます。このコマンドは2.5.3以降でサポートされています。

v2.5とv3.0の権限システムの違いの詳細については、「Upgrade notes」を参照してください：[StarRocksでサポートされる権限](../administration/privilege_item.md)。
