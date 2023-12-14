```yaml
---
displayed_sidebar: "Chinese"
---

# 分钟

## 功能

获取日期中的分钟信息，返回值范围为 0~59。

参数为 DATE 或者 DATETIME 类型。

## 语法

```Haskell
INT MINUTE(DSATETIME date)
```

## 示例

```Plain Text
select minute('2018-12-31 23:59:59');
+-----------------------------+
|minute('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```