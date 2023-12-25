---
displayed_sidebar: English
---
# StarRocks バージョン 3.2

## v3.2.0-RC01

リリース日: 2023年11月15日

### 新機能

#### 共有データクラスター

- ローカルディスク上の[プライマリキーテーブル用永続インデックス](../table_design/table_types/primary_key_table.md)に対応。
- 複数のローカルディスク間でデータキャッシュを均等に分散する機能に対応。

#### データレイクアナリティクス

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)でのデータベースとマネージドテーブルの作成・削除に対応し、INSERTまたはINSERT OVERWRITEを使用してHiveのマネージドテーブルへのデータエクスポートが可能。
- HiveメタストアやAWS Glueなどの共通メタストアを共有する異なるテーブルフォーマット（Hive、Iceberg、Hudi、Delta Lake）にアクセスできる[Unified Catalog](../data_source/catalog/unified_catalog.md)に対応。

#### ストレージエンジン、データ取り込み、およびエクスポート

- 表関数 [FILES()](../sql-reference/sql-functions/table-functions/files.md) を使用したロードに関する以下の機能を追加:
  - AzureまたはGCPからParquetおよびORC形式のデータをロード。
  - ファイルパスからキー/値ペアの値を列の値として抽出する`columns_from_path`パラメーター。
  - ARRAY、JSON、MAP、STRUCTなどの複雑なデータ型のロードに対応。
- dict_mapping列プロパティに対応し、グローバル辞書の構築中のロードプロセスを大幅に簡素化し、正確なCOUNT DISTINCT計算を加速。
- INSERT INTO FILESを使用してStarRocksからAWS S3またはHDFSに保存されているParquet形式のファイルへのデータアンロードに対応。詳細は[INSERT INTO FILESを使用したデータのアンロード](../unloading/unload_using_insert_into_files.md)を参照。

#### SQLリファレンス

以下の関数を追加:

- 文字列関数: substring_index、url_extract_parameter、url_encode、url_decode、translate
- 日付関数: dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date、to_tera_timestamp
- パターンマッチング関数: regexp_extract_all
- ハッシュ関数: xx_hash3_64
- 集約関数: approx_top_k
- ウィンドウ関数: cume_dist、percent_rank、session_number
- ユーティリティ関数: dict_mapping、get_query_profile

#### 権限とセキュリティ

StarRocksはApache Rangerを通じたアクセス制御をサポートし、より高いレベルのデータセキュリティを提供し、外部データソースの既存のRanger Serviceを再利用できます。Apache Rangerと統合した後、StarRocksは以下のアクセス制御方法を提供します:

- StarRocksの内部テーブル、外部テーブル、またはその他のオブジェクトにアクセスする際、RangerのStarRocksサービスで設定されたアクセスポリシーに基づいてアクセス制御が行われます。
- 外部カタログにアクセスする際も、元のデータソース（例えばHiveサービス）の対応するRangerサービスを利用してアクセスを制御することができます（ただし、現在Hiveへのデータエクスポートに関するアクセス制御はサポートされていません）。

詳細は[Apache Rangerを使用した権限管理](../administration/ranger_plugin.md)を参照してください。

### 改善点

#### マテリアライズドビュー

非同期マテリアライズドビュー:

- 作成:
  ビューやマテリアライズドビューに基づいて作成された非同期マテリアライズドビューの自動更新をサポートし、ビューやマテリアライズドビュー、またはそのベーステーブルでスキーマ変更が発生した際に自動的に更新されます。

- 可観測性:
  非同期マテリアライズドビューのクエリダンプに対応。

- ディスクへのスピル機能は、非同期マテリアライズドビューのリフレッシュタスクにデフォルトで有効化され、メモリ消費を削減します。
- データ整合性:
  - 非同期マテリアライズドビュー作成時に`query_rewrite_consistency`プロパティを追加。このプロパティは整合性チェックに基づくクエリ書き換えルールを定義します。
  - 外部カタログに基づく非同期マテリアライズドビュー作成時に`force_external_table_query_rewrite`プロパティを追加。このプロパティは外部カタログに基づいて作成された非同期マテリアライズドビューに対するクエリの強制書き換えを許可するかどうかを定義します。

  詳細は[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

- マテリアライズドビューのパーティションキーの整合性チェックを追加しました。

  ユーザーがPARTITION BY式を含むウィンドウ関数を使用して非同期マテリアライズドビューを作成する場合、ウィンドウ関数のパーティションカラムはマテリアライズドビューのパーティションカラムと一致する必要があります。

#### ストレージエンジン、データ取り込み、およびエクスポート

- 主キーテーブルの永続インデックスを最適化し、メモリ使用ロジックを改善しつつI/O読み書きの増幅を削減しました。[#24875](https://github.com/StarRocks/starrocks/pull/24875) [#27577](https://github.com/StarRocks/starrocks/pull/27577) [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- 主キーテーブルのローカルディスク間でのデータ再分配に対応。
- パーティションテーブルは、パーティションの時間範囲とクールダウン時間に基づいた自動クールダウンに対応。詳細は[初期ストレージ媒体と自動ストレージクールダウン時間の設定](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)を参照してください。
- 主キーテーブルにデータを書き込むロードジョブのパブリッシュフェーズが非同期モードから同期モードに変更され、ロードジョブ完了後すぐにデータをクエリできるようになりました。詳細は[enable_sync_publish](../administration/FE_configuration.md#enable_sync_publish)を参照してください。

#### クエリ

- MetabaseおよびSupersetとの互換性を最適化し、外部カタログとの統合をサポート。

#### SQLリファレンス

- array_aggがDISTINCTキーワードに対応。

### 開発者ツール

- 非同期マテリアライズドビューのクエリプロファイルトレースに対応し、透過的な書き換えを分析できます。

### 互換性の変更

#### 動作変更

更新予定。

#### パラメータ

- データキャッシュに関する新しいパラメータを追加。

#### システム変数

更新予定。

### バグ修正

以下の問題を修正しました:

- libcurlが呼び出された際にBEがクラッシュする問題。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- スキーマ変更が長時間にわたると、指定されたタブレットバージョンがガベージコレクションによって処理されるため失敗することがある問題。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- ファイル外部テーブルを介してMinIOまたはAWS S3のParquetファイルにアクセスできない問題。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- `information_schema.columns`ビューでARRAY、MAP、およびSTRUCT型のカラムが正しく表示されない問題。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- `information_schema.columns`ビューでBINARYまたはVARBINARYデータ型の`DATA_TYPE`および`COLUMN_TYPE`が`unknown`として表示される問題。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### アップグレードノート

更新予定。
