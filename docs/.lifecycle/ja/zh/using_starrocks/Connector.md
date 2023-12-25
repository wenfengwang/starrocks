---
displayed_sidebar: Chinese
---

# コネクタ

## 概念

### カタログ

StarRocksのカタログには、Internal CatalogとExternal Catalogの二種類があります。カタログにはユーザーのすべてのデータベースが含まれています。現在のStarRocksには、`default_catalog`というデフォルト値を持つデフォルトのInternal Catalogインスタンスが存在します。StarRocksに保存されているDatabase/Table/ViewなどはすべてInternal Catalogに属しており、例えばOlapTableやExternal Hive Tableがあります。External Catalogは、Create Catalog DDLステートメントを実行することでユーザーが作成します。各External Catalogには、外部データソースの情報を取得するためのConnectorが存在します。現在のバージョンではHive Connectorのみがサポートされています。ユーザーはfully-qualified nameを指定して特定のカタログのデータテーブルをクエリすることができます。例えば、`hive_catalog.hive_db.hive_table`を指定してユーザー定義のhive_catalog内のテーブルをクエリしたり、`default_catalog.my_db.my_olap_table`を指定してOlapテーブルをクエリしたりすることができます。また、異なるカタログ間でのフェデレーションクエリも可能です。

### コネクタ

StarRocksのConnectorは、ユーザーが定義したCatalogに対応するデータソースのコネクタです。Connectorを通じて、StarRocksは実行時に必要なテーブル情報やスキャン対象のファイル情報などを取得できます。現在、各External CatalogにはそれぞれConnectorインスタンスが対応しています。ConnectorはExternal Catalogを作成する過程で完成します。ユーザーは必要に応じてカスタムConnectorを実装することができます。

## 使用方法

### カタログの作成

#### 文法

```sql
CREATE EXTERNAL CATALOG <catalog_name> PROPERTIES ("key"="value", ...);
```

#### 例

Hiveカタログの作成

```sql
CREATE EXTERNAL CATALOG hive_catalog0 PROPERTIES("type"="hive", "hive.metastore.uris"="thrift://xx.xx.xx.xx:9083");
```

#### 説明

* 現在サポートされているのはhive外部カタログの作成のみです
* PROPERTIES内の`type`は必須項目で、hiveカタログの値は`hive`です

### カタログの削除

#### 文法

```sql
DROP EXTERNAL CATALOG <catalog_name>;
```

#### 例

Hiveカタログの削除

```sql
DROP EXTERNAL CATALOG hive_catalog;
```

### カタログの使用

現在のカタログにはInternal CatalogとExternal Catalogの二種類があります。`SHOW CATALOGS`を使用して、現在存在するカタログを確認することができます。StarRocksには`default_catalog`というデフォルト値を持つInternal Catalogのデフォルトインスタンスが1つだけ存在します。ユーザーがmysqlクライアントからStarRocksにログインした後、現在の接続のデフォルトカタログは`default_catalog`になります。ユーザーがInternal Catalog内のOLAPテーブル機能のみを使用する場合、その使用方法は以前と変わりません。`show databases`を使用してInternal Catalogにどのようなデータベースがあるかを確認できます。
External Catalogについては、`SHOW DATABASES FROM <external_catalog_name>`を使用して存在するデータベースを確認し、`USE external_catalog.db`を使用して現在の接続のcurrent_catalogとcurrent_dbを切り替えることができます。現在はDBレベルまでのUseのみがサポートされており、CatalogレベルまでのUseはサポートされていませんが、今後順次機能が公開される予定です。

#### 例

```sql
-- default_catalogでolap_dbをcurrent databaseとしてuseする;
USE olap_db;

-- default_catalog.olap_dbでolap_tableをクエリする;
SELECT * FROM olap_table LIMIT 1;

-- default_catalog.olap_dbでexternal catalog内のテーブルをクエリする場合は、external tableの完全名を記述する必要があります。
SELECT * FROM hive_catalog.hive_db.hive_tbl;

-- current_catalogとcurrent_databaseをhive_catalogとhive_dbに切り替える
USE hive_catalog.hive_db;

-- hive_catalog.hive_dbでhive tableをクエリする
SELECT * FROM hive_table LIMIT 1;

-- hive_catalog.hive_dbでInternal Catalog内のOLAPテーブルをクエリする
SELECT * FROM default_catalog.olap_db.olap_table;

-- hive_catalog.hive_dbでInternal Catalog内のOLAPテーブルとフェデレーションクエリを実行する
SELECT * FROM hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;

-- 他のhive catalogでhive_catalogとInternal CatalogのOLAPテーブルとフェデレーションクエリを実行する
SELECT * FROM hive_catalog.hive_db.hive_tbl h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
```

### Hive Connectorのメタデータ同期

現在のHive Connectorは、ユーザーのHive Metastoreに記録されているテーブル構造やパーティションファイル情報をFEでキャッシュしています。現在の実装方法とリフレッシュ方法はHive外部テーブルと同じです。詳細は[Hive外部テーブル](../data_source/External_table.md#手動更新元数据缓存)を参照してください。
