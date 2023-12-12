---
displayed_sidebar: "Japanese"
---

# StarRocks バージョン 3.2

## v3.2.0-RC01

リリース日: 2023年11月15日

### 新機能

#### 共有データクラスター

- ローカルディスク上のプライマリキーテーブルの永続的なインデックスをサポートします。([プライマリキーテーブルのテーブルデザイン](../table_design/table_types/primary_key_table.md))
- 複数のローカルディスク間でデータキャッシュを均等に分散することをサポートします。

#### Data Lake アナリティクス

- [Hiveカタログ](../data_source/catalog/hive_catalog.md)内でデータベースおよび管理対象テーブルの作成および削除をサポートし、INSERTまたはINSERT OVERWRITEを使用してHiveの管理対象テーブルにデータをエクスポートすることをサポートします。
- [Unified Catalog](../data_source/catalog/unified_catalog.md)をサポートし、HiveメタストアやAWS Glueなどの共通のメタストアを共有するHive、Iceberg、Hudi、Delta Lakeなどの異なるテーブル形式にアクセスできるようにします。

#### ストレージエンジン、データ取込、およびエクスポート

- テーブル関数 [FILES()](../sql-reference/sql-functions/table-functions/files.md) の次の機能が追加されました。
  - AzureまたはGCPからParquetおよびORC形式のデータの読み込み。
  - ファイルパスからキー/値ペアの値を列の値として抽出し、パラメータ `columns_from_path` を使用して列の値を読み込みます。
  - ARRAY、JSON、MAP、およびSTRUCTを含む複雑なデータ型の読み込み。
- グローバルディクショナリの構築中にデータロードプロセスを大幅に容易にする dict_mapping カラムプロパティをサポートし、正確な COUNT DISTINCT の計算を高速化します。
- StarRocksからAWS S3またはHDFSに格納されたParquet形式のファイルにデータをアンロードすることをサポートし、INSERT INTO FILESを使用します。詳細な手順については、[INSERT INTO FILESを使用したデータのアンロード](../unloading/unload_using_insert_into_files.md) を参照してください。

#### SQL リファレンス

次の関数が追加されました。

- 文字列関数: substring_index、url_extract_parameter、url_encode、url_decode、translate
- 日付関数: dayofweek_iso、week_iso、quarters_add、quarters_sub、milliseconds_add、milliseconds_sub、date_diff、jodatime_format、str_to_jodatime、to_iso8601、to_tera_date、to_tera_timestamp
- パターンマッチング関数: regexp_extract_all
- ハッシュ関数: xx_hash3_64
- 集約関数: approx_top_k
- ウィンドウ関数: cume_dist、percent_rank、session_number
- ユーティリティ関数: dict_mapping、get_query_profile

#### 権限およびセキュリティ

StarRocksはApache Rangerを介したアクセス制御をサポートし、データセキュリティのレベルを高め、外部データソースの既存のRangerサービスを再利用することを可能にします。Apache Rangerと統合した後、StarRocksは次のアクセス制御方法を有効にします。

- StarRocksの内部テーブル、外部テーブル、または他のオブジェクトにアクセスする場合、Rangerで構成されたStarRocksサービスのアクセスポリシーに基づいてアクセス制御を実施できます。
- 外部カタログにアクセスする場合、対応するオリジナルデータソース（Hiveサービスなど）のRangerサービスも利用してアクセス制御を行うことができます（現在、Hiveへのデータのエクスポートに対するアクセス制御はサポートされていません）。

詳細については、[Apache Rangerで権限を管理する](../administration/ranger_plugin.md) を参照してください。

### 改善点

#### マテリアライズドビュー

非同期マテリアライズドビュー

- 作成：

  ビューまたはマテリアライズドビューがスキーマ変更を起こした場合に、非同期マテリアライズドビューに自動的に更新をサポートします。

- 可観測性：

  非同期マテリアライズドビューのクエリダンプをサポートします。

