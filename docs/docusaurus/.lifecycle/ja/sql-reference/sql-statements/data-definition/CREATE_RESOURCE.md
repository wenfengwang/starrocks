---
displayed_sidebar: "Japanese"
---

# リソースの作成

## 説明

このステートメントはリソースを作成するために使用されます。リソースは、ユーザールートまたは管理者のみ作成できます。現在、SparkおよびHiveリソースのみがサポートされています。将来的には、Spark/GPUクエリ用、外部ストレージとしてのHDFS/S3、ETL用のMapReduceなど、その他の外部リソースがStarRocksに追加されるかもしれません。

構文：

```sql
CREATE [EXTERNAL] RESOURCE "resource_name"
PROPERTIES ("key"="value", ...)
```

注：

1. PROPERTIESはリソースの種類を指定します。現在、SparkとHiveのみがサポートされています。
2. PROPERTIESはリソースの種類によって異なります。詳細については例を参照してください。

## 例

1. yarn クラスターモードで名前が spark0 の Spark リソースを作成します。

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
    1. spark.master: 必須。現在、yarnとspark ://host:portのみがサポートされています。
    2. spark.submit.deployMode: Sparkプログラムの展開モードが必要です。clusterおよびclientをサポートしています。
    3. spark.hadoop.yarn.resourcemanager.address: masterがyarnの場合に必要です。
    4. spark.hadoop.fs.defaultFS: masterがyarnの場合に必要です。
    5. その他のパラメータはオプションです。詳細はhttp://spark.apache.org/docs/latest/configuration.htmlを参照してください。
    ```

    SparkをETL用に使用する場合は、working_DIRとbrokerを指定する必要があります。手順は次のとおりです：

    ```plain text
    working_dir: ETLで使用されるディレクトリ。SparkをETLリソースとして使用する場合に必要です。例：hdfs://host:port/tmp/starrocks。
    broker: ブローカーの名前。SparkをETLリソースとして使用し、事前に`ALTER SYSTEM ADD BROKER`コマンドを使用して構成する必要があります。
    broker.property_key: ETLによって作成された中間ファイルをブローカーが読み取る際に指定する必要があるプロパティ情報です。
    ```

2. 名前が hive0 の Hive リソースを作成します。

    ```sql
    CREATE EXTERNAL RESOURCE "hive0"
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.uris" = "thrift://10.10.44.98:9083"
    );
    ```