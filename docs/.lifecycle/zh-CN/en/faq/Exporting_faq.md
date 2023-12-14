```yaml
---
displayed_sidebar: "Chinese"
---

# 数据导出

## 阿里云 OSS 备份和恢复

StarRocks 支持将数据备份到阿里云 OSS / AWS S3（或与 S3 协议兼容的对象存储）。假设有两个 StarRocks 集群，分别为 DB1 集群和 DB2 集群。我们需要将 DB1 中的数据备份到阿里云 OSS，然后在必要时恢复到 DB2。备份和恢复的一般流程如下：

### 创建云存储库

分别在 DB1 和 DB2 中执行 SQL：

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

a. 需要在 DB1 和 DB2 中都创建，创建的 REPOSITORY 名称应保持一致。查看存储库：

```sql
SHOW REPOSITORIES;
```

b. broker_name 需要填入集群中的 broker 名称。查看 BrokerName：

```sql
SHOW BROKER;
```

c. 在 fs.oss.endpoint 后的路径不需要包含 bucket 名称。

### 备份数据表

在 DB1 中将要备份的表备份到云存储库。在 DB1 中执行 SQL：

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
PROPERTIES 目前支持以下属性：
"type" = "full"：表示这是一次全量更新（默认）。
"timeout" = "3600"：任务超时时间。默认为一天。单位为秒。
```

目前 StarRocks 不支持全库备份。我们需要指定要备份的表或分区 ON(...)，这些表或分区将会并行备份。

查看正在进行的备份任务（注意：同时只能执行一个备份任务）：

```sql
SHOW BACKUP FROM db_name;
```

备份完成后，可以检查 OSS 中备份数据是否已经存在（不必要的备份需要在 OSS 中删除）：

```sql
SHOW SNAPSHOT ON OSS repository name; 
```

### 数据恢复

对于 DB2 中的数据恢复，无需在 DB2 中创建要恢复的表结构，恢复操作时会自动创建。执行恢复 SQL：

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROMrepository_name``
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