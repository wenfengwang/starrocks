---
displayed_sidebar: "Chinese"
---

# 显示存储卷

## 功能

显示当前 StarRocks 集群中的存储卷。该功能自 v3.1 起支持。

## 语法

```SQL
SHOW STORAGE VOLUMES [ LIKE '<pattern>' ]
```

## 参数说明

| **参数** | **说明**               |
| -------- | ---------------------- |
| pattern  | 用于匹配存储卷的模式。 |

## 返回

| **返回**       | **说明**       |
| -------------- | -------------- |
| Storage Volume | 存储卷的名称。 |

## 示例

示例一：显示当前 StarRocks 集群中所有的存储卷。

```SQL
MySQL > SHOW STORAGE VOLUMES;
+----------------+
| Storage Volume |
+----------------+
| my_s3_volume   |
+----------------+
1 row in set (0.01 sec)
```

## 相关 SQL

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [修改存储卷](./ALTER_STORAGE_VOLUME.md)
- [删除存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [存储卷描述](./DESC_STORAGE_VOLUME.md)