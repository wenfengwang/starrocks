---
displayed_sidebar: Chinese
---

# from_base64

## 機能

Base64でエンコードされた文字列 `str` をデコードします。逆関数は [to_base64](to_base64.md) です。

## 文法

```Haskell
from_base64(str);
```

## パラメータ説明

`str`: デコードする文字列で、サポートされるデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。入力値が NULL または有効な Base64 文字列でない場合は、NULL を返します。

この関数は一つの文字列のみを受け取ります。複数の文字列を入力すると、エラーが返されます。

## 例

```Plain Text
mysql> select from_base64("starrocks");
+--------------------------+
| from_base64('starrocks') |
+--------------------------+
| ²֫®$                       |
+--------------------------+
1行がセットされました (0.00秒)
```
