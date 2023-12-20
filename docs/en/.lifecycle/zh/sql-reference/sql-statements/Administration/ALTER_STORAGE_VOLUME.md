---
displayed_sidebar: English
---

# 修改存储卷

## 描述

修改存储卷的凭证属性、注释或状态（`enabled`）。有关存储卷属性的更多信息，请参阅[创建存储卷](./CREATE_STORAGE_VOLUME.md)。此功能从 v3.1 版本开始支持。

> **注意**
- 只有拥有特定存储卷的 ALTER 权限的用户才能执行此操作。
- 现有存储卷的 `TYPE`、`LOCATIONS` 和其他与路径相关的属性无法修改。您只能修改其与凭证相关的属性。如果您更改了路径相关的配置项，之前创建的数据库和表将变为只读，您将无法向其中加载数据。
- 当 `enabled` 为 `false` 时，相应的存储卷将无法被引用。

## 语法

```SQL
ALTER STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
{ COMMENT '<comment_string>'
| SET ("key" = "value"[,...]) }
```

## 参数

|**参数**|**描述**|
|---|---|
|storage_volume_name|要修改的存储卷名称。|
|COMMENT|对存储卷的注释。|

有关可以修改或添加的属性的详细信息，请参阅[创建存储卷 - 属性](./CREATE_STORAGE_VOLUME.md#properties)。

## 示例

示例 1：禁用存储卷 `my_s3_volume`。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET ("enabled" = "false");
Query OK, 0 rows affected (0.01 sec)
```

示例 2：修改存储卷 `my_s3_volume` 的凭证信息。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET (
    ->     "aws.s3.use_instance_profile" = "true"
    -> );
Query OK, 0 rows affected (0.00 sec)
```

## 相关 SQL 语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [删除存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [描述存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)