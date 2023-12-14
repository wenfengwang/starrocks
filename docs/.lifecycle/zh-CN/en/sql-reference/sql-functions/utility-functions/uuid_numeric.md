---
displayed_sidebar: "Chinese"
---

# uuid_numeric

## 描述

返回一个大整型类型的随机UUID。该函数的执行性能比`uuid`函数好2个数量级。

## 语法

```Haskell
uuid_numeric();
```

## 参数

无

## 返回值

返回一个大整型类型的值。

## 例子

```Plain Text
MySQL > select uuid_numeric();
+--------------------------+
| uuid_numeric()           |
+--------------------------+
| 558712445286367898661205 |
+--------------------------+
1 行受影响 (0.00 秒)
```