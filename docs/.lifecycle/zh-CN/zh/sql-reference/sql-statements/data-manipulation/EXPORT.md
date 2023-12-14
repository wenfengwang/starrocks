---
displayed_sidebar: "汉语"
---

# 导出

## 功能

该语句用于将指定表中的数据导出至指定的位置。

这是一个异步操作，任务提交成功后将返回结果。执行该操作后，可使用 [SHOW EXPORT](../../../sql-reference/sql-statements/data-manipulation/SHOW_EXPORT.md) 命令来检查进度。

> **注意**
>
> 执行导出操作需要有目标表的 EXPORT 权限。如果您的用户账号没有 EXPORT 权限，请参考 [GRANT](../account-management/GRANT.md) 文档来为用户授权。

## 语法

```sql
EXPORT TABLE <table_name>
[PARTITION (<partition_name>[, ...])]
[(<column_name>[, ...])]
TO <export_path>
[opt_properties]
WITH BROKER
[broker_properties]
```

## 参数说明

- `table_name`

  欲导出数据所在的表。当前支持导出 `engine` 为 `olap` 或 `mysql` 的表。

- `partition_name`

  欲导出的分区。若不指定，则默认导出表中所有分区的数据。

- `column_name`

  欲导出的列。导出列的顺序可以与源表结构 (Schema) 不同。若不指定，则默认导出表中所有列的数据。

- `export_path`

  导出的目标路径。若为目录，需以斜线 (/) 结尾。否则最后一个斜线之后的部分将作为导出文件的前缀。若不指定文件名前缀，则默认文件名前缀为 `data_`。

- `opt_properties`

  与导出相关的属性配置。

  语法：

  ```sql
  [PROPERTIES ("<key>"="<value>", ...)]
  ```

  配置项：

  | **配置项**         | **描述**                                                     |
  | ---------------- | ------------------------------------------------------------ |
  | column_separator | 指定导出文件中的列分隔符。默认值为 `\t`。                       |
  | line_delimiter   | 指定导出文件中的行分隔符。默认值为 `\n`。                       |
  | load_mem_limit   | 指定导出任务在单个 BE 节点上的内存使用上限。单位为字节。默认内存使用上限为 2GB。 |
  | timeout          | 指定导出任务的超时时间。单位为秒。默认值为 `86400`（1天）。  |
  | include_query_id | 指定是否在导出文件名中包含 `query_id`。取值范围为 `true` 或 `false`。默认值为 `true`。`true` 表示包含，`false` 表示不包含。 |

- `WITH BROKER`

  在 v2.4 及之前的版本中，需通过 `WITH BROKER "<broker_name>"` 指定使用哪个 Broker。自 v2.5 起，不再需要指定 `broker_name`，但保留了 `WITH BROKER` 关键字。参见 [使用 EXPORT 导出数据 > 背景信息](../../../unloading/Export.md#背景信息)。

- `broker_properties`

  提供访问数据源的鉴权信息。不同的数据源需要提供的鉴权信息不同，详情参考 [BROKER LOAD](../../../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

## 示例

### 将表中所有数据导出至 HDFS

将 `testTbl` 表中所有数据导出至 HDFS 集群的指定路径下。

```sql
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
    "username"="xxx",
    "password"="yyy"
);
```

### 将表中部分分区的数据导出至 HDFS

将 `testTbl` 表中 `p1` 和 `p2` 分区的数据导出至 HDFS 集群的指定路径下。

```sql
EXPORT TABLE testTbl
PARTITION (p1,p2) 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
WITH BROKER
(
    "username"="xxx",
    "password"="yyy"
);
```

### 指定分隔符

将 `testTbl` 表中所有数据导出至 HDFS 集群的指定路径下，并使用 `,` 作为导出文件的列分隔符。

```sql
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"=","
) 
WITH BROKER
(
    "username"="xxx",
    "password"="yyy"
);
```

将 `testTbl` 表中所有数据导出至 HDFS 集群的指定路径下，并使用 Hive 默认分隔符 `\x01` 作为导出文件的列分隔符。

```sql
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/" 
PROPERTIES
(
    "column_separator"="\\x01"
) 
WITH BROKER;
```

### 指定导出文件名前缀

将 `testTbl` 表中所有数据导出至 HDFS 集群的指定路径下，并将 `testTbl_` 作为导出文件的文件名前缀。

```sql
EXPORT TABLE testTbl 
TO "hdfs://<hdfs_host>:<hdfs_port>/a/b/c/testTbl_" 
WITH BROKER;
```

### 导出数据至阿里云 OSS

将 `testTbl` 表中所有数据导出至阿里云 OSS 存储桶的指定路径下。

```sql
EXPORT TABLE testTbl 
TO "oss://oss-package/export/"
WITH BROKER
(
    "fs.oss.accessKeyId" = "xxx",
    "fs.oss.accessKeySecret" = "yyy",
    "fs.oss.endpoint" = "oss-cn-zhangjiakou-internal.aliyuncs.com"
);
```

### 导出数据至腾讯云 COS

将 `testTbl` 表中所有数据导出至腾讯云 COS 存储桶的指定路径下。

```sql
EXPORT TABLE testTbl 
TO "cosn://cos-package/export/"
WITH BROKER
(
    "fs.cosn.userinfo.secretId" = "xxx",
    "fs.cosn.userinfo.secretKey" = "yyy",
    "fs.cosn.bucket.endpoint_suffix" = "cos.ap-beijing.myqcloud.com"
);
```

### 导出数据至 AWS S3

将 `testTbl` 表中所有数据导出至 AWS S3 存储桶的指定路径下。

```sql
EXPORT TABLE testTbl 
TO "s3a://s3-package/export/"
WITH BROKER
(
    "aws.s3.access_key" = "xxx",
    "aws.s3.secret_key" = "yyy",
    "aws.s3.region" = "zzz"
);
```

### 导出数据至华为云 OBS

将 `testTbl` 表中所有数据导出至华为云 OBS 存储桶的指定路径下。

```sql
EXPORT TABLE testTbl 
TO "obs://obs-package/export/"
WITH BROKER
(
    "fs.obs.access.key" = "xxx",
    "fs.obs.secret.key" = "yyy",
    "fs.obs.endpoint" = "obs.cn-east-3.myhuaweicloud.com"
);
```

> **说明**
>
> 当导出数据至华为云 OBS 时，需先下载[依赖库](https://github.com/huaweicloud/obsa-hdfs/releases/download/v45/hadoop-huaweicloud-2.8.3-hw-45.jar)并添加至 **$BROKER_HOME/lib/** 路径下，然后重启 Broker。