---
displayed_sidebar: English
---

# 显示存储卷

## 描述

显示 StarRocks 集群中的存储卷。该功能从 v3.1 版本开始支持。

:::tip

此操作不需要权限。

:::

## 语法

```SQL
SHOW STORAGE VOLUMES [ LIKE '<pattern>' ]
```

## 参数

|**参数**|**说明**|
|---|---|
|pattern|用于匹配存储卷的模式。|

## 返回值

|**返回**|**说明**|
|---|---|
|存储卷|存储卷的名称。|

## 示例

示例 1：展示 StarRocks 集群中的所有存储卷。

```Plain
MySQL > SHOW STORAGE VOLUMES;
+----------------+
| Storage Volume |
+----------------+
| my_s3_volume   |
+----------------+
1 row in set (0.01 sec)
```

## 相关 SQL 语句

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)