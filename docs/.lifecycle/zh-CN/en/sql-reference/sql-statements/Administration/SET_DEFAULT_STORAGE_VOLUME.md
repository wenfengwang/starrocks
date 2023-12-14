---
displayed_sidebar: "Chinese"
---

# 设置默认存储卷

## 描述

将存储卷设置为默认存储卷。在为外部数据源创建存储卷后，可以将其设置为StarRocks集群的默认存储卷。此功能在v3.1版中得到支持。

> **注意**
>
> - 只有对特定存储卷具有USAGE权限的用户才能执行此操作。
> - 默认存储卷无法被删除或禁用。
> - 必须为共享数据的StarRocks集群设置默认存储卷，因为StarRocks将系统统计信息存储在默认存储卷中。

## 语法

```SQL
SET <storage_volume_name> AS DEFAULT STORAGE VOLUME
```

## 参数

| **参数**            | **描述**                             |
| ------------------- | ------------------------------------ |
| storage_volume_name | 要设置为默认存储卷的存储卷的名称。 |

## 示例

示例1: 将存储卷`my_s3_volume`设置为默认存储卷。

```SQL
MySQL > SET my_s3_volume AS DEFAULT STORAGE VOLUME;
查询成功，影响行数：0，耗时0.01秒
```

## 相关的SQL语句

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)