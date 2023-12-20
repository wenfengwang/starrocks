---
displayed_sidebar: English
---

# 数据导出

## 阿里云OSS备份与还原

StarRocks支持将数据备份至阿里云OSS/AWS S3（或兼容S3协议的对象存储）。设想我们有两个StarRocks集群，分别命名为DB1集群和DB2集群。我们需要将DB1中的数据备份到阿里云OSS，然后在必要时将其还原到DB2。备份与还原的一般流程如下：

### 创建云仓库

在DB1和DB2中分别执行以下SQL：

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

a. 需要在DB1和DB2中都创建仓库，且创建的REPOSITORY名称应保持一致。查看仓库：

```sql
SHOW REPOSITORIES;
```

b. broker_name需要填写对应集群中的broker名称。查看Broker名称：

```sql
SHOW BROKER;
```

c. fs.oss.endpoint后的路径不应包含bucket名称。

### 备份数据表

将DB1中待备份的表备份到云仓库。在DB1中执行SQL：

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
PROPERTIES currently supports the following properties:
"type" = "full": indicates that this is a full update (default).
"timeout" = "3600": task timeout. The default is one day. The unit is seconds.
```

目前StarRocks不支持整个数据库的全量备份。我们需要指定待备份的表或分区ON(...)，这些表或分区将会并行进行备份。

查看正在进行的备份任务（请注意，一次只能执行一个备份任务）：

```sql
SHOW BACKUP FROM db_name;
```

备份完成后，可以检查OSS中的备份数据是否已存在（不必要的备份应从OSS中删除）：

```sql
SHOW SNAPSHOT ON OSS repository name; 
```

### 数据还原

在DB2中进行数据还原时，无需预先在DB2中创建待还原的表结构，系统会在还原操作过程中自动创建。执行还原SQL：

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROMrepository_name``
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

查看还原进度：

```sql
SHOW RESTORE;
```
