---
displayed_sidebar: "Japanese"
---

# INSERT INTO FILESを使用してデータをアンロードする

このトピックでは、INSERT INTO FILESを使用してStarRocksからリモートストレージにデータをアンロードする方法について説明します。

v3.2以降、StarRocksでは、テーブル関数FILES()を使用してリモートストレージに書き込み可能なファイルを定義することがサポートされています。その後、FILES()をINSERT文と組み合わせて使用することで、StarRocksからリモートストレージにデータをアンロードすることができます。

StarRocksがサポートする他のデータエクスポート方法と比較して、INSERT INTO FILESを使用したデータのアンロードはより統一された使いやすいインターフェースを提供します。データをアンロードする際には、データをロードする際に使用した構文と同じ構文を使用してデータを直接リモートストレージにアンロードすることができます。さらに、この方法では指定した列の値を抽出してデータファイルを異なるストレージパスに保存することができるため、エクスポートされたデータをパーティション化されたレイアウトで管理することができます。

> **注意**
>
> - INSERT INTO FILESを使用したデータのアンロードでは、データをローカルファイルシステムにエクスポートすることはサポートされていません。
> - 現在、INSERT INTO FILESはParquetファイル形式でのデータのアンロードのみをサポートしています。

## 準備

以下の例では、以下のチュートリアルで使用できるデータオブジェクトとして、データベース`unload`とテーブル`sales_records`を作成します。ご自身のデータを使用することもできます。

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

テーブル`sales_records`には、各トランザクションの取引ID `record_id`、販売員 `seller`、店舗ID `store_id`、時間 `sales_time`、販売金額 `sales_amt`が含まれています。`sales_time`に従って日単位でパーティション化されています。

また、書き込みアクセス権限を持つリモートストレージシステムを準備する必要があります。以下の例では、シンプルな認証方法が有効になっているHDFSクラスターを使用しています。サポートされているリモートストレージシステムと認証方法の詳細については、[SQLリファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md)を参照してください。

## データのアンロード

INSERT INTO FILESは、単一のファイルまたは複数のファイルにデータをアンロードすることができます。これらのデータファイルを別々のストレージパスにパーティション化することもできます。

INSERT INTO FILESを使用してデータをアンロードする際には、プロパティ`compression`を使用して圧縮アルゴリズムを手動で設定する必要があります。StarRocksがサポートするデータの圧縮アルゴリズムの詳細については、[データの圧縮](../table_design/data_compression.md)を参照してください。

### 複数のファイルにデータをアンロードする

デフォルトでは、INSERT INTO FILESはデータを1 GBのサイズの複数のデータファイルにアンロードします。ファイルサイズはプロパティ`max_file_size`を使用して設定することができます。

以下の例では、`sales_records`のすべてのデータ行を、`data1`で始まる複数のParquetファイルにアンロードします。各ファイルのサイズは1 KBです。

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

### 異なるパスに複数のファイルにデータをアンロードする

プロパティ`partition_by`を使用して、指定した列の値を抽出して異なるストレージパスにデータファイルをパーティション化することもできます。

以下の例では、`sales_records`のすべてのデータ行を、HDFSクラスターのパス**/unload/partitioned/**の下にある複数のParquetファイルにアンロードします。これらのファイルは、列`sales_time`の値によって区別される異なるサブパスに格納されます。

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

### 単一のファイルにデータをアンロードする

データを単一のデータファイルにアンロードするには、プロパティ`single`を`true`として指定する必要があります。

以下の例では、`sales_records`のすべてのデータ行を、`data2`で始まる単一のParquetファイルにアンロードします。

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

## 関連情報

- INSERTの使用方法についての詳細な手順については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。
- FILES()の使用方法についての詳細な手順については、[SQLリファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md)を参照してください。
