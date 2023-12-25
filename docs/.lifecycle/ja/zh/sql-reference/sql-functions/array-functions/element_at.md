---
displayed_sidebar: Chinese
---

# element_at

## 機能

Array 配列中の指定された位置の要素を取得します。入力値が NULL または指定された位置が存在しない場合は、NULL を返します。

この関数は配列の添字 `[]` 演算子の別名です。

MAP データから特定の Key に対応する Value を検索する場合は、Map 関数の [element_at](../map-functions/element_at.md) を参照してください。

この関数はバージョン 3.0 からサポートされています。

## 文法

```Haskell
element_at(any_array, position)
```

## パラメータ説明

- `any_array`: ARRAY 式。
- `position`: 配列の添字で、1 から始まります。正の整数でなければならず、配列の長さを超える値を取ることはできません。`position` が存在しない場合は、NULL を返します。

## 戻り値の説明

`position` に対応する位置の要素を返します。

## 例

```plain text
mysql> select element_at([2,3,11],3);
+-----------------------+
|  element_at([11,2,3]) |
+-----------------------+
|                    11 |
+-----------------------+

mysql> select element_at(["a","b","c"],1);
+--------------------+
| ['a', 'b', 'c'][1] |
+--------------------+
| a                  |
+--------------------+
```

## キーワード

ELEMENT_AT, ARRAY
