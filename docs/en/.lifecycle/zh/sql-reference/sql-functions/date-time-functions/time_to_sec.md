---
displayed_sidebar: English
---

# time_to_sec

## 描述

将时间值转换为秒数。转换所使用的公式如下：

小时 x 3600 + 分钟 x 60 + 秒

## 语法

```Haskell
BIGINT time_to_sec(TIME time)
```

## 参数

`time`：它必须是TIME类型。

## 返回值

返回BIGINT类型的值。如果输入无效，返回NULL。

## 示例

```plain
select time_to_sec('12:13:14');
+-----------------------------+
| time_to_sec('12:13:14')     |
+-----------------------------+
|                        43994|
+-----------------------------+
```