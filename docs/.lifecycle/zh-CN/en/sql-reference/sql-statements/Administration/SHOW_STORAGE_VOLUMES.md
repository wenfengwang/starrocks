---
displayed_sidebar: "Chinese"
---

# 显示存储卷

## 描述

显示StarRocks集群中的存储卷。此功能从v3.1版本开始支持。

## 语法

```SQL
SHOW STORAGE VOLUMES [ LIKE '<pattern>' ]
```

## 参数

| **参数**   | **描述**                      |
| ---------- | ----------------------------- |
| 模式       | 用于匹配存储卷的模式。         |

## 返回值

| **返回**      | **描述**            |
| ------------- | ------------------- |
| 存储卷        | 存储卷的名称。         |

## 示例

示例1：显示StarRocks集群中的所有存储卷。

```Plain
MySQL > SHOW STORAGE VOLUMES;
+----------------+
| 存储卷         |
+----------------+
| my_s3_volume   |
+----------------+
1 row in set (0.01 sec)
```

## 相关的SQL语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [修改存储卷](./ALTER_STORAGE_VOLUME.md)
- [删除存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [存储卷描述](./DESC_STORAGE_VOLUME.md)