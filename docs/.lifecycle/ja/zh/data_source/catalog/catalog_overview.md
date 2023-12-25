---
displayed_sidebar: Chinese
---

# 概要

この文書では、Catalogとは何か、およびCatalogを使用して内部および外部データを管理およびクエリする方法について説明します。

StarRocksはバージョン2.3からCatalog（データカタログ）機能をサポートし、一つのシステム内で内部データと外部データの両方を同時に維持し、様々な外部ソースに保存されているデータに簡単にアクセスしてクエリすることができます。

## 基本概念

- **内部データ**：StarRocksに保存されているデータを指します。
- **外部データ**：外部データソース（例：Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake、JDBC）に保存されているデータを指します。

## Catalog

現在のStarRocksは、internal catalogとexternal catalogの2種類のCatalogを提供しています。

![図1](../../assets/3.12-1.png)

- **Internal catalog**：内部データカタログで、StarRocks内のすべての内部データを管理します。例えば、CREATE DATABASEやCREATE TABLEステートメントで作成されたデータベースとテーブルはinternal catalogによって管理されます。各StarRocksクラスターには、[default_catalog](../catalog/default_catalog.md)という名前の1つのinternal catalogのみが存在します。
- **External catalog**：外部データカタログで、外部のmetastoreに接続するために使用されます。StarRocksでは、external catalogを通じて外部データを直接クエリし、データのインポートや移行を行う必要はありません。現在、以下のタイプのexternal catalogを作成することができます：
  - [Hive catalog](../catalog/hive_catalog.md)：Hiveデータをクエリするために使用します。
  - [Iceberg catalog](../catalog/iceberg_catalog.md)：Icebergデータをクエリするために使用します。
  - [Hudi catalog](../catalog/hudi_catalog.md)：Hudiデータをクエリするために使用します。
  - [Delta Lake catalog](../catalog/deltalake_catalog.md)：Delta Lakeデータをクエリするために使用します。
  - [JDBC catalog](../catalog/jdbc_catalog.md)：JDBCデータソースのデータをクエリするために使用します。

  external catalogを使用してデータをクエリする際、StarRocksは外部データソースの2つのコンポーネントを使用します：

  - **メタデータサービス**：StarRocksのFEがクエリプランを作成するためにメタデータを公開します。
  - **ストレージシステム**：データを保存するために使用されます。データファイルは、分散ファイルシステムまたはオブジェクトストレージシステムに異なる形式で保存されます。FEが生成したクエリプランを各BEに配布した後、各BEはHiveストレージシステム内の対象データを並列にスキャンし、計算を実行してクエリ結果を返します。

## Catalogへのアクセス

[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)を使用して、現在のセッションで有効なCatalogを切り替え、そのCatalogを通じてデータをクエリすることができます。

## データのクエリ

### 内部データのクエリ

StarRocksに保存されているデータをクエリするには、[Default catalog](../catalog/default_catalog.md)を参照してください。

### 外部データのクエリ

外部データソースに保存されているデータをクエリするには、[外部データのクエリ](../catalog/query_external_data.md)を参照してください。

### Catalog間のデータクエリ

あるCatalog内で別のCatalogのデータをクエリする場合は、`catalog_name.db_name`または`catalog_name.db_name.table_name`の形式で目的のデータを参照できます。例えば：

- `default_catalog.olap_db`で`hive_catalog`の`hive_table`をクエリします。

  ```SQL
  SELECT * FROM hive_catalog.hive_db.hive_table;
  ```

- `hive_catalog.hive_db`で`default_catalog`の`olap_table`をクエリします。

  ```SQL
  SELECT * FROM default_catalog.olap_db.olap_table;
  ```

- `hive_catalog.hive_db`で、`hive_table`と`default_catalog`の`olap_table`を結合してクエリします。

  ```SQL
  SELECT * FROM hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
  ```

- 他のCatalogで、`hive_catalog`の`hive_table`と`default_catalog`の`olap_table`を結合してクエリします。

  ```SQL
  SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
  ```
