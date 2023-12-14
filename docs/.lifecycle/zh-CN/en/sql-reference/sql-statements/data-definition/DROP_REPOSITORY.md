---
displayed_sidebar: "Chinese"
---

# 删除存储库

## 描述

删除存储库。存储库用于存储[数据备份和恢复](../../../administration/Backup_and_restore.md)的数据快照。

> **注意**
>
> - 只有root用户和超级用户才能删除存储库。
> - 此操作仅会删除存储库在StarRocks中的映射，并不会删除实际数据。您需要在远程存储系统中手动删除它。删除后，您可以通过指定相同的远程存储系统路径重新映射到该存储库。

## 语法

```SQL
DROP REPOSITORY <repository_name>
```

## 参数

| **参数**         | **描述**             |
| -----------------| ---------------------|
| repository_name  | 要删除的存储库的名称。|

## 示例

示例 1: 删除名为 `oss_repo` 的存储仓库。

```SQL
DROP REPOSITORY `oss_repo`;
```