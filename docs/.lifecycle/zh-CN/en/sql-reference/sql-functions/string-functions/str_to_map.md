---
displayed_sidebar: "Chinese"
---

# str_to_map

## 描述

使用两个分隔符将给定的字符串拆分为键值对，并返回拆分后的键值对的映射。

此函数从v3.1版本开始受支持。

## 语法

```Haskell
MAP<VARCHAR, VARCHAR> str_to_map(VARCHAR content[, VARCHAR delimiter[, VARCHAR map_delimiter]])
```

## 参数

- `content`：必需，要拆分的字符串表达式。
- `delimiter`：可选，用于将 `content` 拆分为键值对的分隔符，默认为 `,`。
- `map_delimiter`：可选，用于分隔每个键值对的分隔符，默认为 `:`。

## 返回值

返回一个STRING元素的MAP。任何空输入都会返回NULL。

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

mysql> SELECT str_to_map('a:1,b:2,c:null',null, ':') as map;
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

## 关键词

STR_TO_MAP