---
displayed_sidebar: English
---

# リソースの作成

## 説明

このステートメントはリソースを作成するために使用されます。rootユーザーまたはadminユーザーのみがリソースを作成できます。現在、SparkとHiveのリソースのみがサポートされています。将来的には、クエリ用のSpark/GPU、外部ストレージ用のHDFS/S3、ETL用のMapReduceなど、StarRocksに他の外部リソースが追加される可能性があります。

構文：

```sql
CREATE [EXTERNAL] RESOURCE "resource_name"
PROPERTIES ("key"="value", ...)
```

注記：  

1. PROPERTIESはリソースタイプを指定します。現在、SparkとHiveのみがサポートされています。
2. PROPERTIESはリソースタイプに応じて異なります。詳細は例を参照してください。

## 例

1. yarnクラスターモードでspark0という名前のSparkリソースを作成します。

    ```sql
    CREATE EXTERNAL RESOURCE "spark0"
    PROPERTIES
    (
        "type" = "spark",
        "spark.master" = "yarn",
        "spark.submit.deployMode" = "cluster",
        "spark.jars" = "xxx.jar,yyy.jar",
        "spark.files" = "/tmp/aaa,/tmp/bbb",
        "spark.executor.memory" = "1g",
        "spark.yarn.queue" = "queue0",
        "spark.hadoop.yarn.resourcemanager.address" = "127.0.0.1:9999",
        "spark.hadoop.fs.defaultFS" = "hdfs://127.0.0.1:10000",
        "working_dir" = "hdfs://127.0.0.1:10000/tmp/starrocks",
        "broker" = "broker0",
        "broker.username" = "user0",
        "broker.password" = "password0"
    );
    ```

    Sparkに関連するパラメータは以下の通りです：

    ```plain text
    1. spark.master: 必須。現在、yarnとspark://host:portがサポートされています。
    2. spark.submit.deployMode: Sparkプログラムのデプロイモードは必須です。clusterとclientがサポートされています。
    3. spark.hadoop.yarn.resourcemanager.address: masterがyarnの場合に必須です。
    4. spark.hadoop.fs.defaultFS: masterがyarnの場合に必須です。
    5. その他のパラメータはオプショナルです。http://spark.apache.org/docs/latest/configuration.html を参照してください。
    ```

    SparkをETLに使用する場合、working_dirとbrokerを指定する必要があります。指示は以下の通りです：

    ```plain text
    working_dir: ETLに使用されるディレクトリ。SparkをETLリソースとして使用する場合に必須です。例：hdfs://host:port/tmp/starrocks。
    broker: ブローカーの名前。SparkをETLリソースとして使用し、`ALTER SYSTEM ADD BROKER`コマンドを使用して事前に設定する必要があります。
    broker.property_key: ETLによって作成された中間ファイルをブローカーが読み取る際に指定する必要があるプロパティ情報です。
    ```

2. hive0という名前のHiveリソースを作成します。

    ```sql
    CREATE EXTERNAL RESOURCE "hive0"
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.uris" = "thrift://10.10.44.98:9083"
    );
    ```
