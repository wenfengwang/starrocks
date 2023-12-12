```yaml
---
displayed_sidebar: "English"
---

# hex_decode_string

## Description

この関数は、[hex()](hex.md)の逆の操作を行います。

入力文字列内の各16進数のペアを数値として解釈し、その数値によって表されるバイトに変換します。戻り値はバイナリ文字列です。

この関数はv3.0からサポートされています。

## Syntax

```Haskell
hex_decode_string(str);
```

## Parameters

`str`: 変換する文字列。サポートされるデータ型はVARCHARです。以下の状況が発生した場合、空の文字列が返されます。

- 文字列の長さが0であるか、文字列内の文字数が奇数であるか。
- 文字列に`[0-9]`、`[a-z]`、`[A-Z]`以外の文字が含まれているか。

## Return value

VARCHAR型の値を返します。

## Examples

```Plain Text
mysql> select hex_decode_string(hex("Hello StarRocks"));
+-------------------------------------------+
| hex_decode_string(hex('Hello StarRocks')) |
+-------------------------------------------+
| Hello StarRocks                           |
+-------------------------------------------+
```

## Keywords

HEX_DECODE_STRING
```