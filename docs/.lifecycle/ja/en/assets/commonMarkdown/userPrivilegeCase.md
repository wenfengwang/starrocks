
ロールをカスタマイズして、権限とユーザーを管理することをお勧めします。以下の例は、一般的なシナリオにおける権限の組み合わせをいくつか分類したものです。

#### StarRocks テーブルに対するグローバル読み取り専用権限の付与

   ```SQL
   -- ロールを作成します。
   CREATE ROLE read_only;
   -- すべてのカタログに対する USAGE 権限をロールに付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE read_only;
   -- すべてのテーブルに対するクエリ権限をロールに付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_only;
   -- すべてのビューに対するクエリ権限をロールに付与します。
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_only;
   -- すべてのマテリアライズドビューに対するクエリ権限と、それらを使用したクエリの高速化権限をロールに付与します。
   GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE read_only;
   ```

   さらに、クエリ内で UDF を使用する権限を付与することができます：

   ```SQL
   -- すべてのデータベースレベルの UDF に対する USAGE 権限をロールに付与します。
   GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE read_only;
   -- グローバル UDF に対する USAGE 権限をロールに付与します。
   GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE read_only;
   ```

#### StarRocks テーブルに対するグローバル書き込み権限の付与

   ```SQL
   -- ロールを作成します。
   CREATE ROLE write_only;
   -- すべてのカタログに対する USAGE 権限をロールに付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE write_only;
   -- すべてのテーブルに対する INSERT および UPDATE 権限をロールに付与します。
   GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE write_only;
   -- すべてのマテリアライズドビューに対する REFRESH 権限をロールに付与します。
   GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE write_only;
   ```

#### 特定の外部カタログに対する読み取り専用権限の付与

   ```SQL
   -- ロールを作成します。
   CREATE ROLE read_catalog_only;
   -- 対象カタログに対する USAGE 権限をロールに付与します。
   GRANT USAGE ON CATALOG hive_catalog TO ROLE read_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG hive_catalog;
   -- すべてのデータベースのすべてのテーブルとビューに対するクエリ権限を付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_catalog_only;
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_catalog_only;
   ```

   注意：Hive テーブルビューのみをクエリできます（v3.1 以降）。

#### 特定の外部カタログに対する書き込み専用権限の付与

Iceberg テーブル（v3.1 以降）および Hive テーブル（v3.2 以降）にのみデータを書き込むことができます。

   ```SQL
   -- ロールを作成します。
   CREATE ROLE write_catalog_only;
   -- 対象カタログに対する USAGE 権限をロールに付与します。
   GRANT USAGE ON CATALOG iceberg_catalog TO ROLE write_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG iceberg_catalog;
   -- Iceberg テーブルにデータを書き込む権限を付与します。
   GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE write_catalog_only;
   ```

#### グローバル、データベース、テーブル、パーティションレベルでのバックアップおよび復元操作を実行する権限の付与

- グローバルバックアップおよび復元操作を実行する権限を付与します：

     グローバルバックアップおよび復元操作を実行する権限は、任意のデータベース、テーブル、またはパーティションのバックアップおよび復元をロールに許可します。これには、SYSTEM レベルでの REPOSITORY 権限、デフォルトカタログでのデータベース作成権限、任意のデータベースでのテーブル作成権限、および任意のテーブルでのデータのロードおよびエクスポート権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover;
     -- SYSTEM レベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover;
     -- デフォルトカタログでのデータベース作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover;
     -- 任意のデータベースでのテーブル作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASES TO ROLE recover;
     -- 任意のテーブルでのデータのロードおよびエクスポート権限を付与します。
     GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE recover;
     ```

- データベースレベルのバックアップおよび復元操作を実行する権限を付与します：

     データベースレベルのバックアップおよび復元操作を実行する権限には、SYSTEM レベルでの REPOSITORY 権限、デフォルトカタログでのデータベース作成権限、任意のデータベースでのテーブル作成権限、任意のテーブルにデータをロードする権限、およびバックアップ対象のデータベース内の任意のテーブルからデータをエクスポートする権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_db;
     -- SYSTEM レベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
     -- データベース作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
     -- テーブル作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASES TO ROLE recover_db;
     -- 任意のテーブルにデータをロードする権限を付与します。
     GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
     -- バックアップ対象のデータベース内の任意のテーブルからデータをエクスポートする権限を付与します。
     GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     ```

- テーブルレベルのバックアップおよび復元操作を実行する権限を付与します：

     テーブルレベルのバックアップおよび復元操作を実行する権限には、SYSTEM レベルでの REPOSITORY 権限、対応するデータベースでのテーブル作成権限、データベース内の任意のテーブルにデータをロードする権限、およびバックアップ対象のテーブルからデータをエクスポートする権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_tbl;
     -- SYSTEM レベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
     -- 対応するデータベースでのテーブル作成権限を付与します。
     GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE recover_tbl;
     -- データベース内の任意のテーブルにデータをロードする権限を付与します。
     GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_tbl;
     -- バックアップ対象のテーブルからデータをエクスポートする権限を付与します。
     GRANT EXPORT ON TABLE <table_name> TO ROLE recover_tbl;     
     ```

- パーティションレベルのバックアップおよび復元操作を実行する権限を付与します：

     パーティションレベルのバックアップおよび復元操作を実行する権限には、SYSTEM レベルでの REPOSITORY 権限、および対応するテーブルにデータをロードおよびエクスポートする権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_par;
     -- SYSTEM レベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_par;
     -- 対応するテーブルにデータをロードおよびエクスポートする権限を付与します。
     GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE recover_par;
     ```
