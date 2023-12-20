---
displayed_sidebar: English
---

# 数据导出

## 阿里云OSS备份与恢复

StarRocks支持将数据备份到阿里云OSS/AWS S3（或兼容S3协议的对象存储）。假设有两个StarRocks集群，分别是DB1集群和DB2集群。我们需要将DB1中的数据备份到阿里云OSS，必要时再恢复到DB2。备份与恢复的大致流程如下：

### 创建云存储库

分别在DB1和DB2中执行SQL：

```sql
CREATE REPOSITORY `repository_name`
WITH BROKER `broker_name`
ON LOCATION "oss://bucket_name/path"
PROPERTIES
(
"fs.oss.accessKeyId" = "xxx",
"fs.oss.accessKeySecret" = "yyy",
"fs.oss.endpoint" = "oss-cn-beijing.aliyuncs.com"
);
```

a. DB1和DB2都需要创建，创建的REPOSITORY名称要相同。查看存储库：

```sql
SHOW REPOSITORIES;
```

b. broker_name需要填写集群中的broker名称。查看Broker名称：

```sql
SHOW BROKER;
```

c. fs.oss.endpoint之后的路径不需要包含bucket名称。

### 备份数据表

将需要备份的表BACKUP到DB1的云存储库中。在DB1中执行SQL：

```sql
BACKUP SNAPSHOT [db_name].{snapshot_name}
TO `repository_name`
ON (
`table_name` [PARTITION (`p1`, ...)],
...
)
PROPERTIES ("key"="value", ...);
```

```plain
PROPERTIES目前支持以下属性：
"type" = "full": 表示这是一个完整更新（默认）。
"timeout" = "3600": 任务超时时间，默认是一天，单位是秒。
```

StarRocks目前不支持全库备份。我们需要在ON (...)中指定要备份的表或分区，这些表或分区将并行备份。

查看正在进行的备份任务（注意：同一时间只能执行一个备份任务）：

```sql
SHOW BACKUP FROM db_name;
```

备份完成后，您可以检查OSS中是否已存在备份数据（不需要的备份需要在OSS中删除）：

```sql
SHOW SNAPSHOT ON OSS `repository_name`; 
```

### 数据恢复

对于DB2中的数据恢复，不需要在DB2中预先创建要恢复的表结构。它将在执行RESTORE操作时自动创建。执行恢复SQL：

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM `repository_name`
ON (
    `table_name` [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

查看恢复进度：

```sql
SHOW RESTORE;
```