```yaml
---
displayed_sidebar: "Chinese"
---

# JSON 运算符

StarRocks支持使用`<`、`<=`、`>`、`>=`、`=`、`!=`运算符查询JSON数据，不支持使用`IN`运算符。

> - 运算符两边必须均为JSON类型的数据。
> - 如果运算符一边是JSON类型的数据，另一边不是，则运算时会通过隐式类型转换，将不是JSON类型的数据转换为JSON类型的数据。类型转换规则，请参见[JSON类型转换](./json-query-and-processing-functions/cast.md)。

## 运算规则

JSON运算符遵循以下规则：

- 当运算符两边JSON数据的值属于相同的数据类型时
  - 如果为基本的数据类型（数字类型、字符串类型、布尔类型)，则运算时，遵循基本类型的运算规则。

   > 如果都是数值类型，但分别为DOUBLE和INT时，则会将INT转型成DOUBLE进行比较。

  - 如果为复合数据类型（对象类型、数组类型），则运算时，按照元素逐个比较：按key的字典序排序，再逐个比较key对应的value。

    比如，对于JSON对象`{"a": 1, "c": 2}`和`{"b": 1, "a": 2}`，按照运算符左侧JSON对象中键的字典序进行对比。对比节点a，由于左边的值1 < 右边的值2，因此`{"a": 1, "c": 2}` < `{"b": 1, "a": 2}`。

```Plain Text
mysql> SELECT PARSE_JSON('{"a": 1, "c": 2}') < PARSE_JSON('{"b": 1, "a": 2} ');
       -> 1
```

   对于JSON对象`{"a": 1, "c": 2}`和`{"b": 1, "a": 1}`，按照运算符左侧JSON对象中键的字典序进行对比。首先对比节点a，左右的值均为1。对比节点c，由于右侧不存在该值，因此`{"a": 1, "c": 2}` > `{"b": 1, "a": 1}`。

```Plain Text
mysql> SELECT PARSE_JSON('{"a": 1, "c": 2}') < PARSE_JSON('{"b": 1, "a": 1}');
       -> 0
```

- 当运算符两边JSON数据的值为不同的数据类型时，运算时，按照类型排序，进行比较。目前类型排序为NULL < BOOLEAN < ARRAY < OBJECT < DOUBLE < INT < STRING。

```Plain Text
mysql> SELECT PARSE_JSON('"a"') < PARSE_JSON('{"a": 1, "c": 2}');
       -> 0
```