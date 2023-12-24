---
displayed_sidebar: English
---

# 创建资源

## 描述

此语句用于创建资源。只有 root 用户或 admin 用户才能创建资源。目前仅支持 Spark 和 Hive 资源。未来 StarRocks 可能会添加其他外部资源，例如用于查询的 Spark/GPU、用于外部存储的 HDFS/S3、用于 ETL 的 MapReduce。

语法：

```sql
CREATE [EXTERNAL] RESOURCE "resource_name"
PROPERTIES ("key"="value", ...)
```

注意：  

1. PROPERTIES 指定资源类型。目前仅支持 Spark 和 Hive。
2. 属性因资源类型而异。有关详细信息，请参阅示例。

## 例子

1. 在 yarn 集群模式下创建名为 spark0 的 Spark 资源。

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

    与 Spark 相关的参数如下：

    ```plain text
    1. spark.master：必填项。目前支持 yarn 和 spark ://host:port。
    2. spark.submit.deployMode：需要指定 Spark 程序的部署模式。支持 cluster 和 client。
    3. spark.hadoop.yarn.resourcemanager.address：当 master 为 yarn 时为必填项。
    4. spark.hadoop.fs.defaultFS：当 master 为 yarn 时为必填项。
    5. 其他参数为可选项。请参考 http://spark.apache.org/docs/latest/configuration.html
    ```

    如果 Spark 用于 ETL，则需要指定 working_dir 和 broker。具体说明如下：

    ```plain text
    working_dir：ETL 使用的目录。当 Spark 用作 ETL 资源时为必填项。例如：hdfs://host:port/tmp/starrocks。
    broker：Broker 的名称。当 Spark 用作 ETL 资源时为必填项，并且需要使用 `ALTER SYSTEM ADD BROKER` 命令预先进行配置。
    broker.property_key：这是在 broker 读取 ETL 创建的中间文件时需要指定的属性信息。
    ```

2. 创建名为 hive0 的 Hive 资源。

    ```sql
    CREATE EXTERNAL RESOURCE "hive0"
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.uris" = "thrift://10.10.44.98:9083"
    );
    ```
