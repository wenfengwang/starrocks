---
displayed_sidebar: English
---

# 创建资源

## 说明

此语句用于创建资源。只有 root 或 admin 用户有权限创建资源。目前，系统仅支持创建 Spark 和 Hive 类型的资源。将来，StarRocks 可能会增加对其他外部资源的支持，例如用于查询的 Spark/GPU、用于外部存储的 HDFS/S3，以及用于 ETL 的 MapReduce。

语法：

```sql
CREATE [EXTERNAL] RESOURCE "resource_name"
PROPERTIES ("key"="value", ...)
```

注意：

1. PROPERTIES 用于指定资源类型。目前，系统支持的类型仅限于 Spark 和 Hive。
2. 不同类型的资源其 PROPERTIES 配置项各不相同。详情请参考以下示例。

## 示例

1. 在 Yarn 集群模式下创建一个名为 spark0 的 Spark 资源。

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

   ```plain
   1. spark.master: required. Currently, yarn and spark ://host:port. are supported. 
   2. spark.submit.deployMode: deployment mode of the Spark program is required. Support cluster and client.
   3. spark.hadoop.yarn.resourcemanager.address: required when master is yarn.
   4. spark.hadoop.fs.defaultFS: required when master is yarn.
   5. Other parameters are optional. Please refer to http://spark.apache.org/docs/latest/configuration.html
   ```

   如果要使用 Spark 执行 ETL 任务，需要指定 working_DIR 和 broker。具体操作如下：

   ```plain
   working_dir: Directory used by ETL. It is required when spark is used as ETL resource. For example: hdfs://host:port/tmp/starrocks.
   broker: Name of broker. It is required when spark is used as ETL resource and needs to be configured beforehand by using `ALTER SYSTEM ADD BROKER` command. 
   broker.property_key: It is the property information needed to be specified when broker reads the intermediate files created by ETL. 
   ```

2. 创建一个名为 hive0 的 Hive 资源。

   ```sql
   CREATE EXTERNAL RESOURCE "hive0"
   PROPERTIES
   (
       "type" = "hive",
       "hive.metastore.uris" = "thrift://10.10.44.98:9083"
   );
   ```
