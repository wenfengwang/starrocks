```markdown
---
displayed_sidebar: "Japanese"
---

# if

## Description

`expr1` が TRUE を評価する場合、`expr2` を返します。それ以外の場合は、`expr3` を返します。

## Syntax

```Haskell
if(expr1, expr2, expr3);
```

## Parameters

`expr1`: 条件。BOOLEAN 値でなければなりません。

`expr2` および `expr3`: データ型が互換性がある必要があります。

## Return value

戻り値の型は `expr2` と同じです。

## Examples

```Plain Text
mysql> select if(true,1,2);
+----------------+
| if(TRUE, 1, 2) |
+----------------+
|              1 |
+----------------+

mysql> select if(false,2.14,2);
+--------------------+
| if(FALSE, 2.14, 2) |
+--------------------+
|               2.00 |
+--------------------+
```