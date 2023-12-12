ユーザーには、権限とユーザーを管理するために、ロールをカスタマイズすることをお勧めします。以下の例では、一部の一般的なシナリオにおける権限のいくつかの組み合わせを分類しています。

#### StarRocksテーブルにグローバルな読み取り専用権限を付与する

   ```SQL
   -- ロールを作成します。
   CREATE ROLE read_only;
   -- すべてのカタログに対するUSAGE権限をロールに付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE read_only;
   -- ロールに全てのデータベースの全てのテーブルへのクエリ権限を付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_only;
   -- ロールに全てのデータベースの全てのビューへのクエリ権限を付与します。
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_only;
   -- ロールに全てのデータベースの全てのマテリアライズドビューへのクエリ権限とそれらを使用したクエリを高速化する権限を付与します。
   GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE read_only;
   ```

   そして、クエリでUDFを使用する権限をさらに付与できます。

   ```SQL
   -- データベースレベルのUDFに対するUSAGE権限をロールに付与します。
   GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE read_only;
   -- グローバルなUDFに対するUSAGE権限をロールに付与します。
   GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE read_only;
   ```

#### StarRocksテーブルにグローバルな書き込み権限を付与する

   ```SQL
   -- ロールを作成します。
   CREATE ROLE write_only;
   -- すべてのカタログに対するUSAGE権限をロールに付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE write_only;
   -- 全てのデータベースの全てのテーブルに対するINSERTおよびUPDATE権限をロールに付与します。
   GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE write_only;
   -- 全てのデータベースの全てのマテリアライズドビューに対するREFRESH権限をロールに付与します。
   GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE write_only;
   ```

#### 特定の外部カタログに対する読み取り専用権限を付与する

   ```SQL
   -- ロールを作成します。
   CREATE ROLE read_catalog_only;
   -- 宛先カタログに対するUSAGE権限をロールに付与します。
   GRANT USAGE ON CATALOG hive_catalog TO ROLE read_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG hive_catalog;
   -- 全てのデータベースの全てのテーブルおよび全てのビューに対するクエリ権限を付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_catalog_only;
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_catalog_only;
   ```

   注意：Hiveテーブルビューのみクエリできます（v3.1以降）。

#### 特定の外部カタログに対する書き込み専用権限を付与する

アイスバーグテーブル（v3.1以降）およびHiveテーブル（v3.2以降）にのみデータを書き込むことができます。

   ```SQL
   -- ロールを作成します。
   CREATE ROLE write_catalog_only;
   -- 宛先カタログに対するUSAGE権限をロールに付与します。
   GRANT USAGE ON CATALOG iceberg_catalog TO ROLE read_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG iceberg_catalog;
   -- アイスバーグテーブルにデータを書き込む権限を付与します。
   GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE write_catalog_only;
   ```

#### グローバル、データベース、テーブル、およびパーティションレベルでバックアップおよびリストア操作を実行する権限を付与する

- グローバルなバックアップおよびリストア操作を実行する権限を付与する：

     グローバルなバックアップおよびリストア操作を実行する権限を付与すると、そのロールは任意のデータベース、テーブル、またはパーティションをバックアップおよびリストアできます。これにはシステムレベルでのREPOSITORY権限、デフォルトカタログでのデータベースの作成権限、任意のデータベースでのテーブルの作成権限、および任意のテーブルでのデータのロードおよびエクスポート権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover;
     -- システムレベルでのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover;
     -- デフォルトカタログでのデータベースの作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover;
     -- 任意のデータベースでのテーブルの作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover;
     -- 任意のテーブルでのデータのロードおよびエクポート権限を付与します。
     GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE recover;
     ```

- データベースレベルのバックアップおよびリストア操作を実行する権限を付与する：

     データベースレベルのバックアップおよびリストア操作を実行する権限を付与するには、システムレベルでのREPOSITORY権限、デフォルトカタログでのデータベースの作成権限、任意のデータベースでのテーブルの作成権限、任意のテーブルにデータをロードする権限、およびバックアップ対象のデータベースのテーブルからデータをエクスポートする権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_db;
     -- システムレベルでのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
     -- デフォルトカタログでのデータベースの作成権限を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
     -- 任意のデータベースでのテーブルの作成権限を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover_db;
     -- 任意のテーブルにデータをロードする権限を付与します。
     GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
     -- バックアップ対象のデータベースのテーブルからデータをエクスポートする権限を付与します。
     GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     ```

- テーブルレベルのバックアップおよびリストア操作を実行する権限を付与する：

     テーブルレベルのバックアップおよびリストア操作を実行する権限を付与するには、システムレベルでのREPOSITORY権限、対応するデータベースでのテーブルの作成権限、対応するデータベースの任意のテーブルにデータをロードする権限、およびバックアップ対象のテーブルからデータをエクスポートする権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_tbl;
     -- システムレベルでのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
     -- 対応するデータベースでのテーブルの作成権限を付与します。
     GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE recover_tbl;
     -- 対応するデータベースの任意のテーブルにデータをロードする権限を付与します。
     GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     -- バックアップ対象のテーブルからデータをエクスポートする権限を付与します。
     GRANT EXPORT ON TABLE <table_name> TO ROLE recover_tbl;     
     ```

- パーティションレベルのバックアップおよびリストア操作を実行する権限を付与する：

     パーティションレベルのバックアップおよびリストア操作を実行する権限を付与するには、システムレベルでのREPOSITORY権限、対応するテーブルでのデータのロードおよびエクスポート権限が必要です。

     ```SQL
     -- ロールを作成します。
     CREATE ROLE recover_par;
     -- システムレベルでのREPOSITORY権限を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_par;
     -- 対応するテーブルでのデータのロードおよびエクスポート権限を付与します。
     GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE recover_par;
     ```