---
displayed_sidebar: "Chinese"
---

# 导出

## 阿里云OSS备份与还原

StarRocks supports backing up data to Alibaba Cloud OSS/AWS S3 (or S3 compatible object storage). Suppose there are two StarRocks clusters, DB1 cluster and DB2 cluster. We need to backup the data from DB1 to Alibaba Cloud OSS, and then restore it to DB2 when needed. The general backup and restore process is as follows:

### Create Cloud Repository

Execute the following SQL statements in DB1 and DB2 separately:

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

a. Both DB1 and DB2 need to create repositories, and the names of the created repositories should be the same. You can view the repositories with:

```sql
SHOW REPOSITORIES;
```

b. The broker_name should be filled in with the name of a broker in the cluster. You can view the BrokerName with:

```sql
SHOW BROKER;
```

c. The path after fs.oss.endpoint does not need to include the bucket name.

### Backup Data Table

In DB1, BACKUP the tables that need to be backed up to the cloud repository. Execute the following SQL in DB1:

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
PROPERTIES currently support the following attributes:
"type" = "full": indicates a full update (default).
"timeout" = "3600": the task timeout period, default is one day, in seconds.
```

StarRocks currently does not support backing up the entire database. We need to specify the tables or partitions that need to be backed up in ON (……), and these tables or partitions will be backed up in parallel.
View the backup tasks in progress (note that only one backup task can be performed at the same time):

```sql
SHOW BACKUP FROM db_name;
```

After the backup is completed, you can check if the backup data already exists in OSS (unnecessary backups should be deleted from OSS):

```sql
SHOW SNAPSHOT ON OSS repository_name; 
```

### Data Restoration

Restore data in DB2. There is no need to create the table structure that needs to be restored in DB2, as it will be automatically created during the Restore operation. Execute the following SQL for restoration:

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM repository_name
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

View the restoration progress:

```sql
SHOW RESTORE;
```