
カスタムロールを使用して権限とユーザーを管理することをお勧めします。以下は、一般的なシナリオで必要とされる権限項目を整理したものです。

1. StarRocks 内部テーブルのグローバルクエリ権限

   ```SQL
   -- カスタムロールを作成します。
   CREATE ROLE read_only;
   -- ロールにすべてのカタログの使用権限を付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE read_only;
   -- ロールにすべてのテーブルのクエリ権限を付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_only;
   -- ロールにすべてのビューのクエリ権限を付与します。
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_only;
   -- ロールにすべてのマテリアライズドビューのクエリとアクセラレーション権限を付与します。
   GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE read_only;
   ```

   さらに、ロールにクエリ内でUDFを使用する権限を付与することもできます：

   ```SQL
   -- ロールにすべてのデータベースレベルのUDFの使用権限を付与します。
   GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE read_only;
   -- ロールにすべてのグローバルUDFの使用権限を付与します。
   GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE read_only;
   ```

2. StarRocks 内部テーブルのグローバル書き込み権限

   ```SQL
   -- カスタムロールを作成します。
   CREATE ROLE write_only;
   -- ロールにすべてのカタログの使用権限を付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE write_only;
   -- ロールにすべてのテーブルのインポートおよび更新権限を付与します。
   GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE write_only;
   -- ロールにすべてのマテリアライズドビューの更新権限を付与します。
   GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE write_only;
   ```

3. 特定の外部データディレクトリ（External Catalog）のクエリ権限

   ```SQL
   -- カスタムロールを作成します。
   CREATE ROLE read_catalog_only;
   -- ロールに対象カタログのUSAGE権限を付与します。
   GRANT USAGE ON CATALOG hive_catalog TO ROLE read_catalog_only;
   -- 対応するデータディレクトリに切り替えます。
   SET CATALOG hive_catalog;
   -- ロールにすべてのテーブルのクエリ権限を付与します。現在はHiveテーブルのビューのクエリのみをサポートしています（バージョン3.1以降）。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_catalog_only;
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_catalog_only;
   ```

4. 特定の外部データディレクトリ（External Catalog）の書き込み権限

   現在、Icebergテーブル（バージョン3.1以降）とHiveテーブル（バージョン3.2以降）へのデータ書き込みのみをサポートしています。

   ```SQL
   -- カスタムロールを作成します。
   CREATE ROLE write_catalog_only;
   -- ロールに対象カタログのUSAGE権限を付与します。
   GRANT USAGE ON CATALOG iceberg_catalog TO ROLE write_catalog_only;
   -- 対応するデータディレクトリに切り替えます。
   SET CATALOG iceberg_catalog;
   -- ロールにすべてのIcebergテーブルの書き込み権限を付与します。
   GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE write_catalog_only;
   ```

5. グローバル、データベースレベル、テーブルレベル、およびパーティションレベルのバックアップおよびリカバリ権限

   - グローバルバックアップおよびリカバリ権限

     グローバルバックアップおよびリカバリ権限では、任意のデータベース、テーブル、パーティションに対してバックアップおよびリカバリを行うことができます。SYSTEMレベルのREPOSITORY権限、Default Catalogでのデータベース作成権限、任意のデータベースでのテーブル作成権限、および任意のテーブルへのインポートおよびエクスポート権限が必要です。

     ```SQL
     -- カスタムロールを作成します。
     CREATE ROLE recover;
     -- ロールにSYSTEMレベルのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover;
     -- ロールにデータベース作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover;
     -- ロールに任意のテーブル作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover;
     -- ロールに任意のテーブルへのインポートおよびエクスポート権限を付与します。
     GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE recover;
     ```

   - データベースレベルのバックアップおよびリカバリ権限

     データベースレベルのバックアップおよびリカバリ権限では、特定のデータベース全体に対してバックアップおよびリカバリを行うことができます。SYSTEMレベルのREPOSITORY権限、Default Catalogでのデータベース作成権限、任意のデータベースでのテーブル作成権限、およびバックアップ対象データベース内のすべてのテーブルのエクスポート権限が必要です。

     ```SQL
     -- カスタムロールを作成します。
     CREATE ROLE recover_db;
     -- ロールにSYSTEMレベルのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
     -- ロールにデータベース作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
     -- ロールに任意のテーブル作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover_db;
     -- ロールに任意のテーブルへのインポート権限を付与します。
     GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
     -- ロールにバックアップ対象データベース内のすべてのテーブルのエクスポート権限を付与します。
     GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     ```

   - テーブルレベルのバックアップおよびリカバリ権限

     テーブルレベルのバックアップおよびリカバリ権限では、特定のテーブルに対してバックアップおよびリカバリを行うことができます。SYSTEMレベルのREPOSITORY権限、バックアップ対象データベースでのテーブル作成およびデータインポート権限、およびバックアップ対象テーブルのエクスポート権限が必要です。

     ```SQL
     -- カスタムロールを作成します。
     CREATE ROLE recover_tbl;
     -- ロールにSYSTEMレベルのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
     -- ロールに対象データベースでのテーブル作成権限を付与します。
     GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE recover_tbl;
     -- ロールに任意のテーブルへのインポート権限を付与します。
     GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_tbl;
     -- ロールにバックアップ対象テーブルのエクスポート権限を付与します。
     GRANT EXPORT ON TABLE <table_name> TO ROLE recover_tbl;     
     ```

   - パーティションレベルのバックアップおよびリカバリ権限

     パーティションレベルのバックアップおよびリカバリ権限では、特定のテーブルのパーティションに対してバックアップおよびリカバリを行うことができます。SYSTEMレベルのREPOSITORY権限およびバックアップ対象テーブルへのインポートおよびエクスポート権限が必要です。

     ```SQL
     -- カスタムロールを作成します。
     CREATE ROLE recover_par;
     -- ロールにSYSTEMレベルのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_par;
     -- ロールに対象テーブルへのインポートおよびエクスポート権限を付与します。
     GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE recover_par;
     ```
