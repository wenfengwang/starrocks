---
displayed_sidebar: English
---

# JSON 运算符

StarRocks 支持以下 JSON 比较运算符：<、<=、>、>=、= 和 !=。您可以使用这些运算符来查询 JSON 数据。但是，StarRocks 不允许您使用 `IN` 来查询 JSON 数据。

    > 运算符的操作数必须都是 JSON 值。

    > 如果运算符的一个操作数是 JSON 值，而另一个不是，则在算术运算过程中非 JSON 值的操作数会被转换为 JSON 值。有关转换规则的更多信息，请参见 [CAST](./json-query-and-processing-functions/cast.md)。

## 算术规则

JSON 运算符遵循以下算术规则：

- 当运算符的操作数为相同数据类型的 JSON 值时：
  - 如果两个操作数都是基础数据类型的 JSON 值，例如 NUMBER、STRING 或 BOOLEAN，运算符将按照基础数据类型的算术规则执行运算。

> 注意：如果两个操作数都是数字，但一个是 DOUBLE 类型，另一个是 INT 类型，运算符会将 INT 类型转换为 DOUBLE 类型。

- 如果两个操作数都是复合数据类型的 JSON 值，例如 OBJECT 或 ARRAY，运算符会根据第一个操作数中键的顺序，按字典序对键进行排序，然后比较两个操作数中键的值。

示例 1：

第一个操作数是 `{"a": 1, "c": 2}`，第二个操作数是 `{"b": 1, "a": 2}`。在此示例中，运算符比较操作数之间键 `a` 的值。第一个操作数中键 `a` 的值为 `1`，而第二个操作数中键 `a` 的值为 `2`。值 `1` 小于值 `2`。因此，运算符得出结论：第一个操作数 `{"a": 1, "c": 2}` 小于第二个操作数 `{"b": 1, "a": 2}`。

```plaintext
mysql> SELECT PARSE_JSON('{"a": 1, "c": 2}') < PARSE_JSON('{"b": 1, "a": 2}');

       -> 1
```

示例 2：

第一个操作数是 `{"a": 1, "c": 2}`，第二个操作数是 `{"b": 1, "a": 1}`。在此示例中，运算符首先比较操作数之间键 `a` 的值。两个操作数中键 `a` 的值都是 `1`。然后，运算符比较操作数中键 `c` 的值。第二个操作数不包含键 `c`。因此，运算符得出结论：第一个操作数 `{"a": 1, "c": 2}` 大于第二个操作数 `{"b": 1, "a": 1}`。

```plaintext
mysql> SELECT PARSE_JSON('{"a": 1, "c": 2}') > PARSE_JSON('{"b": 1, "a": 1}');

       -> 1
```

- 当运算符的操作数为两种不同数据类型的 JSON 值时，运算符将按照以下算术规则比较操作数：NULL < BOOLEAN < ARRAY < OBJECT < DOUBLE < INT < STRING。

```plaintext
mysql> SELECT PARSE_JSON('"a"') < PARSE_JSON('{"a": 1, "c": 2}');

       -> 1
```