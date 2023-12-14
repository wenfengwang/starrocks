---
displayed_sidebar: "中文"
---

# 删除存储卷

## 功能

删除指定的存储卷。一旦存储卷被删除，就无法再被引用。此功能从 v3.1 版本开始支持。

> **注意**
>
> - 只有被授权 DROP 权限的用户才能执行这个操作。
> - 默认存储卷和内置的存储卷 `builtin_storage_volume` 是不能被删除的。您可以使用 [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md) 命令来检查存储卷是否是默认存储卷。
> - 正在被数据库或云原生表引用的存储卷不能被删除。

## 语法

```SQL
DROP STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
```

## 参数说明

| **参数**            | **说明**               |
| ------------------- | ---------------------- |
| storage_volume_name | 需要删除的存储卷名称。 |

## 示例

示例一：删除名为 `my_s3_volume` 的存储卷。

```Plain
MySQL > DROP STORAGE VOLUME my_s3_volume;
Query OK, 0 rows affected (0.01 sec)
```

## 相关 SQL

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)