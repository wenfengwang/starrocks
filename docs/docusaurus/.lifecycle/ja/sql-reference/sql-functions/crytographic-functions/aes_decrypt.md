---
displayed_sidebar: "Japanese"
---

# aes_decrypt

## 説明

AES_128_ECBアルゴリズムを使用して文字列を復号し、バイナリ文字列を返します。

AESはadvanced encryption standardの略で、ECBはelectronic code bookの略です。文字列を暗号化するために使用される鍵は128ビットの文字列です。

## 構文

```Haskell
aes_decrypt(str, key_str);
```

## パラメータ

`str`: 復号する文字列。VARCHAR型である必要があります。

`key_str`:  `str`を暗号化するために使用される鍵。VARCHAR型である必要があります。

## 戻り値

VARCHAR型の値を返します。入力が無効な場合は、NULLが返されます。

## 例

Base64文字列をデコードし、この関数を使用してデコードした文字列を元の文字列に復号します。

```Plain Text
mysql> select AES_DECRYPT(from_base64('uv/Lhzm74syo8JlfWarwKA==  '),'F3229A0B371ED2D9441B830D21A390C3');
+--------------------------------------------------------------------------------------------+
| aes_decrypt(from_base64('uv/Lhzm74syo8JlfWarwKA==  '), 'F3229A0B371ED2D9441B830D21A390C3') |
+--------------------------------------------------------------------------------------------+
| starrocks                                                                                  |
+--------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```