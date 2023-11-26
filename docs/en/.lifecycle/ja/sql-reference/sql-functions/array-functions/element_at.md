---
displayed_sidebar: "Japanese"
---

# element_at

## 説明

指定された位置（インデックス）の要素を与えられた配列から返します。どのパラメータがNULLであるか、または位置が存在しない場合、結果はNULLです。

この関数は、サブスクリプト演算子`[]`のエイリアスです。v3.0以降でサポートされています。

マップ内のキーと値のペアから値を取得したい場合は、[element_at](../map-functions/element_at.md)を参照してください。

## 構文

```Haskell
element_at(any_array, position)
```

## パラメータ

- `any_array`: 要素を取得する配列式です。
- `position`: 配列内の要素の位置です。正の整数である必要があります。値の範囲: [1, 配列の長さ]。`position`が存在しない場合、NULLが返されます。

## 例

```plain text
mysql> select element_at([2,3,11],3);
+-----------------------+
|  element_at([11,2,3]) |
+-----------------------+
|                     11 |
+-----------------------+
1 行が返されました (0.00 秒)
```

## キーワード

ELEMENT_AT, ARRAY
