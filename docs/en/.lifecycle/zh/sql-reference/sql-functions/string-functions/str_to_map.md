---
displayed_sidebar: English
---

# str_to_map

## 描述

使用两个分隔符将给定字符串拆分为键值对，并返回拆分后的键值对映射。

该函数从 v3.1 版本开始支持。

## 语法

```Haskell
MAP<VARCHAR, VARCHAR> str_to_map(VARCHAR content[, VARCHAR delimiter[, VARCHAR map_delimiter]])
```

## 参数

- `content`：必填，要拆分的字符串表达式。
- `delimiter`：可选，用于将`content`拆分成键值对的分隔符，默认为`,`。
- `map_delimiter`：可选，用于分隔每个键值对的分隔符，默认为`:`。

## 返回值

返回一个包含 STRING 元素的 MAP。任何空输入都将返回 NULL。

## 示例

```SQL
mysql> SELECT str_to_map('a:1|b:2|c:3', '|', ':') as map;
+---------------------------+
| map                       |
+---------------------------+
| {"a":"1","b":"2","c":"3"} |
+---------------------------+

mysql> SELECT str_to_map('a:1;b:2;c:3', ';', ':') as map;
+---------------------------+
| map                       |
+---------------------------+
| {"a":"1","b":"2","c":"3"} |
+---------------------------+

mysql> SELECT str_to_map('a:1,b:2,c:3', ',', ':') as map;
+---------------------------+
| map                       |
+---------------------------+
| {"a":"1","b":"2","c":"3"} |
+---------------------------+

mysql> SELECT str_to_map('a') as map;
+------------+
| map        |
+------------+
| {"a":null} |
+------------+

mysql> SELECT str_to_map('a:1,b:2,c:3',null, ':') as map;
+------+
| map  |
+------+
| NULL |
+------+

mysql> SELECT str_to_map('a:1,b:2,c:null') as map;
+------------------------------+
| map                          |
+------------------------------+
| {"a":"1","b":"2","c":"null"} |
+------------------------------+
```

## 关键字

STR_TO_MAP