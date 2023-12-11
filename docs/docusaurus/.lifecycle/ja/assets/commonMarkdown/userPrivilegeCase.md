ユーザーの権限を管理するために役割をカスタマイズすることをお勧めします。以下の例は、一部の一般的なシナリオに対する特権の組み合わせをいくつか分類したものです。

#### StarRocks テーブルに対するグローバルな読み取り専用権限を付与

   ```SQL
   -- ロールを作成します。
   CREATE ROLE read_only;
   -- そのロールにすべてのカタログの USAGE 権限を付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE read_only;
   -- そのロールにすべてのデータベースのすべてのテーブルのクエリ権限を付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_only;
   -- そのロールにすべてのデータベースのすべてのビューのクエリ権限を付与します。
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_only;
   -- そのロールにすべてのデータベースのすべてのマテリアライズド ビューのクエリ権限とそれらを使用してクエリの高速化を行う権限を付与します。
   GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE read_only;
   ```

   そして、クエリで UDF を使用する権限をさらに付与することができます：

   ```SQL
   -- すべてのデータベースレベルの UDF に対する USAGE 権限をそのロールに付与します。
   GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE read_only;
   -- グローバル UDF に対する USAGE 権限をそのロールに付与します。
   GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE read_only;
   ```

#### StarRocks テーブルに対するグローバルな書き込み権限を付与

   ```SQL
   -- ロールを作成します。
   CREATE ROLE write_only;
   -- そのロールにすべてのカタログの USAGE 権限を付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE write_only;
   -- そのロールにすべてのデータベースのすべてのテーブルへの INSERT および UPDATE 権限を付与します。
   GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE write_only;
   -- そのロールにすべてのデータベースのすべてのマテリアライズド ビューの REFRESH 権限を付与します。
   GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE write_only;
   ```

#### 特定の外部カタログに対する読み取り専用権限を付与

   ```SQL
   -- ロールを作成します。
   CREATE ROLE read_catalog_only;
   -- そのロールに対象のカタログへの USAGE 権限を付与します。
   GRANT USAGE ON CATALOG hive_catalog TO ROLE read_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG hive_catalog;
   -- すべてのデータベースのすべてのテーブルとすべてのビューのクエリ権限を付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_catalog_only;
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_catalog_only;
   ```

   注：Hive テーブルビューだけをクエリできます（v3.1 以降）。

#### 特定の外部カタログに対する書き込み専用権限を付与

Iceberg テーブル（v3.1 以降）および Hive テーブル（v3.2 以降）にのみデータを書き込むことができます。

   ```SQL
   -- ロールを作成します。
   CREATE ROLE write_catalog_only;
   -- そのロールに対象のカタログへの USAGE 権限を付与します。
   GRANT USAGE ON CATALOG iceberg_catalog TO ROLE read_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG iceberg_catalog;
   -- Iceberg テーブルへのデータ書き込み権限を付与します。
   GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE write_catalog_only;
   ```

#### グローバル、データベース、テーブル、およびパーティションレベルでのバックアップおよびリストア操作を実行するための権限を付与

- グローバルなバックアップおよびリストア操作を実行する権限を付与します：

     グローバルなバックアップおよびリストア操作を実行する権限は、そのロールにシステムレベルでの REPOSITORY 権限、デフォルトカタログでのデータベース作成権限、すべてのデータベースでのテーブル作成権限、およびすべてのテーブルでのデータの読み込みとエクスポート権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover;
     -- システムレベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover;
     -- デフォルトカタログでのデータベース作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover;
     -- すべてのデータベースでのテーブル作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover;
     -- すべてのデータベースでのデータの読み込みとエクスポート権限を付与します。
     GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE recover;
     ```

- データベースレベルでのバックアップおよびリストア操作を実行する権限を付与します：

     データベースレベルでのバックアップおよびリストア操作を実行する権限には、そのロールにシステムレベルでの REPOSITORY 権限、デフォルトカタログでのデータベース作成権限、すべてのデータベースでのテーブル作成権限、すべてのテーブルでのデータの読み込み権限、およびバックアップ対象データベース内の任意のテーブルからデータのエクスポート権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_db;
     -- システムレベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
     -- デフォルトカタログでのデータベース作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
     -- すべてのデータベースでのテーブル作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover_db;
     -- すべてのデータベースでのデータの読み込み権限を付与します。
     GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
     -- バックアップ対象データベース内の任意のテーブルからデータのエクスポート権限を付与します。
     GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     ```

- テーブルレベルでのバックアップおよびリストア操作を実行する権限を付与します：

     テーブルレベルでのバックアップおよびリストア操作を実行する権限には、そのロールにシステムレベルでの REPOSITORY 権限、対応するデータベース内でのテーブル作成権限、対応するデータベース内の任意のテーブルでのデータの読み込み権限、およびバックアップ対象テーブルからデータのエクスポート権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_tbl;
     -- システムレベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
     -- 対応するデータベース内でのテーブル作成権限を付与します。
     GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE recover_tbl;
     -- 対応するデータベース内の任意のテーブルでのデータの読み込み権限を付与します。
     GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     -- バックアップ対象テーブルからデータのエクスポート権限を付与します。
     GRANT EXPORT ON TABLE <table_name> TO ROLE recover_tbl;     
     ```

- パーティションレベルでのバックアップおよびリストア操作を実行する権限を付与します：

     パーティションレベルでのバックアップおよびリストア操作を実行する権限には、そのロールにシステムレベルでの REPOSITORY 権限、対応するテーブルでのデータの読み込みおよびエクスポート権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_par;
     -- システムレベルでの REPOSITORY 権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_par;
     -- 対応するテーブルでのデータの読み込みおよびエクスポート権限を付与します。
     GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE recover_par;
     ```