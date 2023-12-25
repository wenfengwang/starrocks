---
displayed_sidebar: Chinese
---

# INSERT INTO FILES を使用してデータをエクスポートする

この記事では、INSERT INTO FILES を使用して StarRocks データをリモートストレージにエクスポートする方法について説明します。

バージョン v3.2 から、StarRocks はテーブル関数 FILES() を使用してリモートストレージに書き込み可能（Writable）なファイルを定義することをサポートしています。FILES() を INSERT ステートメントと組み合わせることで、StarRocks からデータをリモートストレージにエクスポートできます。

StarRocks がサポートする他のデータエクスポート方法と比較して、INSERT INTO FILES を使用したエクスポートは、より統一された使いやすいインターフェースを提供します。データのインポートと同じ構文を使用して、直接リモートストレージにデータをエクスポートできます。さらに、この方法は指定された列の値を抽出してデータファイルを異なるストレージパスに保存することをサポートし、エクスポートされたデータをさらにパーティション分けすることを可能にします。

> **説明**
>
> - INSERT INTO FILES を使用したデータのエクスポートは、ローカルファイルシステムへのエクスポートはサポートしていません。
> - 現在、INSERT INTO FILES は Parquet ファイル形式でのデータエクスポートのみをサポートしています。

## 準備

以下の例では、`unload` という名前のデータベースと `sales_records` という名前のテーブルを作成し、以下のチュートリアルで使用するデータオブジェクトとしています。代わりに自分のデータを使用することもできます。

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

`sales_records` テーブルには、各取引の取引 ID `record_id`、販売員 `seller`、店舗 ID `store_id`、時間 `sales_time`、および売上高 `sales_amt` が含まれています。このテーブルは `sales_time` に基づいて日単位でパーティション分けされています。

さらに、書き込み権限のあるリモートストレージシステムを準備する必要があります。以下の例では、シンプル認証が有効になっている HDFS クラスターを使用しています。サポートされているリモートストレージシステムと認証方法については、[SQL 参照 - FILES()](../sql-reference/sql-functions/table-functions/files.md) を参照してください。

## データのエクスポート

INSERT INTO FILES は、データを単一のファイルまたは複数のファイルにエクスポートすることをサポートしています。これらのファイルに異なるストレージパスを指定することで、さらにパーティション分けを行うことができます。

INSERT INTO FILES を使用してデータをエクスポートする際には、`compression` 属性を設定して手動で圧縮アルゴリズムを設定する必要があります。StarRocks がサポートするデータ圧縮アルゴリズムについては、[データ圧縮](../table_design/data_compression.md) を参照してください。

### 複数のファイルにデータをエクスポートする

デフォルトでは、INSERT INTO FILES はデータを複数のデータファイルにエクスポートし、各ファイルのサイズは 1 GB です。`max_file_size` 属性を使用してファイルサイズを設定できます。

以下の例では、`sales_records` のすべてのデータ行を `data1` というプレフィックスを持つ複数の Parquet ファイルとしてエクスポートします。各ファイルのサイズは 1 KB です。

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

### 異なるパスにある複数のファイルにデータをエクスポートする

`partition_by` 属性を使用して指定された列の値を抽出し、データファイルを異なるパスにそれぞれ保存することもできます。

以下の例では、`sales_records` のすべてのデータ行を HDFS クラスターの **/unload/partitioned/** パスにある複数の Parquet ファイルとしてエクスポートします。これらのファイルは `sales_time` 列の値に基づいて異なるサブパスに保存されます。

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

### 単一のファイルにデータをエクスポートする

データを単一のデータファイルにエクスポートするには、`single` 属性を `true` に設定する必要があります。

以下の例では、`sales_records` のすべてのデータ行を `data2` というプレフィックスを持つ単一の Parquet ファイルとしてエクスポートします。

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

- INSERT の使用に関する詳細は、[SQL 参照 - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。
- FILES() の使用に関する詳細は、[SQL 参照 - FILES()](../sql-reference/sql-functions/table-functions/files.md) を参照してください。
