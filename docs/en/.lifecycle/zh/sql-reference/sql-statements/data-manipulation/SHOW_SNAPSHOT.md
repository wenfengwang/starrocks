---
displayed_sidebar: English
---

# 显示快照

## 描述

在指定的存储库中查看数据快照。更多信息，请参见[data backup and restoration](../../../administration/Backup_and_restore.md)。

## 语法

```SQL
SHOW SNAPSHOT ON <repo_name>
[WHERE SNAPSHOT = <snapshot_name> [AND TIMESTAMP = <backup_timestamp>]]
```

## 参数

|**参数**|**说明**|
|---|---|
|repo_name|快照所属的存储库名称。|
|snapshot_name|快照的名称。|
|backup_timestamp|快照的备份时间戳。|

## 返回值

|**返回**|**说明**|
|---|---|
|Snapshot|快照的名称。|
|Timestamp|快照的备份时间戳。|
|Status|如果快照状态良好，则显示`OK`。如果快照有问题，则显示错误信息。|
|Database|快照所属的数据库名称。|
|Details|快照的目录和结构的JSON格式。|

## 示例

示例 1：查看存储库`example_repo`中的快照。

```SQL
SHOW SNAPSHOT ON example_repo;
```

示例 2：查看存储库`example_repo`中名为`backup1`的快照。

```SQL
SHOW SNAPSHOT ON example_repo
WHERE SNAPSHOT = "backup1";
```

示例 3：查看存储库`example_repo`中名为`backup1`、备份时间戳为`2018-05-05-15-34-26`的快照。

```SQL
SHOW SNAPSHOT ON example_repo 
WHERE SNAPSHOT = "backup1" AND TIMESTAMP = "2018-05-05-15-34-26";
```