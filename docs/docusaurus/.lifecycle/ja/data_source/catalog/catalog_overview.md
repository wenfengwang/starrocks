---
displayed_sidebar: "Japanese"
---

# 概要

このトピックでは、カタログとは何か、およびカタログを使用して内部データと外部データを管理およびクエリする方法について説明します。

StarRocksはv2.3以降でカタログ機能をサポートしています。カタログを使用すると、1つのシステムで内部および外部データを管理し、さまざまな外部システムに格納されているデータを簡単にクエリおよび解析できる柔軟な方法を提供します。

## 基本概念

- **内部データ**: StarRocksに格納されているデータを指します。
- **外部データ**: Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake、およびJDBCなどの外部データソースに格納されているデータを指します。

## カタログ

現在、StarRocksには内部カタログと外部カタログの2種類のカタログがあります。

![図1](../../assets/3.8.1.png)

- **内部カタログ**はStarRocksの内部データを管理します。たとえば、CREATE DATABASEまたはCREATE TABLEステートメントを実行してデータベースまたはテーブルを作成する場合、そのデータベースまたはテーブルは内部カタログに格納されます。各StarRocksクラスタには、[default_catalog](../catalog/default_catalog.md)という名前の内部カタログが1つだけあります。

- **外部カタログ**は外部で管理されたメタストアへのリンクのような役割を果たし、StarRocksに外部データソースへの直接アクセス権を付与します。データのロードや移行を行わずに外部データを直接クエリできます。現在、StarRocksは次の種類の外部カタログをサポートしています。
  - [Hiveカタログ](../catalog/hive_catalog.md): Hiveからデータをクエリするために使用されます。
  - [Icebergカタログ](../catalog/iceberg_catalog.md): Icebergからデータをクエリするために使用されます。
  - [Hudiカタログ](../catalog/hudi_catalog.md): Hudiからデータをクエリするために使用されます。
  - [Delta Lakeカタログ](../catalog/deltalake_catalog.md): Delta Lakeからデータをクエリするために使用されます。
  - [JDBCカタログ](../catalog/jdbc_catalog.md): JDBC互換のデータソースからデータをクエリするために使用されます。

  外部データをクエリする際に、StarRocksは次の2つの外部データソースのコンポーネントとやり取りします。

  - **メタストアサービス**: FEsが外部データソースのメタデータにアクセスするために使用されます。FEsはメタデータに基づいてクエリ実行計画を生成します。
  - **データストレージシステム**: 外部データを格納するために使用されます。分散ファイルシステムおよびオブジェクトストレージシステムの両方を使用して、さまざまな形式のデータファイルを格納できます。FEsがクエリ実行計画をすべてのBEに配布した後、すべてのBEが対象の外部データを並列でスキャンし、計算を実行し、その後クエリ結果を返します。

## カタログへのアクセス

[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)ステートメントを使用して、現在のセッションで指定されたカタログに切り替えることができます。その後、そのカタログを使用してデータをクエリすることができます。

## データのクエリ

### 内部データのクエリ

StarRocksでデータをクエリするには、[Default catalog](../catalog/default_catalog.md)を参照してください。

### 外部データのクエリ

外部データソースからデータをクエリするには、[Query external data](../catalog/query_external_data.md)を参照してください。

### クロスカタログクエリ

現在のカタログからクロスカタログ連携クエリを実行するには、`catalog_name.database_name`または`catalog_name.database_name.table_name`の形式でクエリしたいデータを指定します。

- 現在のセッションが`default_catalog.olap_db`の場合に、`hive_db`の`hive_table`をクエリします。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`の場合に、`default_catalog`の`olap_table`をクエリします。

   ```SQL
    SELECT * FROM default_catalog.olap_db.olap_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`の場合に、`hive_catalog`の`hive_table`と`default_catalog`の`olap_table`をJOINクエリします。

    ```SQL
    SELECT * FROM hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```

- 他のカタログの場合に、`hive_catalog.hive_db`の`hive_table`と`default_catalog.olap_db`の`olap_table`をJOIN句を使用してJOINクエリします。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```