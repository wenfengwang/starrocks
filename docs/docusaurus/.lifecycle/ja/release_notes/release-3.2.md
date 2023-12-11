---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン3.2

## v3.2.0-RC01

リリース日: 2023年11月15日

### 新機能

#### 共有データクラスタ

- ローカルディスク上で[プライマリキーテーブルの永続インデックス](../table_design/table_types/primary_key_table.md)をサポートします。
- 複数のローカルディスク間でデータキャッシュを均等に分散することをサポートします。

#### データレイクアナリティクス

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)でデータベースと管理されたテーブルを作成および削除することをサポートし、INSERTまたはINSERT OVERWRITEを使用してHiveの管理テーブルにデータをエクスポートすることをサポートします。
- [統一カタログ](../data_source/catalog/unified_catalog.md) をサポートし、HiveメタストアやAWS Glueなどの共通のメタストアを共有するHive、Iceberg、Hudi、Delta Lakeなどの異なるテーブル形式にアクセスできるようにします。

#### ストレージエンジン、データ取り込み、エクスポート

- テーブル関数 [FILES()](../sql-reference/sql-functions/table-functions/files.md) に以下の機能を追加しました：
  - AzureまたはGCPからParquetおよびORC形式のデータをロードします。
  - ファイルパスからキー/値ペアの値を `columns_from_path` パラメータを使用して列の値として抽出します。
  - ARRAY、JSON、MAP、STRUCTを含む複雑なデータ型をロードします。
- グローバル辞書の構築時にロードプロセスを大幅に容易にする `dict_mapping` 列プロパティをサポートし、正確なCOUNT DISTINCT計算を高速化します。
- StarRocksからINSERT INTO FILESを使用してAWS S3またはHDFSに保存されたParquet形式のファイルにデータをアンロードします。詳細な手順については、[INSERT INTO FILESを使用したデータのアンロード](../unloading/unload_using_insert_into_files.md)を参照してください。

#### SQLリファレンス

以下の機能を追加しました：

- 文字列関数: substring_index、url_extract_parameter、url_encode、url_decode、およびtranslate
- 日付関数: dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date、およびto_tera_timestamp
- パターンマッチング関数: regexp_extract_all
- ハッシュ関数: xx_hash3_64
- 集約関数: approx_top_k
- ウィンドウ関数: cume_dist、percent_rank、およびsession_number
- ユーティリティ関数: dict_mapping およびget_query_profile

#### 権限とセキュリティ

StarRocksはApache Rangerを介したアクセス制御をサポートし、より高いレベルのデータセキュリティを提供し、既存の外部データソースのRangerサービスを再利用できます。Apache Rangerと統合した後、StarRocksでは次のアクセス制御方法が可能となります：

- StarRocksのアクセスポリシーによって、内部テーブル、外部テーブル、またはその他のオブジェクトへのアクセス時にRangerで構成されたアクセスポリシーに基づいてアクセス制御を強制できます。
- 外部カタログへのアクセス時にも、対応するRangerサービスを使用してアクセス制御を行います（現在は、Hiveへのデータエクスポートのアクセス制御はまだサポートされていません）。

詳細については、[Apache Rangerを使用して権限を管理する](../administration/ranger_plugin.md)を参照してください。

### 改善点

#### マテリアライズドビュー

非同期マテリアライズドビュー

- 作成：

  ビューやマテリアライズドビューが変更されたときに非同期マテリアライズドビューに対する自動リフレッシュをサポートします。

- 可観測性：

  非同期マテリアライズドビューのためのクエリダンプをサポートします。

- デフォルトで非同期マテリアライズドビューのリフレッシュタスクのスピルオーバーディスクが有効になり、メモリ消費量が減少します。
- データの整合性：

  - 非同期マテリアライズドビューの作成に対して `query_rewrite_consistency` プロパティが追加されました。このプロパティは整合性チェックに基づくクエリの書き換えルールを定義します。
  - 外部カタログベースの非同期マテリアライズドビューの作成に対して `force_external_table_query_rewrite` プロパティが追加されました。このプロパティは外部カタログに対する非同期マテリアライズドビューの強制クエリ書き換えを許可するかどうかを定義します。

  詳細については、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md) を参照してください。

- マテリアライズドビューのパーティショニングキーに整合性チェックが追加されました。

  ユーザーがパーティショニングキーを含むウィンドウ関数を持つ非同期マテリアライズドビューを作成するとき、ウィンドウ関数のパーティション列はマテリアライズドビューのそれと一致しなければなりません。

#### ストレージエンジン、データ取り込み、エクスポート

- プライマリキーテーブルの永続インデックスを最適化し、メモリ使用量ロジックを改善し、I/Oの読み書きを削減しました。[#24875](https://github.com/StarRocks/starrocks/pull/24875)  [#27577](https://github.com/StarRocks/starrocks/pull/27577)  [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- プライマリキーテーブルのデータをローカルディスク間で再分散することをサポートします。
- パーティションテーブルは、パーティションの時間範囲とクールダウン時間に基づいて自動的にクールダウンをサポートします。詳細については、[初期ストレージ媒体の指定と自動ストレージクールダウン時間の設定](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)を参照してください。
- プライマリキーテーブルにデータを書き込むロードジョブのPublishフェーズが非同期モードから同期モードに変更されました。そのため、ロードジョブが終了した後すぐにデータをクエリできるようになります。詳細については、[enable_sync_publish](../administration/Configuration.md#enable_sync_publish)を参照してください。

#### クエリ

- StarRocksとMetabase、Supersetの互換性を最適化し、外部カタログとの統合をサポートします。

#### SQLリファレンス

- array_agg は DISTINCT キーワードをサポートします。

### 開発者向けツール

- 非同期マテリアライズドビューのトレースクエリプロファイルをサポートし、その透過的な書き換えを分析するために使用できます。

### 互換性の変更

#### 動作の変更

更新予定。

#### パラメータ

- データキャッシュの新しいパラメータが追加されました。

#### システム変数

更新予定。

### バグ修正

以下の問題を修正しました：

- libcurlが呼び出されたときにBEsクラッシュします。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- スキーマ変更によって指定されたタブレットバージョンがガベージコレクションによって処理されるため、スキーマ変更に過度の時間がかかる場合があります。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- MinIOまたはAWS S3のParquetファイルにファイル外部テーブル経由でアクセスできないことがあります。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- `information_schema.columns`でARRAY、MAP、およびSTRUCT型の列が正しく表示されません。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- BINARYまたはVARBINARYデータ型の `DATA_TYPE` および `COLUMN_TYPE` が `information_schema.columns` ビューで `unknown` と表示されます。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### アップグレードノート

更新予定。