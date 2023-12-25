---
displayed_sidebar: English
---

# 概要

このトピックでは、カタログとは何か、およびカタログを使用して内部データと外部データを管理およびクエリする方法について説明します。

StarRocksはv2.3以降、カタログ機能をサポートしています。カタログを使用すると、内部データと外部データを一つのシステムで管理でき、さまざまな外部システムに保存されているデータを簡単にクエリし、分析するための柔軟な方法を提供します。

## 基本概念

- **内部データ**: StarRocksに保存されているデータを指します。
- **外部データ**: Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake、JDBCなどの外部データソースに格納されているデータを指します。

## カタログ

現在、StarRocksは内部カタログと外部カタログの2種類のカタログを提供しています。

![図1](../../assets/3.8.1.png)

- **内部カタログ**はStarRocksの内部データを管理します。たとえば、CREATE DATABASEやCREATE TABLEステートメントを実行してデータベースやテーブルを作成すると、そのデータベースやテーブルは内部カタログに格納されます。各StarRocksクラスターには、[default_catalog](../catalog/default_catalog.md)という名前の1つの内部カタログがあります。

- **外部カタログ**は、外部で管理されているメタストアへのリンクとして機能し、StarRocksに外部データソースへの直接アクセスを可能にします。データのロードや移行なしに、外部データを直接クエリできます。現在、StarRocksは以下のタイプの外部カタログをサポートしています:
  - [Hiveカタログ](../catalog/hive_catalog.md): Hiveからデータをクエリするために使用されます。
  - [Icebergカタログ](../catalog/iceberg_catalog.md): Icebergからデータをクエリするために使用されます。
  - [Hudiカタログ](../catalog/hudi_catalog.md): Hudiからデータをクエリするために使用されます。
  - [Delta Lakeカタログ](../catalog/deltalake_catalog.md): Delta Lakeからデータをクエリするために使用されます。
  - [JDBCカタログ](../catalog/jdbc_catalog.md): JDBC互換のデータソースからデータをクエリするために使用されます。

  StarRocksは、外部データをクエリする際に、外部データソースの以下の2つのコンポーネントと対話します:

  - **メタストアサービス**: FEが外部データソースのメタデータにアクセスするために使用します。FEはメタデータに基づいてクエリ実行プランを生成します。
  - **データストレージシステム**: 外部データを保存するために使用されます。分散ファイルシステムとオブジェクトストレージシステムの両方が、様々な形式のデータファイルを保存するためのデータストレージシステムとして使用できます。FEがクエリ実行プランをすべてのBEに配布すると、すべてのBEが対象の外部データを並行してスキャンし、計算を行い、その後クエリ結果を返します。

## カタログへのアクセス

[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) ステートメントを使用して、現在のセッションで特定のカタログに切り替えることができます。その後、そのカタログを使用してデータをクエリできます。

## データのクエリ

### 内部データのクエリ

StarRocks内のデータをクエリするには、[Default catalog](../catalog/default_catalog.md)を参照してください。

### 外部データのクエリ

外部データソースからデータをクエリするには、[Query external data](../catalog/query_external_data.md)を参照してください。

### カタログ間クエリ

現在のカタログからカタログ間連合クエリを実行するには、`catalog_name.database_name`または`catalog_name.database_name.table_name`の形式でクエリしたいデータを指定します。

- 現在のセッションが`default_catalog.olap_db`で、`hive_db`内の`hive_table`をクエリする場合:

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`で、`default_catalog`内の`olap_table`をクエリする場合:

   ```SQL
    SELECT * FROM default_catalog.olap_db.olap_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`で、`hive_catalog`の`hive_table`と`default_catalog`の`olap_table`に対してJOINクエリを実行する場合:

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```

- 現在のセッションが別のカタログで、`hive_catalog`の`hive_table`と`default_catalog`の`olap_table`に対してJOIN句を使用してJOINクエリを実行する場合:

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```
