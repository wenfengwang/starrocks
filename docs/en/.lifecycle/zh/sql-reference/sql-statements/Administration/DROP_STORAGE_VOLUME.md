---
displayed_sidebar: English
---

# 删除存储卷

## 描述

删除一个存储卷。已删除的存储卷将无法再被引用。此功能从 v3.1 版本开始支持。

> **注意**
>
> - 只有对特定存储卷具有删除（DROP）权限的用户才能执行此操作。
> - 默认存储卷和内置存储卷 `builtin_storage_volume` 无法被删除。您可以使用 [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md) 来检查一个存储卷是否为默认存储卷。
> - 被现有数据库或云原生表引用的存储卷无法被删除。

## 语法

```SQL
DROP STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
```

## 参数

| **参数**       | **描述**                         |
| ------------------- | --------------------------------------- |
| storage_volume_name | 要删除的存储卷的名称。 |

## 例子

示例 1：删除存储卷 `my_s3_volume`。

```Plain
MySQL > DROP STORAGE VOLUME my_s3_volume;
Query OK, 0 行受到影响 (0.01 秒)
```

## 相关 SQL 语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [更改存储卷](./ALTER_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC 存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)
