---
displayed_sidebar: Chinese
---

# url_decode

## 機能

文字列を [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 形式からデコードします。逆の関数は [url_encode](./url_encode.md) です。

この関数は v3.2 バージョンからサポートされています。

## 文法

```haskell
VARCHAR url_decode(VARCHAR str)
```

## パラメータ説明

- `str`：デコードする文字列。`str` が文字列形式でない場合、暗黙の型変換を試みます。

## 戻り値の説明

デコードされた文字列を返します。入力文字列が application/x-www-form-urlencoded 形式に準拠していない場合、この関数はエラーを返します。

## 例

```plaintext
mysql> select url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F');
+------------------------------------------------------------------------------------------+
| url_decode('https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F') |
+------------------------------------------------------------------------------------------+
| https://docs.starrocks.io/docs/introduction/StarRocks_intro/                             |
+------------------------------------------------------------------------------------------+
```
