---
displayed_sidebar: "Japanese"
---

# INSERT INTO FILESを使用してデータをアンロードする

このトピックでは、INSERT INTO FILESを使用してStarRocksからリモートストレージへデータをアンロードする方法について説明します。

v3.2以降、StarRocksは表関数FILES（）を使用してリモートストレージの書き込み可能なファイルを定義することをサポートしています。その後、FILES（）をINSERT文と組み合わせて、StarRocksからデータをリモートストレージにアンロードすることができます。

StarRocksがサポートする他のデータエクスポート方法と比較して、INSERT INTO FILESを使用してデータをアンロードすると、より統一された使いやすいインターフェースが提供されます。また、この方法では指定した列の値を抽出してデータファイルを異なるストレージパスに保存することができるため、パーティションレイアウトでエクスポートデータを管理することができます。

> **注意**
>
> - INSERT INTO FILESを使用してデータをアンロードする場合、ローカルファイルシステムにデータをエクスポートすることはサポートされていません。
> - 現在、INSERT INTO FILESはParquetファイル形式でのデータのアンロードのみをサポートしています。

## 準備

次の例では、「unload」というデータベースと「sales_records」というテーブルを作成し、これらのデータオブジェクトを以下のチュートリアルで使用できるようにしています。お使いのデータを使用することもできます。

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

テーブル「sales_records」には、各トランザクションの取引ID「record_id」、販売員「seller」、店舗ID「store_id」、時間「sales_time」、販売金額「sales_amt」が含まれています。「sales_time」に従って、日次でパーティションが作成されています。

また、書き込みアクセス権を持つリモートストレージシステムを準備する必要があります。次の例では、シンプルな認証方法を有効にしたHDFSクラスターを使用しています。サポートされるリモートストレージシステムや認証メソッドについて詳しくは、[SQLリファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md)を参照してください。

## データをアンロードする

INSERT INTO FILESでは、データを単一のファイルまたは複数のファイルにアンロードすることができます。さらに、これらのデータファイルを別々のストレージパスにパーティション分割することもできます。

INSERT INTO FILESを使用してデータをアンロードする際には、プロパティ「compression」を手動で設定する必要があります。StarRocksでサポートされているデータ圧縮アルゴリズムについては、[データ圧縮](../table_design/data_compression.md)を参照してください。

### 複数のファイルにデータをアンロード

デフォルトでは、INSERT INTO FILESはデータを1 GBのサイズの複数のデータファイルにアンロードします。プロパティ「max_file_size」を使用してファイルサイズを設定することができます。

次の例では、`sales_records`のすべてのデータ行を、「data1」で始まる複数のParquetファイルとしてアンロードします。各ファイルのサイズは1 KBです。

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

### 異なるパスに複数のファイルをアンロード

プロパティ「partition_by」を使用して、指定した列の値を抽出して異なるストレージパスにデータファイルをパーティション分割することができます。

次の例では、`sales_records`のすべてのデータ行をHDFSクラスターのパス**/unload/partitioned/**の下に複数のParquetファイルとしてアンロードします。これらのファイルは`sales_time`の値で区別された異なるサブパスに保存されます。

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

### 単一のファイルにデータをアンロード

データを単一のデータファイルにアンロードする場合は、プロパティ「single」を「true」に指定する必要があります。

次の例では、`sales_records`のすべてのデータ行を、「data2」で始まる単一のParquetファイルとしてアンロードします。

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

- INSERTの使用方法についての詳細は、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。
- FILES()の使用方法についての詳細は、[SQLリファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md)を参照してください。