- スピル・トゥ・ディスク機能は、非同期マテリアライズドビューのリフレッシュタスクのメモリ消費を削減し、デフォルトで有効になっています。
- データ整合性：

  - 非同期マテリアライズドビューの作成に対する query_rewrite_consistency プロパティが追加されました。このプロパティは整合性チェックに基づくクエリの書き換えルールを定義します。
  - 外部カタログベースの非同期マテリアライズドビューの作成に対する force_external_table_query_rewrite プロパティが追加されました。このプロパティは外部カタログに基づく非同期マテリアライズドビューの強制クエリ書き換えを許可するかどうかを定義します。

  詳細については、[CREATE MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/CREATE_MATERIALIZED_VIEW.md)を参照してください。

- マテリアライズドビューのパーティショニングキーに対する整合性チェックが追加されました。

  ユーザーが非同期マテリアライズドビューを作成する際に、PARTITION BY式を含むウィンドウ関数を使用する場合、ウィンドウ関数のパーティションキーはマテリアライズドビューのパーティションキーと一致する必要があります。

#### ストレージエンジン、データ取込、およびエクスポート

- 主キーテーブルの永続インデックスの最適化により、メモリ使用量ロジックが改善され、I/Oの読み込みと書き込みの増幅が削減されました。[#24875](https://github.com/StarRocks/starrocks/pull/24875)  [#27577](https://github.com/StarRocks/starrocks/pull/27577)  [#28769](https://github.com/StarRocks/starrocks/pull/28769)
- プライマリーキーテーブルのローカルディスク間でのデータ再分配をサポートします。
- パーティションテーブルは、パーティションの時間範囲および冷却時間に基づいて自動的な冷却をサポートします。詳細については、[初期ストレージ媒体、自動ストレージ冷却時間、レプリカ数を指定する](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md#specify-initial-storage-medium-automatic-storage-cooldown-time-replica-number)を参照してください。
- プライマリーキーテーブルにデータを書き込むロードジョブのPublish段階が非同期モードから同期モードに変更されました。そのため、データはロードジョブが終了した後にすぐにクエリ可能になります。詳細については、[enable_sync_publish](../administration/Configuration.md#enable_sync_publish)を参照してください。

#### クエリ

- StarRocksのMetabaseおよびSupersetとの互換性を最適化し、外部カタログと統合をサポートします。

#### SQL リファレンス

- array_agg は DISTINCT キーワードをサポートします。

### 開発者ツール

- 非同期マテリアライズドビューに対するTraceクエリプロファイルのサポートを追加し、その透過的な書き換えを解析するために使用できます。

### 互換性変更

#### 動作変更

更新予定です。

#### パラメータ

- データキャッシュの新しいパラメータが追加されました。

#### システム変数

更新予定です。

### バグ修正

以下の問題が修正されました。

- libcurlを呼び出した際にBEsがクラッシュする問題が修正されました。[#31667](https://github.com/StarRocks/starrocks/pull/31667)
- スキーマ変更が過度な時間を要する場合に失敗する可能性がある問題が修正されました。指定されたタブレットバージョンがガベージコレクションによって処理されるためです。[#31376](https://github.com/StarRocks/starrocks/pull/31376)
- file外部テーブルを介してMinIOまたはAWS S3のParquetファイルにアクセスできない問題が修正されました。[#29873](https://github.com/StarRocks/starrocks/pull/29873)
- ARRAY、MAP、およびSTRUCTタイプの列が `information_schema.columns` で正しく表示されない問題が修正されました。[#33431](https://github.com/StarRocks/starrocks/pull/33431)
- `information_schema.columns` ビューでBINARYまたはVARBINARYデータ型の `DATA_TYPE` および `COLUMN_TYPE` が `unknown` として表示される問題が修正されました。[#32678](https://github.com/StarRocks/starrocks/pull/32678)

### アップグレードノート

更新予定です。
