---
displayed_sidebar: Chinese
---

# url_encode

## 機能

文字列を [application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 形式でエンコードします。

この関数はv3.2バージョンからサポートされています。

## 文法

```haskell
VARCHAR url_encode(VARCHAR str)
```

## パラメータ説明

- `str`: エンコードする文字列。`str` が文字列形式でない場合、暗黙の型変換を試みます。

## 戻り値の説明

[application/x-www-form-urlencoded](https://www.w3.org/TR/html4/interact/forms.html#h-17.13.4.1) 形式に準拠したエンコードされた文字列を返します。

## 例

```plaintext
mysql> select url_encode('https://docs.starrocks.io/docs/introduction/StarRocks_intro/');
+----------------------------------------------------------------------------+
| url_encode('https://docs.starrocks.io/docs/introduction/StarRocks_intro/') |
+----------------------------------------------------------------------------+
| https%3A%2F%2Fdocs.starrocks.io%2Fdocs%2Fintroduction%2FStarRocks_intro%2F |
+----------------------------------------------------------------------------+
```
