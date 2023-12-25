---
displayed_sidebar: English
---

# INSERT INTO FILES を使用したデータのアンロード

このトピックでは、INSERT INTO FILES を使用して StarRocks からリモートストレージへデータをアンロードする方法について説明します。

v3.2 以降、StarRocks はテーブル関数 FILES() を使用してリモートストレージに書き込み可能なファイルを定義することをサポートしています。FILES() と INSERT ステートメントを組み合わせることで、StarRocks からリモートストレージへデータをアンロードできます。

StarRocks がサポートする他のデータエクスポート方法と比較して、INSERT INTO FILES によるデータのアンロードは、より統一された使いやすいインターフェースを提供します。リモートストレージからデータをロードする際に使用したのと同じ構文でデータを直接リモートストレージにアンロードできます。さらに、この方法は指定された列の値を抽出して異なるストレージパスにデータファイルを格納することをサポートし、エクスポートされたデータをパーティション化されたレイアウトで管理することができます。

> **注記**
>
> - INSERT INTO FILES によるデータのアンロードは、ローカルファイルシステムへのデータエクスポートをサポートしていません。
> - 現在、INSERT INTO FILES は Parquet ファイル形式でのデータアンロードのみをサポートしています。

## 準備

以下の例では、データベース `unload` とテーブル `sales_records` を作成し、後続のチュートリアルで使用できるデータオブジェクトとしています。代わりにご自身のデータを使用しても構いません。

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

テーブル `sales_records` には、各トランザクションのトランザクションID `record_id`、販売員 `seller`、店舗ID `store_id`、時間 `sales_time`、および売上金額 `sales_amt` が含まれています。これは `sales_time` に従って日次でパーティション分割されています。

書き込みアクセス権を持つリモートストレージシステムの準備も必要です。次の例では、シンプル認証方式を有効にした HDFS クラスターを使用しています。サポートされているリモートストレージシステムおよび認証方式の詳細については、[SQL リファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md) を参照してください。

## データのアンロード

INSERT INTO FILES は、単一のファイルまたは複数のファイルへのデータアンロードをサポートしています。これらのデータファイルを個別のストレージパスに分割することもできます。

INSERT INTO FILES を使用してデータをアンロードする際には、プロパティ `compression` を使用して圧縮アルゴリズムを手動で設定する必要があります。StarRocks がサポートするデータ圧縮アルゴリズムの詳細については、[データ圧縮](../table_design/data_compression.md) を参照してください。

### 複数のファイルにデータをアンロードする

デフォルトでは、INSERT INTO FILES は、それぞれ 1 GB のサイズを持つ複数のデータファイルにデータをアンロードします。ファイルサイズは、プロパティ `max_file_size` を使用して設定できます。

次の例では、`sales_records` のすべてのデータ行を `data1` というプレフィックスが付いた複数の Parquet ファイルとしてアンロードします。各ファイルのサイズは 1 KB です。

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

### 異なるパス下の複数のファイルにデータをアンロードする

プロパティ `partition_by` を使用して指定した列の値を抽出し、異なるストレージパスにデータファイルを分割することもできます。

次の例では、HDFS クラスターの **/unload/partitioned/** パス下にある複数の Parquet ファイルとして `sales_records` のすべてのデータ行をアンロードします。これらのファイルは、`sales_time` 列の値によって区別される異なるサブパスに格納されます。

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

### データを単一のファイルにアンロードする

データを単一のデータファイルにアンロードするには、プロパティ `single` を `true` として指定する必要があります。

次の例では、`sales_records` のすべてのデータ行を `data2` というプレフィックスが付いた単一の Parquet ファイルとしてアンロードします。

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

- INSERT の使用法についての詳細は、[SQL リファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。
- FILES() の使用法についての詳細は、[SQL リファレンス - FILES()](../sql-reference/sql-functions/table-functions/files.md) を参照してください。
