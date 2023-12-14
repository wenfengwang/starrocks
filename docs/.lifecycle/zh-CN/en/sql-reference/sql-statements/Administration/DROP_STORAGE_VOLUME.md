---
displayed_sidebar: "Chinese"
---

# 删除存储卷

## 描述

删除一个存储卷。删除后的存储卷将不再被引用。此功能被支持自v3.1。

> **注意**
>
> - 只有在特定存储卷上具有DROP权限的用户可以执行此操作。
> - 默认存储卷和内置存储卷`builtin_storage_volume` 不能被删除。您可以使用[DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)来查看一个存储卷是否为默认存储卷。
> - 被现有数据库或云原生表引用的存储卷不能被删除。

## 语法

```SQL
DROP STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
```

## 参数

| **参数**            | **描述**            |
| ------------------- | ------------------- |
| storage_volume_name | 要删除的存储卷名称。 |

## 示例

示例1：删除存储卷`my_s3_volume`。

```Plain
MySQL > DROP STORAGE VOLUME my_s3_volume;
Query OK, 0 rows affected (0.01 sec)
```

## 相关SQL语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [修改存储卷](./ALTER_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [描述存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)