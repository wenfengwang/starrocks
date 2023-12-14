---
displayed_sidebar: "Chinese"
---

# 创建资源

## 描述

此语句用于创建资源。只有用户root或管理员可以创建资源。当前仅支持Spark和Hive资源。将来可能会添加其他外部资源到StarRocks，例如用于查询的Spark/GPU，外部存储的HDFS/S3以及用于ETL的MapReduce。

语法：

```sql
CREATE [EXTERNAL] RESOURCE "resource_name"
PROPERTIES ("key"="value", ...)
```

注意：

1. PROPERTIES指定资源类型。目前仅支持Spark和Hive。
2. PROPERTIES根据资源类型而变化。详情请参考示例。

## 示例

1. 创建一个名为spark0的Spark资源，处于yarn Cluster模式。

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

    与Spark相关的参数如下：

    ```plain text
    1. spark.master: 必需。目前支持yarn和spark：//host:port。
    2. spark.submit.deployMode: 必需的Spark程序部署模式。支持cluster和client。
    3. spark.hadoop.yarn.resourcemanager.address: 当master为yarn时必需。
    4. spark.hadoop.fs.defaultFS: 当master为yarn时必需。
    5. 其他参数是可选的。详情请参考 http://spark.apache.org/docs/latest/configuration.html
    ```

    如果Spark用于ETL，需要指定working_DIR和broker。操作指南如下：

    ```plain text
    working_dir: ETL使用的目录。当Spark用作ETL资源时需要。例如：hdfs://host:port/tmp/starrocks。
    broker: broker的名称。当Spark用作ETL资源并且需要通过`ALTER SYSTEM ADD BROKER`命令预先配置时是必需的。
    broker.property_key: 这是broker读取ETL创建的中间文件时需要指定的属性信息。
    ```

2. 创建一个名为hive0的Hive资源。

    ```sql
    CREATE EXTERNAL RESOURCE "hive0"
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.uris" = "thrift://10.10.44.98:9083"
    );
    ```