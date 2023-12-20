---
displayed_sidebar: English
---

# 修改存储卷

## 描述

更改存储卷的凭证属性、备注或状态（`enabled`）。有关存储卷属性的更多信息，请参见[创建存储卷](./CREATE_STORAGE_VOLUME.md)。该功能从v3.1版本开始支持。

> **注意**
- 只有拥有特定存储卷ALTER权限的用户才能执行此操作。
- 现有存储卷的TYPE、LOCATIONS以及其他与路径相关的属性无法被修改。您只能修改与凭证相关的属性。如果您更改了与路径相关的配置项，那么更改之前创建的数据库和表将变为只读状态，您将无法往其中加载数据。
- 当enabled设置为false时，相应的存储卷将无法被引用。

## 语法

```SQL
ALTER STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
{ COMMENT '<comment_string>'
| SET ("key" = "value"[,...]) }
```

## 参数

|参数|说明|
|---|---|
|storage_volume_name|要更改的存储卷的名称。|
|COMMENT|对存储卷的评论。|

关于可以修改或添加的属性的详细信息，请参见[创建存储卷 - 属性](./CREATE_STORAGE_VOLUME.md#properties)部分。

## 示例

示例 1：禁用名为my_s3_volume的存储卷。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET ("enabled" = "false");
Query OK, 0 rows affected (0.01 sec)
```

示例 2：修改名为my_s3_volume的存储卷的凭证信息。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET (
    ->     "aws.s3.use_instance_profile" = "true"
    -> );
Query OK, 0 rows affected (0.00 sec)
```

## 相关SQL语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [删除存储卷](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)（设置默认存储卷）
- [描述存储卷](./DESC_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)
