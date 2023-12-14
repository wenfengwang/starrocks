---
displayed_sidebar: "Chinese"
---

# strcmp

## 描述

此函数用于比较两个字符串。如果 lhs 和 rhs 相等，则返回 0。如果 lhs 在字典顺序中出现在 rhs 之前，则返回 -1。如果 lhs 在字典顺序中出现在 rhs 之后，则返回 1。当参数为 NULL 时，结果也为 NULL。

## 语法

```Haskell
INT strcmp(VARCHAR lhs, VARCHAR rhs)
```

## 示例

```Plain Text
mysql> select strcmp("test1", "test1");
+--------------------------+
| strcmp('test1', 'test1') |
+--------------------------+
|                        0 |
+--------------------------+

mysql> select strcmp("test1", "test2");
+--------------------------+
| strcmp('test1', 'test2') |
+--------------------------+
|                       -1 |
+--------------------------+

mysql> select strcmp("test2", "test1");
+--------------------------+
| strcmp('test2', 'test1') |
+--------------------------+
|                        1 |
+--------------------------+

mysql> select strcmp("test1", NULL);
+-----------------------+
| strcmp('test1', NULL) |
+-----------------------+
|                  NULL |
+-----------------------+
```

## 关键词

STRCMP