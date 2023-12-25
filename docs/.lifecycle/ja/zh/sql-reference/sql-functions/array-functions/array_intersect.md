---
displayed_sidebar: Chinese
---

# array_intersect

## 機能

複数の同じ型の配列に対して、交差部分を返します。

## 文法

```Haskell
output array_intersect(input0, input1, ...)
```

## パラメータ説明

* input：数量は無制限（1つ以上）で、同じ要素型を持つ配列(input0, input1, ...)。具体的な型は任意です。

## 戻り値の説明

型はArray（要素の型はinputに含まれる配列の要素の型と一致）、内容はすべての入力配列(input0, input1, ...)の交差部分です。

## 例

**例1**:

```plain text
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", "query", "SQL"], ["SQL"])
AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| ["SQL"]      |
+--------------+
```

**例2**:

```plain text
mysql> SELECT array_intersect(["SQL", "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| []           |
+--------------+
```

**例3**:

```plain text
mysql> SELECT array_intersect(["SQL", null, "storage"], ["mysql", null], [null]) AS no_intersect ;
+--------------+
| no_intersect |
+--------------+
| [null]       |
+--------------+
```
