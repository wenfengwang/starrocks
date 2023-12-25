---
displayed_sidebar: Chinese
---

# hex_decode_binary

## 機能

十六進数でエンコードされた文字列をVARBINARY型の値にデコードします。

この関数はバージョン3.0からサポートされています。

## 文法

```Haskell
VARBINARY hex_decode_binary(VARCHAR str);
```

## パラメータ説明

`str`：デコードする文字列で、VARCHAR型でなければなりません。

以下のいずれかの場合、BINARY型のNULL値が返されます：

- 入力文字列の長さが0、または入力文字列の文字数が奇数です。
- 入力文字列に[0-9]、[a-f]、[A-F]以外の文字が含まれています。

## 戻り値の説明

VARBINARY型の値を返します。

## 例

```Plain
mysql> select hex(hex_decode_binary(hex("Hello StarRocks")));
+------------------------------------------------+
| hex(hex_decode_binary(hex('Hello StarRocks'))) |
+------------------------------------------------+
| 48656C6C6F2053746172526F636B73                 |
+------------------------------------------------+

mysql> select hex_decode_binary(NULL);
+--------------------------------------------------+
| hex_decode_binary(NULL)                          |
+--------------------------------------------------+
| NULL                                             |
+--------------------------------------------------+
```

## キーワード

HEX_DECODE_BINARY
