---
displayed_sidebar: English
---

# aes_encrypt

## 描述

使用 AES_128_ECB 算法对字符串进行加密，并返回一个二进制字符串。

AES 是高级加密标准的缩写，ECB 是电子密码本的缩写。用于加密字符串的密钥是一个 128 位的字符串。

## 语法

```Haskell
aes_encrypt(str, key_str);
```

## 参数

`str`：要加密的字符串。它必须是 VARCHAR 类型。

`key_str`：用于加密 `str` 的密钥。它必须是 VARCHAR 类型。

## 返回值

返回一个 VARCHAR 类型的值。如果输入为 NULL，则返回 NULL。

## 例子

使用此函数对 `starrocks` 进行 AES 加密，并将加密后的字符串转换为 Base64 编码的字符串。

```Plain Text
mysql> select to_base64(AES_ENCRYPT('starrocks','F3229A0B371ED2D9441B830D21A390C3'));
+-------------------------------------------------------------------------+
| to_base64(aes_encrypt('starrocks', 'F3229A0B371ED2D9441B830D21A390C3')) |
+-------------------------------------------------------------------------+
| uv/Lhzm74syo8JlfWarwKA==                                                |
+-------------------------------------------------------------------------+
1 行受影响 (0.01 秒)