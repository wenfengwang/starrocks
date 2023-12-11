---
displayed_sidebar: "Japanese"
---

# リソースの作成

## 説明

このステートメントはリソースを作成するために使用されます。ユーザーrootまたは管理者のみがリソースを作成できます。現在、StarRocksはSparkとHiveリソースのみをサポートしています。将来的には、クエリ用のSpark/GPU、外部ストレージ用のHDFS/S3、ETL用のMapReduceなど、その他の外部リソースがStarRocksに追加されるかもしれません。

構文:

```sql
CREATE [EXTERNAL] RESOURCE "resource_name"
PROPERTIES ("key"="value", ...)
```

注意:  

1. PROPERTIESではリソースの種類が指定されます。現在、SparkとHiveのみがサポートされています。
2. PROPERTIESはリソースの種類によって異なります。詳細は例を参照してください。

## 例

1. ヤーンクラスターモードでspark0という名前のSparkリソースを作成します。

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

    Spark関連のパラメータは以下の通りです:

    ```plain text
    1. spark.master: 必須。現在、yarnとspark ://host:portをサポートしています。
    2. spark.submit.deployMode: Sparkプログラムのデプロイモードが必要です。clusterとclientをサポートしています。
    3. spark.hadoop.yarn.resourcemanager.address: マスターがyarnの場合に必要です。
    4. spark.hadoop.fs.defaultFS: マスターがyarnの場合に必要です。
    5. その他のパラメータはオプションです。詳細はhttp://spark.apache.org/docs/latest/configuration.htmlを参照してください。
    ```

    ETLにSparkを使用する場合は、working_DIRとbrokerを指定する必要があります。指示は以下の通りです:

    ```plain text
    working_dir: ETLに使用されるディレクトリ。SparkをETLリソースとして使用する場合に必要です。例: hdfs://host:port/tmp/starrocks。
    broker: ブローカーの名前。SparkをETLリソースとして使用し、以前に`ALTER SYSTEM ADD BROKER`コマンドを使用して構成する必要があります。
    broker.property_key: ブローカーがETLによって作成された中間ファイルを読み込む際に指定する必要があるプロパティ情報です。
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