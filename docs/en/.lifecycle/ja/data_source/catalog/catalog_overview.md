---
displayed_sidebar: "Japanese"
---

# 概要

このトピックでは、カタログとは何か、およびカタログを使用して内部データと外部データを管理およびクエリする方法について説明します。

StarRocksはv2.3以降、カタログ機能をサポートしています。カタログを使用すると、内部および外部のデータを1つのシステムで管理し、さまざまな外部システムに格納されているデータを簡単にクエリおよび分析する柔軟な方法を提供します。

## 基本的な概念

- **内部データ**: StarRocksに格納されているデータを指します。
- **外部データ**: Apache Hive™、Apache Iceberg、Apache Hudi、Delta Lake、JDBCなどの外部データソースに格納されているデータを指します。

## カタログ

現在、StarRocksには2種類のカタログがあります。内部カタログと外部カタログです。

![図1](../../assets/3.8.1.png)

- **内部カタログ**は、StarRocksの内部データを管理します。たとえば、CREATE DATABASEまたはCREATE TABLEステートメントを実行してデータベースまたはテーブルを作成すると、データベースまたはテーブルは内部カタログに格納されます。各StarRocksクラスタには、[default_catalog](../catalog/default_catalog.md)という名前の内部カタログが1つだけあります。

- **外部カタログ**は、外部で管理されているメタストアへのリンクのような役割を果たし、StarRocksに外部データソースへの直接アクセスを提供します。データのロードや移行なしで外部データを直接クエリできます。現在、StarRocksは次のタイプの外部カタログをサポートしています：
  - [Hiveカタログ](../catalog/hive_catalog.md)：Hiveからデータをクエリするために使用します。
  - [Icebergカタログ](../catalog/iceberg_catalog.md)：Icebergからデータをクエリするために使用します。
  - [Hudiカタログ](../catalog/hudi_catalog.md)：Hudiからデータをクエリするために使用します。
  - [Delta Lakeカタログ](../catalog/deltalake_catalog.md)：Delta Lakeからデータをクエリするために使用します。
  - [JDBCカタログ](../catalog/jdbc_catalog.md)：JDBC互換のデータソースからデータをクエリするために使用します。

  StarRocksは、外部データをクエリする際に次の2つの外部データソースのコンポーネントと対話します：

  - **メタストアサービス**：FE（Frontend）が外部データソースのメタデータにアクセスするために使用します。FEはメタデータに基づいてクエリ実行計画を生成します。
  - **データストレージシステム**：外部データを格納するために使用します。分散ファイルシステムとオブジェクトストレージシステムの両方をデータストレージシステムとして使用して、さまざまな形式のデータファイルを格納できます。FEがクエリ実行計画をすべてのBE（Backend）に配布した後、すべてのBEは対象の外部データを並列にスキャンし、計算を実行し、クエリ結果を返します。

## カタログへのアクセス

[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)ステートメントを使用して、現在のセッションで指定されたカタログに切り替えることができます。その後、そのカタログを使用してデータをクエリできます。

## データのクエリ

### 内部データのクエリ

StarRocksでデータをクエリするには、[デフォルトカタログ](../catalog/default_catalog.md)を参照してください。

### 外部データのクエリ

外部データソースからデータをクエリするには、[外部データのクエリ](../catalog/query_external_data.md)を参照してください。

### カタログ間のクエリ

現在のカタログからクロスカタログのフェデレーテッドクエリを実行するには、`catalog_name.database_name`または`catalog_name.database_name.table_name`の形式でクエリしたいデータを指定します。

- 現在のセッションが`default_catalog.olap_db`の場合、`hive_db`の`hive_table`をクエリします。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`の場合、`default_catalog`の`olap_table`をクエリします。

   ```SQL
    SELECT * FROM default_catalog.olap_db.olap_table;
    ```

- 現在のセッションが`hive_catalog.hive_db`の場合、`hive_catalog`の`hive_table`と`default_catalog`の`olap_table`をJOINクエリします。

    ```SQL
    SELECT * FROM hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```

- 現在のセッションが別のカタログの場合、`hive_catalog`の`hive_db`の`hive_table`と`default_catalog`の`olap_db`の`olap_table`をJOIN句を使用してクエリします。

    ```SQL
    SELECT * FROM hive_catalog.hive_db.hive_table h JOIN default_catalog.olap_db.olap_table o WHERE h.id = o.id;
    ```
