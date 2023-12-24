---
displayed_sidebar: English
---

# 显示快照

## 描述

查看指定存储库中的数据快照。更多信息，请参见 [数据备份和恢复](../../../administration/Backup_and_restore.md)。

## 语法

```SQL
SHOW SNAPSHOT ON <repo_name>
[WHERE SNAPSHOT = <snapshot_name> [AND TIMESTAMP = <backup_timestamp>]]
```

## 参数

| **参数**    | **描述**                                      |
| ---------------- | ---------------------------------------------------- |
| repo_name        | 快照所属的存储库名称。 |
| snapshot_name     | 快照的名称。                                |
| backup_timestamp | 快照的备份时间戳。                    |

## 返回

| **返回** | **描述**                                              |
| ---------- | ------------------------------------------------------------ |
| 快照   | 快照的名称。                                        |
| 时间戳  | 快照的备份时间戳。                            |
| 状态     | 如果快照正常，则显示 `OK`。如果快照不正常，则显示错误消息。 |
| 数据库   | 快照所属的数据库名称。           |
| 详细    | 快照的 JSON 格式目录和结构。      |

## 例

示例 1：查看存储库 `example_repo` 中的快照。

```SQL
SHOW SNAPSHOT ON example_repo;
```

示例 2：查看存储库 `example_repo` 中名为 `backup1` 的快照。

```SQL
SHOW SNAPSHOT ON example_repo
WHERE SNAPSHOT = "backup1";
```

示例 3：查看存储库 `example_repo` 中名为 `backup1` 且备份时间戳为 `2018-05-05-15-34-26` 的快照。

```SQL
SHOW SNAPSHOT ON example_repo 
WHERE SNAPSHOT = "backup1" AND TIMESTAMP = "2018-05-05-15-34-26";
```
