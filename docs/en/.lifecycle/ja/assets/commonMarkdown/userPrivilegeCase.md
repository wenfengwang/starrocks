特権とユーザーを管理するために、役割をカスタマイズすることをお勧めします。以下の例は、いくつかの一般的なシナリオにおける特権の組み合わせを分類しています。

#### StarRocksテーブルに対するグローバルな読み取り専用特権を付与する

   ```SQL
   -- 役割を作成します。
   CREATE ROLE read_only;
   -- すべてのカタログに対するUSAGE特権を役割に付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE read_only;
   -- すべてのデータベースのすべてのテーブルに対するクエリ特権を役割に付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_only;
   -- すべてのデータベースのすべてのビューに対するクエリ特権を役割に付与します。
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_only;
   -- すべてのデータベースのすべてのマテリアライズドビューに対するクエリ特権と、それらを使用してクエリを高速化する特権を役割に付与します。
   GRANT SELECT ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE read_only;
   ```

   また、クエリでUDFを使用する特権をさらに付与することもできます：

   ```SQL
   -- すべてのデータベースレベルのUDFに対するUSAGE特権を役割に付与します。
   GRANT USAGE ON ALL FUNCTIONS IN ALL DATABASES TO ROLE read_only;
   -- グローバルUDFに対するUSAGE特権を役割に付与します。
   GRANT USAGE ON ALL GLOBAL FUNCTIONS TO ROLE read_only;
   ```

#### StarRocksテーブルに対するグローバルな書き込み特権を付与する

   ```SQL
   -- 役割を作成します。
   CREATE ROLE write_only;
   -- すべてのカタログに対するUSAGE特権を役割に付与します。
   GRANT USAGE ON ALL CATALOGS TO ROLE write_only;
   -- すべてのデータベースのすべてのテーブルに対するINSERTおよびUPDATE特権を役割に付与します。
   GRANT INSERT, UPDATE ON ALL TABLES IN ALL DATABASES TO ROLE write_only;
   -- すべてのデータベースのすべてのマテリアライズドビューに対するREFRESH特権を役割に付与します。
   GRANT REFRESH ON ALL MATERIALIZED VIEWS IN ALL DATABASES TO ROLE write_only;
   ```

#### 特定の外部カタログに対する読み取り専用特権を付与する

   ```SQL
   -- 役割を作成します。
   CREATE ROLE read_catalog_only;
   -- 宛先カタログに対するUSAGE特権を役割に付与します。
   GRANT USAGE ON CATALOG hive_catalog TO ROLE read_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG hive_catalog;
   -- すべてのデータベースのすべてのテーブルとすべてのビューに対する特権を役割に付与します。
   GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE read_catalog_only;
   GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE read_catalog_only;
   ```

   注意：Hiveテーブルビューのみクエリできます（v3.1以降）。

#### 特定の外部カタログに対する書き込み専用特権を付与する

Icebergテーブル（v3.1以降）およびHiveテーブル（v3.2以降）にのみデータを書き込むことができます。

   ```SQL
   -- 役割を作成します。
   CREATE ROLE write_catalog_only;
   -- 宛先カタログに対するUSAGE特権を役割に付与します。
   GRANT USAGE ON CATALOG iceberg_catalog TO ROLE read_catalog_only;
   -- 対応するカタログに切り替えます。
   SET CATALOG iceberg_catalog;
   -- Icebergテーブルにデータを書き込む特権を付与します。
   GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE write_catalog_only;
   ```

#### グローバル、データベース、テーブル、およびパーティションレベルでのバックアップおよびリストア操作を実行するための特権を付与する

- グローバルなバックアップおよびリストア操作を実行するための特権を付与する：

     グローバルなバックアップおよびリストア操作を実行するための特権は、役割が任意のデータベース、テーブル、またはパーティションをバックアップおよびリストアできるようにします。これには、SYSTEMレベルでのREPOSITORY特権、デフォルトカタログでのデータベースの作成特権、任意のデータベースでのテーブルの作成特権、および任意のテーブルでのデータのロードとエクスポート特権が必要です。

     ```SQL
     -- 役割を作成します。
     CREATE ROLE recover;
     -- SYSTEMレベルでのREPOSITORY特権を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover;
     -- デフォルトカタログでのデータベースの作成特権を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover;
     -- 任意のデータベースでのテーブルの作成特権を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover;
     -- 任意のテーブルでのデータのロードとエクスポート特権を付与します。
     GRANT INSERT, EXPORT ON ALL TABLES IN ALL DATABASES TO ROLE recover;
     ```

- データベースレベルでのバックアップおよびリストア操作を実行するための特権を付与する：

     データベースレベルでのバックアップおよびリストア操作を実行するための特権には、SYSTEMレベルでのREPOSITORY特権、デフォルトカタログでのデータベースの作成特権、任意のデータベースでのテーブルの作成特権、任意のテーブルにデータをロードする特権、およびバックアップ対象のデータベースの任意のテーブルからデータをエクスポートする特権が必要です。

     ```SQL
     -- 役割を作成します。
     CREATE ROLE recover_db;
     -- SYSTEMレベルでのREPOSITORY特権を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_db;
     -- デフォルトカタログでのデータベースの作成特権を付与します。
     GRANT CREATE DATABASE ON CATALOG default_catalog TO ROLE recover_db;
     -- 任意のデータベースでのテーブルの作成特権を付与します。
     GRANT CREATE TABLE ON ALL DATABASE TO ROLE recover_db;
     -- 任意のテーブルにデータをロードする特権を付与します。
     GRANT INSERT ON ALL TABLES IN ALL DATABASES TO ROLE recover_db;
     -- バックアップ対象のデータベースの任意のテーブルからデータをエクスポートする特権を付与します。
     GRANT EXPORT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     ```

- テーブルレベルでのバックアップおよびリストア操作を実行するための特権を付与する：

     テーブルレベルでのバックアップおよびリストア操作を実行するための特権には、SYSTEMレベルでのREPOSITORY特権、対応するデータベースでのテーブルの作成特権、データベース内の任意のテーブルにデータをロードする特権、およびバックアップ対象のテーブルからデータをエクスポートする特権が必要です。

     ```SQL
     -- 役割を作成します。
     CREATE ROLE recover_tbl;
     -- SYSTEMレベルでのREPOSITORY特権を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_tbl;
     -- 対応するデータベースでのテーブルの作成特権を付与します。
     GRANT CREATE TABLE ON DATABASE <db_name> TO ROLE recover_tbl;
     -- データベース内の任意のテーブルにデータをロードする特権を付与します。
     GRANT INSERT ON ALL TABLES IN DATABASE <db_name> TO ROLE recover_db;
     -- バックアップ対象のテーブルからデータをエクスポートする特権を付与します。
     GRANT EXPORT ON TABLE <table_name> TO ROLE recover_tbl;     
     ```

- パーティションレベルでのバックアップおよびリストア操作を実行するための特権を付与する：

     パーティションレベルでのバックアップおよびリストア操作を実行するための特権には、SYSTEMレベルでのREPOSITORY特権、および対応するテーブルでのデータのロードとエクスポート特権が必要です。

     ```SQL
     -- 役割を作成します。
     CREATE ROLE recover_par;
     -- SYSTEMレベルでのREPOSITORY特権を付与します。
     GRANT REPOSITORY ON SYSTEM TO ROLE recover_par;
     -- 対応するテーブルでのデータのロードとエクスポート特権を付与します。
     GRANT INSERT, EXPORT ON TABLE <table_name> TO ROLE recover_par;
     ```
