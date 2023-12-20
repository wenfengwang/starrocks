---
displayed_sidebar: English
---

# host_name

## 描述

获取执行计算的节点的主机名。

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
1行在集合中 (0.01秒)
```