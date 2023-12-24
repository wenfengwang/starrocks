---
displayed_sidebar: English
---

# array_distinct

## 描述

从数组中删除重复的元素。

## 语法

```Haskell
array_distinct(array)
```

## 参数

`array`：要从中删除重复元素的数组。仅支持 ARRAY 数据类型。

## 返回值

返回一个数组。

## 使用说明

- 返回的数组可能按不同顺序排序，与指定的数组元素顺序不同。

- 返回的数组元素与指定的数组元素的数据类型相同。

## 例子

在本节中，使用下表作为示例：

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

从列 `c2` 中删除重复值。

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