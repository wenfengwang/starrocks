---
displayed_sidebar: Chinese
---

# StarRocks バージョン 3.2

## v3.2.0-RC01

リリース日：2023年11月15日

### 新機能

#### ストレージと計算の分離アーキテクチャ

- [プライマリキーモデルテーブル](../table_design/table_types/primary_key_table.md)のインデックスをローカルディスクに永続化することをサポート。
- Data Cacheが複数のディスク間で均等に分散されるようにサポート。

#### データレイク分析

- [Hive Catalog](../data_source/catalog/hive_catalog.md)でデータベースやManaged Tableの作成、削除をサポートし、INSERTまたはINSERT OVERWRITEを使用してHiveのManaged Tableにデータをエクスポートすることをサポート。
- [Unified Catalog](../data_source/catalog/unified_catalog.md)をサポート。同一のHive MetastoreまたはAWS Glueメタデータサービスが複数のテーブルフォーマット（Hive、Iceberg、Hudi、Delta Lakeなど）を含む場合、Unified Catalogを通じて一元的にアクセスできる。

#### インポート、エクスポート、ストレージ

- テーブル関数[FILES()](../sql-reference/sql-functions/table-functions/files.md)を使用したデータインポートに以下の機能を追加：
  - AzureおよびGCPにあるParquetまたはORC形式のファイルからデータをインポートすることをサポート。
  - `columns_from_path`パラメータをサポートし、ファイルパスからフィールド情報を抽出できる。
  - 複雑な型（JSON、ARRAY、MAP、STRUCT）のデータのインポートをサポート。
- dict_mapping列属性をサポートし、グローバル辞書のデータインポートプロセスを大幅に簡素化し、正確な重複排除などの計算を加速する。
- INSERT INTO FILES()ステートメントを使用して、AWS S3またはHDFSのParquet形式のファイルにデータをエクスポートすることをサポート。詳細については、[INSERT INTO FILESを使用したデータのエクスポート](../unloading/unload_using_insert_into_files.md)を参照してください。

#### SQLステートメントと関数

以下の関数を新たに追加：

- 文字列関数：substring_index、url_extract_parameter、url_encode、url_decode、translate
- 日付関数：dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date、to_tera_timestamp
- 曖昧/正規表現マッチング関数：regexp_extract_all
- ハッシュ関数：xx_hash3_64
- 集約関数：approx_top_k
- ウィンドウ関数：cume_dist、percent_rank、session_number
- ユーティリティ関数：dict_mapping、get_query_profile

#### 権限

Apache Rangerを通じてアクセス制御を実現し、より高いレベルのデータセキュリティを提供し、外部データソースに対応するRanger Serviceを再利用することをサポート。StarRocksがApache Rangerを統合すると、以下のアクセス制御方法を実現できる：

- StarRocks内のテーブル、外部テーブル、その他のオブジェクトにアクセスする際、Rangerに作成されたStarRocks Serviceのアクセスポリシーに基づいて制御を行う。
- External Catalogにアクセスする際も、対応するデータソースの既存のRanger service（例：Hive Service）を再利用して制御を行う（Hiveへのデータエクスポートに関しては、現在対応するアクセス制御ポリシーは提供されていない）。

詳細は、[Apache Rangerを使用した権限管理](../administration/ranger_plugin.md)を参照してください。

### 機能改善

#### マテリアライズドビュー

非同期マテリアライズドビュー

- マテリアライズドビューのリフレッシュ：

  自動リフレッシュ：マテリアライズドビューに関連するテーブル、ビュー、およびビュー内のテーブル、マテリアライズドビューがSchema ChangeまたはSwap操作を行った後、自動的にリフレッシュされる。

- マテリアライズドビューの可観測性：

  マテリアライズドビューはQuery Dumpをサポート。

- マテリアライズドビューのリフレッシュは、デフォルトで中間結果をディスクに書き込むようになり、リフレッシュ時のメモリ消費を削減。
- データ一貫性：

  - マテリアライズドビューを作成する際に、`query_rewrite_consistency`属性を追加。この属性を使用して、一貫性チェックの結果に基づいてクエリの書き換えルールを定義できる。
  - マテリアライズドビューを作成する際に、`force_external_table_query_rewrite`属性を追加。この属性は、外部テーブルのマテリアライズドビューに対してクエリ書き換えを強制的に有効にするかどうかを定義する。

  詳細は、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

- パーティション列の一貫性チェックを追加：パーティションマテリアライズドビューを作成する際、マテリアライズドビューのクエリにパーティション付きのウィンドウ関数が含まれている場合、ウィンドウ関数のパーティション列はマテリアライズドビューのパーティション列と一致する必要がある。

#### インポート、エクスポート、ストレージ

- プライマリキーモデルテーブルの永続化インデックス機能を最適化し、メモリ使用のロジックを改善し、I/Oの読み書きの増幅を低減。
- プライマリキーモデルテーブルは、ローカルの複数のディスク間でデータが均等に分散されるようにサポート。
- パーティション内のデータは時間の経過とともに自動的にコールドダウンされる（Listパーティション方式は現在サポートされていない）。従来の設定に比べて、パーティションのホット/コールド管理がより便利になる。詳細は、[データの初期ストレージメディア、自動コールドダウン時間の設定](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#データの初期ストレージメディア自動コールドダウン時間とレプリカ数の設定)を参照してください。
- プライマリキーモデルテーブルのデータ書き込み時のPublishプロセスが非同期から同期に変更され、インポートジョブが成功した後にデータが即時に表示されるようになった。詳細は、[enable_sync_publish](../administration/FE_configuration.md#enable_sync_publish)を参照してください。

#### クエリ

- MetabaseおよびSupersetとの互換性が向上し、External Catalogの統合をサポート。

#### SQLステートメントと関数

- array_aggがDISTINCTキーワードの使用をサポート。

### 開発者ツール

- 非同期マテリアライズドビューがTrace Query Profileをサポートし、マテリアライズドビューの透明な書き換えシナリオを分析できる。

### 互換性の変更

#### 挙動の変更

詳細は追加される予定です。

#### 設定パラメータ

- Data Cache関連のパラメータを新たに追加。

#### システム変数

詳細は追加される予定です。

### 問題の修正

以下の問題を修正しました：

- libcurlの呼び出しによりBE Crashが発生する問題。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- Schema Changeの実行時間が長すぎると、Tabletのバージョンがガベージコレクションによって削除され、失敗する問題。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- ファイル外部テーブルを通じてMinIO上に保存されているParquetファイルを読み取ることができない問題。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- `information_schema.columns`ビューでARRAY、MAP、STRUCT型のフィールドが正しく表示されない問題。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- BINARYまたはVARBINARY型が`information_schema.columns`ビューの`DATA_TYPE`および`COLUMN_TYPE`に`unknown`として表示される問題。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### アップグレードに関する注意事項

詳細は追加される予定です。
