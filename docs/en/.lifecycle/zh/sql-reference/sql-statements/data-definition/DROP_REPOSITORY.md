---
displayed_sidebar: English
---

# 删除存储库

## 描述

删除存储库。存储库用于存储数据快照，以便进行[数据备份和恢复](../../../administration/Backup_and_restore.md)。

> **注意**
>
> - 只有 root 用户和超级用户可以删除存储库。
> - 此操作仅会删除 StarRocks 中存储库的映射，不会删除实际数据。您需要在远程存储系统中手动删除数据。删除后，您可以通过指定相同的远程存储系统路径再次映射到该存储库。

## 语法

```SQL
DROP REPOSITORY <repository_name>
```

## 参数

| **参数**   | **描述**                       |
| --------------- | ------------------------------------- |
| repository_name | 要删除的存储库的名称。 |

## 示例

示例 1：删除名为 `oss_repo` 的存储库。

```SQL
DROP REPOSITORY `oss_repo`;