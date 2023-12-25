---
displayed_sidebar: Chinese
---

# aes_encrypt

## 機能

AES_128_ECB アルゴリズムを使用して文字列 `str` を暗号化し、バイナリ文字列を返します。

AES は Advanced Encryption Standard の略です。ECB は Electronic Code Book の略で、電子コードブックモードを指します。このアルゴリズムは 128ビットのキーを使用して文字列を暗号化します。

## 文法

```Haskell
aes_encrypt(str,key_str);
```

## パラメータ説明

`str`: 暗号化する対象の文字列で、サポートされるデータ型は VARCHAR です。

`key_str`: `str` を暗号化するためのキー文字列で、サポートされるデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。入力値が NULL の場合は、NULL を返します。

## 例

文字列 `starrocks` を AES で暗号化し、Base64 文字列に変換します。

```Plain Text

mysql> select to_base64(AES_ENCRYPT('starrocks','F3229A0B371ED2D9441B830D21A390C3'));
+-------------------------------------------------------------------------+
| to_base64(aes_encrypt('starrocks', 'F3229A0B371ED2D9441B830D21A390C3')) |
+-------------------------------------------------------------------------+
| uv/Lhzm74syo8JlfWarwKA==                                                |
+-------------------------------------------------------------------------+
1 row in set (0.01 sec)
```
