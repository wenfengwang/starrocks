---
displayed_sidebar: English
---

# hex_decode_string

## 説明

この関数は [hex()](hex.md) の逆の操作を行います。

入力文字列内の16進数の各ペアを数値として解釈し、その数値が表すバイトに変換します。戻り値はバイナリ文字列です。

この関数はv3.0からサポートされています。

## 構文

```Haskell
hex_decode_string(str);
```

## パラメーター

`str`: 変換する文字列。サポートされるデータ型はVARCHARです。以下のいずれかの状況が発生した場合、空文字列が返されます。

- 文字列の長さが0、または文字数が奇数の場合。
- 文字列に `[0-9]`、`[a-z]`、`[A-Z]` 以外の文字が含まれている場合。

## 戻り値

VARCHAR型の値を返します。

## 例

```Plain Text
mysql> select hex_decode_string(hex("Hello StarRocks"));
+-------------------------------------------+
| hex_decode_string(hex('Hello StarRocks')) |
+-------------------------------------------+
| Hello StarRocks                           |
+-------------------------------------------+
```

## キーワード

HEX_DECODE_STRING
