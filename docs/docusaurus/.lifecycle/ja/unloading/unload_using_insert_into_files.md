---
displayed_sidebar: "Japanese"
---

# INSERT INTO FILESを使用してデータをアンロードする

このトピックでは、INSERT INTO FILESを使用してStarRocksからリモートストレージにデータをアンロードする方法について説明します。

v3.2以降、StarRocksはFILES()テーブル関数を使用してリモートストレージに書き込み可能なファイルを定義することをサポートしています。その後、INSERTステートメントとFILES()を組み合わせて、StarRocksからデータをリモートストレージにアンロードすることができます。

StarRocksがサポートする他のデータエクスポート方法と比較して、INSERT INTO FILESを使用してデータをアンロードすると、統一された使いやすいインターフェースを提供します。データをロードした際と同じ構文を使用してデータを直接リモートストレージにアンロードできます。さらに、この方法では指定した列の値を抽出してデータファイルを異なるストレージパスに格納することができるため、エクスポートされたデータをパーティションレイアウトで管理することができます。

> **注記**
>
> - INSERT INTO FILESを使用してデータをアンロードする際、ローカルファイルシステムにデータをエクスポートすることはサポートされていません。
> - 現時点では、INSERT INTO FILESはParquetファイル形式でのデータアンロードのみをサポートしています。

## 準備

次の例では、以下のチュートリアルで使用できるデータオブジェクトとしてデータベース`unload`とテーブル`sales_records`を作成します。独自のデータを使用することもできます。

```SQL
CREATE DATABASE unload;
USE unload;
CREATE TABLE sales_records(
    record_id     BIGINT,
    seller        STRING,
    store_id      INT,
    sales_time    DATETIME,
    sales_amt     DOUBLE
)
DUPLICATE KEY(record_id)
PARTITION BY date_trunc('day', sales_time)
DISTRIBUTED BY HASH(record_id);

INSERT INTO sales_records
VALUES
    (220313001,"Amy",1,"2022-03-13 12:00:00",8573.25),
    (220314002,"Bob",2,"2022-03-14 12:00:00",6948.99),
    (220314003,"Amy",1,"2022-03-14 12:00:00",4319.01),
    (220315004,"Carl",3,"2022-03-15 12:00:00",8734.26),
    (220316005,"Carl",3,"2022-03-16 12:00:00",4212.69),
    (220317006,"Bob",2,"2022-03-17 12:00:00",9515.88);
```

テーブル`sales_records`には、各トランザクションのトランザクションID`record_id`、販売者`seller`、店舗ID`store_id`、時間`sales_time`、販売額`sales_amt`が含まれています。`sales_time`に従って日次でパーティション化されています。

また、書き込みアクセス権を持つリモートストレージシステムを準備する必要があります。次の例では、シンプルな認証方法が有効になっているHDFSクラスターが使用されています。サポートされているリモートストレージシステムおよび認証方法については、[SQLリファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md)を参照してください。

## データをアンロードする

INSERT INTO FILESを使用すると、データを単一ファイルまたは複数ファイルにアンロードすることができます。さらに、これらのデータファイルを別々のストレージパスに分割することで、データをさらにパーティション化することができます。

INSERT INTO FILESを使用してデータをアンロードする際には、プロパティ`compression`を手動で設定する必要があります。StarRocksがサポートするデータ圧縮アルゴリズムについての詳細は、[データ圧縮](../table_design/data_compression.md)を参照してください。

### 複数ファイルにデータをアンロード

デフォルトでは、INSERT INTO FILESはデータを1 GBのサイズの複数のデータファイルにアンロードします。ファイルサイズはプロパティ`max_file_size`を使用して構成することができます。

次の例では、`sales_records`のすべてのデータ行を`data1`でプレフィックス付けた複数のParquetファイルとしてアンロードします。各ファイルのサイズは1 KBです。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/data1",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "max_file_size" = "1KB"
)
SELECT * FROM sales_records;
```

### 異なるパスに複数のファイルにデータをアンロード

プロパティ`partition_by`を使用して、指定した列の値を抽出して異なるストレージパスにデータファイルをパーティション化することもできます。

次の例では、`sales_records`のすべてのデータ行をHDFSクラスター内の**/unload/partitioned/**パスに配置された複数のParquetファイルとしてアンロードします。これらのファイルは、列`sales_time`の値によって識別される異なるサブパスに保存されます。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/partitioned/",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "partition_by" = "sales_time"
)
SELECT * FROM sales_records;
```

### 単一ファイルにデータをアンロード

データを単一のデータファイルにアンロードするには、プロパティ`single`を`true`として指定する必要があります。

次の例では、`sales_records`のすべてのデータ行を`data2`でプレフィックス付けた単一のParquetファイルとしてアンロードします。

```SQL
INSERT INTO 
FILES(
    "path" = "hdfs://xxx.xx.xxx.xx:9000/unload/data2",
    "format" = "parquet",
    "hadoop.security.authentication" = "simple",
    "username" = "xxxxx",
    "password" = "xxxxx",
    "compression" = "lz4",
    "single" = "true"
)
SELECT * FROM sales_records;
```

## 関連項目

- INSERTの使用方法についての詳細は、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。
- FILES()の使用方法についての詳細は、[SQLリファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md)を参照してください。