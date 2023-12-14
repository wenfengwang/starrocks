---
displayed_sidebar: "Chinese"
---

# array_distinct

## 描述

从数组中移除重复的元素。

## 语法

```Haskell
array_distinct(array)
```

## 参数

`array`: 您要从中移除重复元素的数组。仅支持ARRAY数据类型。

## 返回值

返回一个数组。

## 使用注意事项

- 返回的数组元素的顺序可能与您指定的数组元素的顺序不同。

- 返回的数组元素与您指定的数组元素具有相同的数据类型。

## 示例

在本节中，使用以下表作为示例：

```plaintext
mysql> select * from test;

+------+---------------+

| c1   | c2            |

+------+---------------+

|    1 | [1,1,2]       |

|    2 | [1,null,null] |

|    3 | NULL          |

|    4 | [null]        |

+------+---------------+
```

从列`c2`中删除重复的值。

```plaintext
mysql> select c1, array_distinct(c2) from test;

+------+----------------------+

| c1   | array_distinct(`c2`) |

+------+----------------------+

|    1 | [2,1]                |

|    2 | [null,1]             |

|    3 | NULL                 |

|    4 | [null]               |

+------+----------------------+
```