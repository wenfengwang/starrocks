---
displayed_sidebar: English
---

# 修改存储卷

## 描述

修改存储卷的凭据属性、注释或状态（`enabled`）。有关存储卷属性的详细信息，请参阅[创建存储卷](./CREATE_STORAGE_VOLUME.md)。此功能从v3.1版本开始支持。

> **注意**
>
> - 只有具有对特定存储卷的ALTER权限的用户才能执行此操作。
> - 无法更改现有存储卷的`TYPE`、`LOCATIONS`和其他与路径相关的属性。您只能更改其凭据相关的属性。如果更改了与路径相关的配置项，那么在更改之前创建的数据库和表将变为只读，并且无法向其中加载数据。
> - 当`enabled`为`false`时，无法引用相应的存储卷。

## 语法

```SQL
ALTER STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
{ COMMENT '<comment_string>'
| SET ("key" = "value"[,...]) }
```

## 参数

| **参数**             | **描述**                   |
| ------------------- | ------------------------ |
| storage_volume_name | 要修改的存储卷的名称。        |
| COMMENT             | 存储卷的注释。              |

有关可以修改或添加的属性的详细信息，请参阅[创建存储卷 - 属性](./CREATE_STORAGE_VOLUME.md#properties)。

## 例子

示例1：禁用存储卷`my_s3_volume`。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET ("enabled" = "false");
Query OK, 0 rows affected (0.01 sec)
```

示例2：修改存储卷`my_s3_volume`的凭据信息。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET (
    ->     "aws.s3.use_instance_profile" = "true"
    -> );
Query OK, 0 rows affected (0.00 sec)
```

## 相关 SQL 语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [丢弃存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC 存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)
