---
displayed_sidebar: English
---

# 显示存储卷

## 描述

显示您的 StarRocks 集群中的存储卷。此功能从 v3.1 版本开始支持。

:::提示

此操作不需要特殊权限。

:::

## 语法

```SQL
SHOW STORAGE VOLUMES [ LIKE '<pattern>' ]
```

## 参数

| **参数** | **描述**                                |
| ------------- | ---------------------------------------------- |
| pattern       | 用于匹配存储卷的模式。 |

## 返回值

| **返回**     | **描述**                 |
| -------------- | ------------------------------- |
| 存储卷 | 存储卷的名称。 |

## 例子

示例 1：显示 StarRocks 集群中的所有存储卷。

```Plain
MySQL > SHOW STORAGE VOLUMES;
+----------------+
| 存储卷 |
+----------------+
| my_s3_volume   |
+----------------+
1 行结果 (0.01 秒)

```

## 相关 SQL 语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [更改存储卷](./ALTER_STORAGE_VOLUME.md)
- [丢弃存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC 存储卷](./DESC_STORAGE_VOLUME.md)
