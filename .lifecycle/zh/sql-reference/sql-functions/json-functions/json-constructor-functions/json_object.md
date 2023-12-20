---
displayed_sidebar: English
---

# json_object

## 描述

此函数可将一个或多个键值对转换成包含这些键值对的JSON对象。这些键值对会按照字典顺序进行键的排序。

## 语法

```Haskell
json_object(key, value, ...)
```

## 参数

- key：JSON对象中的键。只支持VARCHAR数据类型。

- value：JSON对象中的值。仅支持NULL值以及以下数据类型：STRING、VARCHAR、CHAR、JSON、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、DOUBLE、FLOAT和BOOLEAN。

## 返回值

返回一个JSON对象。

> 如果键和值的总数量是奇数，那么JSON_OBJECT函数会在最后一个字段填充NULL。

## 示例

示例1：构建一个包含不同数据类型值的JSON对象。

```plaintext
mysql> SELECT json_object('name', 'starrocks', 'active', true, 'published', 2020);

       -> {"active": true, "name": "starrocks", "published": 2020}            
```

示例2：使用嵌套的JSON_OBJECT函数来构建一个JSON对象。

```plaintext
mysql> SELECT json_object('k1', 1, 'k2', json_object('k2', 2), 'k3', json_array(4, 5));

       -> {"k1": 1, "k2": {"k2": 2}, "k3": [4, 5]} 
```

示例3：创建一个空的JSON对象。

```plaintext
mysql> SELECT json_object();

       -> {}
```
