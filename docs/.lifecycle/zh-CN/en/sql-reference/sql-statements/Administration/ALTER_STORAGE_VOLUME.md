---
displayed_sidebar: "Chinese"
---

# ALTER STORAGE VOLUME

## 描述

修改存储卷的凭据属性、注释或状态（`enabled`）。有关存储卷属性的更多信息，请参阅[CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)。此功能在 v3.1 版本中受支持。

> **注意**
>
> - 只有具有对特定存储卷的 ALTER 权限的用户才能执行此操作。
> - 现有存储卷的 `TYPE`、`LOCATIONS` 和其他与路径相关的属性无法更改。您只能更改其与凭据相关的属性。如果更改了与路径相关的配置项，则在更改之前创建的数据库和表将变为只读，并且无法将数据加载到其中。
> - 当 `enabled` 为 `false` 时，相应的存储卷将无法被引用。

## 语法

```SQL
ALTER STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
{ COMMENT '<comment_string>'
| SET ("key" = "value"[,...]) }
```

## 参数

| **参数**             | **描述**                   |
| ------------------- | ------------------------ |
| storage_volume_name | 要修改的存储卷的名称。      |
| COMMENT             | 存储卷的注释。             |

有关可以修改或添加的属性的详细信息，请参阅[CREATE STORAGE VOLUME - PROPERTIES](./CREATE_STORAGE_VOLUME.md#properties)。

## 示例

示例 1: 禁用存储卷 `my_s3_volume`。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET ("enabled" = "false");
查询成功，影响行数为 0（0.01 秒）
```

示例 2: 修改存储卷 `my_s3_volume` 的凭据信息。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET (
    ->     "aws.s3.use_instance_profile" = "true"
    -> );
查询成功，影响行数为 0（0.00 秒）
```

## 相关 SQL 语句

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)