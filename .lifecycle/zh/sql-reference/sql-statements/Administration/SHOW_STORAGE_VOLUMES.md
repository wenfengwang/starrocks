---
displayed_sidebar: English
---

# 显示存储卷

## 描述

展示您的 StarRocks 集群中的存储卷信息。该功能自 v3.1 版本起提供支持。

:::tip

执行此操作不需要特定权限。

:::

## 语法

```SQL
SHOW STORAGE VOLUMES [ LIKE '<pattern>' ]
```

## 参数

|参数|说明|
|---|---|
|pattern|用于匹配存储卷的模式。|

## 返回值

|返回|说明|
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

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [修改存储卷](./ALTER_STORAGE_VOLUME.md)
- [删除存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [查看存储卷详情](./DESC_STORAGE_VOLUME.md)
