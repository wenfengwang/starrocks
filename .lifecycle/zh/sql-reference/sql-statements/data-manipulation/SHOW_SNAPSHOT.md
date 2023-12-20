---
displayed_sidebar: English
---

# 查看快照

## 描述

在指定的存储库中查看数据快照。欲了解更多信息，请参阅[data backup and restoration](../../../administration/Backup_and_restore.md)。

## 语法

```SQL
SHOW SNAPSHOT ON <repo_name>
[WHERE SNAPSHOT = <snapshot_name> [AND TIMESTAMP = <backup_timestamp>]]
```

## 参数

|参数|说明|
|---|---|
|repo_name|快照所属存储库的名称。|
|snapshot_nam|快照的名称。|
|backup_timestamp|快照的备份时间戳。|

## 返回值

|返回|说明|
|---|---|
|快照|快照的名称。|
|时间戳|快照的备份时间戳。|
|状态|如果快照正常则显示“正常”。如果快照不正确，则显示错误消息。|
|数据库|快照所属数据库的名称。|
|详细信息|JSON 格式的快照目录和结构。|

## 示例

示例 1：查看存储库 example_repo 中的快照。

```SQL
SHOW SNAPSHOT ON example_repo;
```

示例 2：查看存储库 example_repo 中名为 backup1 的快照。

```SQL
SHOW SNAPSHOT ON example_repo
WHERE SNAPSHOT = "backup1";
```

示例 3：查看存储库 example_repo 中，名为 backup1 且备份时间戳为 2018-05-05-15-34-26 的快照。

```SQL
SHOW SNAPSHOT ON example_repo 
WHERE SNAPSHOT = "backup1" AND TIMESTAMP = "2018-05-05-15-34-26";
```
