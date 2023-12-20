---
displayed_sidebar: English
---

# uuid_numeric

## 描述

返回 LARGEINT 类型的随机 UUID。该函数的执行性能比 `uuid` 函数高出2个数量级。

## 语法

```Haskell
uuid_numeric();
```

## 参数

无

## 返回值

返回一个 LARGEINT 类型的值。

## 示例

```Plain
MySQL > select uuid_numeric();
+--------------------------+
| uuid_numeric()           |
+--------------------------+
| 558712445286367898661205 |
+--------------------------+
1 行在集合中 (0.00 秒)
```