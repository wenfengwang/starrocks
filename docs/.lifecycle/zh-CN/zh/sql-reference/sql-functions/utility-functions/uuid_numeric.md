---
displayed_sidebar: "中文"
---

# uuid_numeric

## 功能

返回一个数值型的随机 UUID 值。与`uuid`函数相比，此函数的执行性能提高了将近两个数量级。

## 语法

```Haskell
uuid_numeric();
```

## 参数说明

无

## 返回值说明

返回 LARGEINT 类型的值。

## 示例

```Plain Text
MySQL > select uuid_numeric();
+--------------------------+
| uuid_numeric()           |
+--------------------------+
| 558712445286367898661205 |
+--------------------------+
1 行在集合中 (0.00 秒)
```