```yaml
---
displayed_sidebar: "Japanese"
---

# array_intersect

## Description

1つ以上の配列の共通部分の要素からなる配列を返します。

## Syntax

```Haskell
array_intersect(input0, input1, ...)
```

## Parameters

`input`: 取得したい共通部分を持つ1つ以上の配列です。`(input0, input1, ...)`の形式で配列を指定し、指定する配列が同じデータ型であることを確認してください。

## Return value

指定した配列と同じデータ型の配列を返します。

## Examples

Example 1:

```Plain
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", "query", "SQL"], ["SQL"])
AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| ["SQL"]      |
+--------------+
```

Example 2:

```Plain
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| []           |
+--------------+
```

Example 3:

```Plain
mysql> SELECT array_intersect(["SQL", null, "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| [null]       |
+--------------+
```