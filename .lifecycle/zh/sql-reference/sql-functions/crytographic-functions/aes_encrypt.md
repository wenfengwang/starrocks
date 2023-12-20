---
displayed_sidebar: English
---

# AES加密

## 描述

采用AES_128_ECB算法对字符串进行加密，并返回一个二进制字符串。

AES代表高级加密标准，ECB代表电子密码本。用于加密字符串的密钥是一个128位的字符串。

## 语法

```Haskell
aes_encrypt(str,key_str);
```

## 参数

str：需要加密的字符串。必须是VARCHAR类型。

key_str：用于加密str的密钥。必须是VARCHAR类型。

## 返回值

返回VARCHAR类型的值。如果输入值为NULL，也返回NULL。

## 示例

使用此函数对starrocks进行AES加密，并将加密字符串转换为Base64编码字符串。

```Plain
mysql> select to_base64(AES_ENCRYPT('starrocks','F3229A0B371ED2D9441B830D21A390C3'));
+-------------------------------------------------------------------------+
| to_base64(aes_encrypt('starrocks', 'F3229A0B371ED2D9441B830D21A390C3')) |
+-------------------------------------------------------------------------+
| uv/Lhzm74syo8JlfWarwKA==                                                |
+-------------------------------------------------------------------------+
1 row in set (0.01 sec)
```
