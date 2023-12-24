---
displayed_sidebar: English
---

# 数据导出

## 阿里云OSS备份和恢复

StarRocks支持将数据备份到阿里云OSS/AWS S3（或兼容S3协议的对象存储）。假设有两个StarRocks集群，分别是DB1集群和DB2集群。我们需要在必要时将DB1中的数据备份到阿里云OSS，然后恢复到DB2。备份和恢复的一般过程如下：

### 创建云存储库

分别在DB1和DB2中执行以下SQL：

```sql
CREATE REPOSITORY `repository name`
WITH BROKER `broker_name`
ON LOCATION "oss://bucket name/path"
PROPERTIES
(
"fs.oss.accessKeyId" = "xxx",
"fs.oss.accessKeySecret" = "yyy",
"fs.oss.endpoint" = "oss-cn-beijing.aliyuncs.com"
);
```

a. 需要在DB1和DB2中都创建，并且创建的REPOSITORY名称应该相同。查看存储库：

```sql
SHOW REPOSITORIES;
```

b. broker_name需要填写集群中的broker名称。查看BrokerName：

```sql
SHOW BROKER;
```

c. fs.oss.endpoint之后的路径不需要有存储桶名称。

### 备份数据表

在DB1中将要备份的表备份到云存储库。执行以下SQL在DB1中：

```sql
BACKUP SNAPSHOT [db_name].{snapshot_name}
TO `repository_name`
ON (
`table_name` [PARTITION (`p1`, ...)],
...
)
PROPERTIES ("key"="value", ...);
```

```plain text
PROPERTIES目前支持以下属性：
"type" = "full"：表示这是一个全量更新（默认）。
"timeout" = "3600"：任务超时。默认为一天。单位为秒。
```

目前StarRocks不支持全数据库备份。我们需要指定要备份的表或分区ON(...)，这些表或分区将并行备份。

查看正在进行的备份任务（请注意，同时只能执行一个备份任务）：

```sql
SHOW BACKUP FROM db_name;
```

备份完成后，您可以检查OSS中是否已经存在备份数据（不必要的备份需要在OSS中删除）：

```sql
SHOW SNAPSHOT ON OSS repository name; 
```

### 数据恢复

对于DB2中的数据恢复，无需在DB2中创建要恢复的表结构。在恢复操作期间，它将自动创建。执行还原SQL：

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM `repository_name`
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

查看恢复进度：

```sql
SHOW RESTORE;
```
