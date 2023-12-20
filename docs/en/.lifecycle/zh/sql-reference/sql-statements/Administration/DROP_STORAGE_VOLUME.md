---
displayed_sidebar: English
---

# 删除存储卷

## 描述

删除一个存储卷。删除后的存储卷将无法再被引用。该功能从 v3.1 版本开始支持。

> **注意**
- 只有对特定存储卷拥有 DROP 权限的用户才能执行此操作。
- 默认存储卷和内置存储卷 `builtin_storage_volume` 不能被删除。您可以使用 [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md) 来检查一个存储卷是否是默认存储卷。
- 正被现有数据库或云原生表引用的存储卷不能被删除。

## 语法

```SQL
DROP STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
```

## 参数

|**参数**|**说明**|
|---|---|
|storage_volume_name|要删除的存储卷名称。|

## 示例

示例 1：删除存储卷 `my_s3_volume`。

```Plain
MySQL > DROP STORAGE VOLUME my_s3_volume;
Query OK, 0 rows affected (0.01 sec)
```

## 相关 SQL 语句

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)