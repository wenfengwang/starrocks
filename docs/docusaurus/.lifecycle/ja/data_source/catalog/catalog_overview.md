---
displayed_sidebar: "Japanese"
---

# 概要

このトピックでは、カタログとは何か、カタログを使用して内部データおよび外部データを管理およびクエリする方法について説明します。

StarRocksはv2.3からカタログ機能をサポートしています。カタログを使用すると、内部および外部のデータを1つのシステムで管理し、さまざまな外部システムに格納されたデータを簡単にクエリおよび分析できる柔軟な方法を提供します。

## 基本的な概念

- **内部データ**: StarRocksに格納されたデータを指します。
- **外部データ**: Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake、およびJDBCなどの外部データソースに格納されたデータを指します。

## カタログ

現在、StarRocksは内部カタログと外部カタログの2種類のカタログを提供しています。

![図1](../../assets/3.8.1.png)

- **内部カタログ** はStarRocksの内部データを管理します。たとえば、CREATE DATABASEまたはCREATE TABLEステートメントを実行してデータベースまたはテーブルを作成すると、そのデータベースやテーブルは内部カタログに格納されます。各StarRocksクラスタには、[default_catalog](../catalog/default_catalog.md)という名前の内部カタログが1つだけあります。

- **外部カタログ** は外部で管理されているメタストアへのリンクのような役割を果たし、StarRocksに外部データソースへの直接アクセスを提供します。データのロードや移行なしに外部データを直接クエリすることができます。現在、StarRocksは次のタイプの外部カタログをサポートしています。
  - [Hiveカタログ](../catalog/hive_catalog.md): Hiveからデータをクエリするために使用します。
  - [Icebergカタログ](../catalog/iceberg_catalog.md): Icebergからデータをクエリするために使用します。
  - [Hudiカタログ](../catalog/hudi_catalog.md): Hudiからデータをクエリするために使用します。
  - [Delta Lakeカタログ](../catalog/deltalake_catalog.md): Delta Lakeからデータをクエリするために使用します。
  - [JDBCカタログ](../catalog/jdbc_catalog.md): JDBC互換のデータソースからデータをクエリするために使用します。

  StarRocksは外部データをクエリする際に、次の2つの外部データソースのコンポーネントとやり取りします。

  - **メタストアサービス**: FEsが外部データソースのメタデータにアクセスするために使用します。FEはメタデータに基づいてクエリ実行計画を生成します。
  - **データストレージシステム**: 外部データを格納するために使用されます。分散ファイルシステムやオブジェクトストレージシステムなど、さまざまな形式のデータファイルを格納するためにデータストレージシステムを使用できます。FEがクエリ実行計画をすべてのBEに配布した後、すべてのBEは対象の外部データを並列でスキャンし、計算を実行し、その後クエリ結果を返します。

## カタログへのアクセス

[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)ステートメントを使用して、現在のセッションで指定されたカタログに切り替えることができます。その後、そのカタログを使用してデータをクエリすることができます。

## データのクエリ

### 内部データのクエリ

StarRocksでデータをクエリする方法については、[デフォルトカタログ](../catalog/default_catalog.md)を参照してください。

### 外部データのクエリ

外部データソースからデータをクエリする方法については、[外部データのクエリ](../catalog/query_external_data.md)を参照してください。

### クロスカタログクエリ

現在のカタログからクロスカタログ連携クエリを実行するには、`catalog_name.database_name`または`catalog_name.database_name.table_name`の形式でクエリしたいデータを指定します。

- 現在のセッションが`default_catalog.olap_db`の場合に`hive_db`の`hive_table`をクエリする。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`の場合に、`default_catalog`の`olap_db`で`olap_table`をクエリする。

   ```SQL
    SELECT * FROM default_catalog.olap_db.olap_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`の場合に、`hive_catalog`の`hive_table`と`default_catalog`の`olap_table`でJOINクエリを実行する。

    ```SQL
    SELECT * FROM hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```

- 他のカタログで現在のセッションが`hive_catalog.hive_db`の場合に、`hive_catalog`の`hive_table`と`default_catalog`の`olap_table`でJOIN句を使用してJOINクエリを実行する。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```