---
displayed_sidebar: English
---

# 设置默认存储卷

## 描述

将存储卷设置为默认存储卷。在为外部数据源创建存储卷后，您可以将其设置为 StarRocks 集群的默认存储卷。此功能从 v3.1 版本开始支持。

> **注意**
>
> - 只有对特定存储卷具有 USAGE 权限的用户才能执行此操作。
> - 默认存储卷无法被删除或禁用。
> - 对于共享数据的 StarRocks 集群，必须设置默认存储卷，因为 StarRocks 将系统统计信息存储在默认存储卷中。

## 语法

```SQL
SET <storage_volume_name> AS DEFAULT STORAGE VOLUME
```

## 参数

| **参数**       | **描述**                                              |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | 要设置为默认存储卷的存储卷名称。 |

## 例子

示例 1：将存储卷 `my_s3_volume` 设置为默认存储卷。

```SQL
MySQL > SET my_s3_volume AS DEFAULT STORAGE VOLUME;
Query OK, 0 rows affected (0.01 sec)
```

## 相关 SQL 语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [更改存储卷](./ALTER_STORAGE_VOLUME.md)
- [丢弃存储卷](./DROP_STORAGE_VOLUME.md)
- [DESC 存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)
