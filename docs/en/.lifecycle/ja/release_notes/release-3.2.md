---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 3.2

## v3.2.0-RC01

リリース日: 2023年11月15日

### 新機能

#### 共有データクラスタ

- ローカルディスク上の[主キーテーブルの永続インデックス](../table_design/table_types/primary_key_table.md)をサポートします。
- 複数のローカルディスク間でデータキャッシュを均等に分散することをサポートします。

#### データレイクアナリティクス

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)でデータベースと管理テーブルの作成と削除をサポートし、INSERTまたはINSERT OVERWRITEを使用してデータをHiveの管理テーブルにエクスポートすることをサポートします。
- [統一カタログ](../data_source/catalog/unified_catalog.md)をサポートし、HiveメタストアやAWS Glueなどの共通のメタストアを共有するHive、Iceberg、Hudi、Delta Lakeなどの異なるテーブル形式にアクセスできるようにします。

#### ストレージエンジン、データ取り込み、およびエクスポート

- テーブル関数[FILES()](../sql-reference/sql-functions/table-functions/files.md)を使用した以下の機能を追加しました：
  - AzureまたはGCPからParquetおよびORC形式のデータをロードする機能。
  - パラメータ`columns_from_path`を使用して、ファイルパスからキー/値ペアの値をカラムの値として抽出する機能。
  - ARRAY、JSON、MAP、STRUCTを含む複雑なデータ型のロードをサポートします。
- グローバル辞書の構築中にロードプロセスを大幅に容易にするdict_mapping列プロパティをサポートし、正確なCOUNT DISTINCTの計算を高速化します。
- INSERT INTO FILESを使用してStarRocksからAWS S3またはHDFSに格納されたParquet形式のファイルにデータをアンロードする機能をサポートします。詳細な手順については、[INSERT INTO FILESを使用したデータのアンロード](../unloading/unload_using_insert_into_files.md)を参照してください。

#### SQLリファレンス

以下の関数を追加しました：

- 文字列関数：substring_index、url_extract_parameter、url_encode、url_decode、translate
- 日付関数：dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date、to_tera_timestamp
- パターンマッチング関数：regexp_extract_all
- ハッシュ関数：xx_hash3_64
- 集約関数：approx_top_k
- ウィンドウ関数：cume_dist、percent_rank、session_number
- ユーティリティ関数：dict_mapping、get_query_profile

#### 権限とセキュリティ

StarRocksはApache Rangerを介したアクセス制御をサポートし、より高いレベルのデータセキュリティを提供し、外部データソースの既存のRangerサービスの再利用を可能にします。Apache Rangerと統合した後、StarRocksは次のアクセス制御方法を有効にします：

- StarRocksの内部テーブル、外部テーブル、またはその他のオブジェクトにアクセスする場合、RangerでStarRocksサービスに構成されたアクセスポリシーに基づいてアクセス制御を強制することができます。
- 外部カタログにアクセスする場合、アクセス制御は元のデータソース（Hiveサービスなど）の対応するRangerサービスを利用して制御することもできます（現在、Hiveへのデータエクスポートのアクセス制御はまだサポートされていません）。

詳細については、[Apache Rangerでの権限管理](../administration/ranger_plugin.md)を参照してください。

### 改善点

#### マテリアライズドビュー

非同期マテリアライズドビュー

- 作成：

  ビューやマテリアライズドビューが変更された場合、非同期マテリアライズドビューに自動的なリフレッシュをサポートします。

- 可視性：

  非同期マテリアライズドビューのクエリダンプをサポートします。

- 非同期マテリアライズドビューのリフレッシュタスクのスピル先をディスクに変更し、メモリ使用量を削減します。
- データの整合性：

  - 非同期マテリアライズドビューの作成に対して`query_rewrite_consistency`プロパティを追加しました。このプロパティは整合性チェックに基づいてクエリの書き換えルールを定義します。
  - 外部カタログベースの非同期マテリアライズドビューの作成に対して`force_external_table_query_rewrite`プロパティを追加しました。このプロパティは、外部カタログに基づいて非同期マテリアライズドビューの強制的なクエリ書き換えを許可するかどうかを定義します。

  詳細な情報については、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

- マテリアライズドビューのパーティショニングキーに整合性チェックを追加しました。

  ユーザーがウィンドウ関数を含む非同期マテリアライズドビューを作成する場合、ウィンドウ関数のパーティショニング列はマテリアライズドビューのパーティショニング列と一致する必要があります。

#### ストレージエンジン、データ取り込み、およびエクスポート

- プライマリキーテーブルの永続インデックスのパフォーマンスを向上させ、I/Oの読み取りと書き込みの増幅を削減するために、メモリ使用量のロジックを改善しました。[#24875](https://github.com/StarRocks/starrocks/pull/24875)  [#27577](https://github.com/StarRocks/starrocks/pull/27577)  [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- プライマリキーテーブルのデータをローカルディスク間で再分配する機能をサポートします。
- パーティションテーブルは、パーティションの時間範囲とクールダウン時間に基づいて自動的にクールダウンします。詳細については、[初期ストレージメディアと自動ストレージクールダウン時間の設定](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)を参照してください。
- プライマリキーテーブルにデータを書き込むロードジョブのパブリッシュフェーズを非同期モードから同期モードに変更しました。そのため、ロードジョブが完了した後すぐにデータをクエリできます。詳細については、[enable_sync_publish](../administration/Configuration.md#enable_sync_publish)を参照してください。

#### クエリ

- StarRocksのMetabaseおよびSupersetとの互換性を最適化しました。外部カタログとの統合をサポートします。

#### SQLリファレンス

- array_aggがキーワードDISTINCTをサポートします。

### 開発者ツール

- 非同期マテリアライズドビューのトレースクエリプロファイルをサポートし、透過的なリライトを分析するために使用できます。

### 互換性の変更

#### 動作の変更

更新予定です。

#### パラメータ

- データキャッシュの新しいパラメータを追加しました。

#### システム変数

更新予定です。

### バグ修正

以下の問題が修正されました：

- libcurlが呼び出されたときにBEがクラッシュする問題を修正しました。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- スキーマ変更が長時間にわたる場合、指定されたタブレットバージョンがガベージコレクションで処理されるため、スキーマ変更が失敗する場合があります。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- MinIOまたはAWS S3のParquetファイルにファイル外部テーブルを介してアクセスできない問題を修正しました。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- `information_schema.columns`でARRAY、MAP、STRUCT型の列が正しく表示されない問題を修正しました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- `information_schema.columns`ビューでBINARYまたはVARBINARYデータ型の`DATA_TYPE`および`COLUMN_TYPE`が`unknown`と表示される問題を修正しました。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### アップグレードノート

更新予定です。
