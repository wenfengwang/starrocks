---
displayed_sidebar: English
---

# 删除仓库

## 说明

删除一个仓库。仓库用于存储数据快照，以便进行[数据备份和恢复](../../../administration/Backup_and_restore.md)。

> **注意**
- 只有 root 用户和超级用户有权限删除仓库。
- 此操作只会移除 StarRocks 中对应仓库的映射，并不会删除实际的数据。您需要手动在远程存储系统中进行删除。删除映射后，您可以通过指定相同的远程存储系统路径来重新映射该仓库。

## 语法

```SQL
DROP REPOSITORY <repository_name>
```

## 参数

|参数|说明|
|---|---|
|repository_name|要删除的存储库的名称。|

## 示例

示例 1：删除一个名为 oss_repo 的仓库。

```SQL
DROP REPOSITORY `oss_repo`;
```
