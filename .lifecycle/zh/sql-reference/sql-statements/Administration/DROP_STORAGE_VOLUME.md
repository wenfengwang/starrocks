---
displayed_sidebar: English
---

# 删除存储卷

## 描述

删除一个存储卷。删除后的存储卷将无法再被引用。该功能从 v3.1 版本开始支持。

> **注意**
- 只有被授权 DROP 权限的用户才能对特定存储卷执行此操作。
- 默认存储卷和内置存储卷 `builtin_storage_volume` 不能被删除。您可以使用 [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md) 命令来检查一个存储卷是否是默认存储卷。
- 正在被现有数据库或云原生表引用的存储卷不能被删除。

## 语法

```SQL
DROP STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
```

## 参数

|参数|说明|
|---|---|
|storage_volume_name|要删除的存储卷的名称。|

## 示例

示例 1：删除名为 my_s3_volume 的存储卷。

```Plain
MySQL > DROP STORAGE VOLUME my_s3_volume;
Query OK, 0 rows affected (0.01 sec)
```

## 相关 SQL 语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [修改存储卷](./ALTER_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)
