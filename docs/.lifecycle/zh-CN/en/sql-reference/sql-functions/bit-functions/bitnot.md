```markdown
---
displayed_sidebar: "Chinese"
---

# bitnot

## Description

返回一个数值表达式的按位取反结果。

## Syntax

```Haskell
BITNOT(x);
```

## Parameters

`x`: 这个表达式必须能够求值为以下任一数据类型：TINYINT, SMALLINT, INT, BIGINT, LARGEINT。

## Return value

返回值的类型与 `x` 相同。如果任何值为 NULL，则结果也为 NULL。

## Examples

```Plain Text
mysql> select bitnot(3);
+-----------+
| bitnot(3) |
+-----------+
|        -4 |
+-----------+
```