---
displayed_sidebar: Chinese
---

# element_at

## 機能

Map 中の指定されたキー (Key) に対応する値 (Value) を取得します。入力値が NULL または指定された Key が存在しない場合は、NULL を返します。

Array データから特定の位置の要素を取得したい場合は、Array 関数の [element_at](../array-functions/element_at.md) を参照してください。

この関数はバージョン 3.0 からサポートされています。

## 文法

```Haskell
element_at(any_map, any_key)
```

## パラメータ説明

- `any_map`: Map 式。
- `key`: 指定された key。`key` が存在しない場合は NULL を返します。

## 戻り値の説明

`key` に対応する Value の値を返します。

## 例

```plain text
mysql> select element_at(map{1:3,2:4},1);
+-------------------------+
| element_at({1:3,2:4},1) |
+-------------------------+
|                     3   |
+-------------------------+

mysql> select element_at(map{1:3,2:4},3);
+-------------------------+
| element_at({1:3,2:4},3) |
+-------------------------+
|                    NULL |
+-------------------------+

mysql> select element_at(map{'a':1,'b':2},'a');
+-----------------------+
| map{'a':1,'b':2}['a'] |
+-----------------------+
|                     1 |
+-----------------------+
```

## キーワード

ELEMENT_AT, MAP
