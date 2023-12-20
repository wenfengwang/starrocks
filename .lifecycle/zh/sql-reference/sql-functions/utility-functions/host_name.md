---
displayed_sidebar: English
---

# 主机名

## 描述

获取执行计算节点的主机名称。

## 语法

```Haskell
host_name();
```

## 参数

无

## 返回值

返回一个 VARCHAR 类型的值。

## 示例

```Plaintext
select host_name();
+-------------+
| host_name() |
+-------------+
| sandbox-sql |
+-------------+
1 row in set (0.01 sec)
```
