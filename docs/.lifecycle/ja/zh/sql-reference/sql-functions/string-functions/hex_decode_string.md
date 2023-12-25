---
displayed_sidebar: Chinese
---

# hex_decode_string

## 機能

入力された文字列内の各十六進数のペアを数字に解析し、その数字を表すバイトに変換してから、バイナリ文字列として返します。この関数は [hex()](../../../sql-reference/sql-functions/string-functions/hex.md) 関数の逆関数です。

この関数はバージョン 3.0 からサポートされています。

## 文法

```Haskell
VARCHAR hex_decode_string(VARCHAR str);
```

## パラメータ説明

`str`：デコードする文字列で、VARCHAR 型である必要があります。

以下のいずれかの状況が発生した場合、空文字列を返します：

- 入力文字列の長さが 0 であるか、または入力文字列内の文字の数が奇数である。
- 入力文字列が [0-9]、[a-f]、[A-F] 以外の文字を含む。

## 戻り値の説明

VARCHAR 型の値を返します。

## 例

```Plain
mysql> select hex_decode_string(hex("Hello StarRocks"));
+-------------------------------------------+
| hex_decode_string(hex('Hello StarRocks')) |
+-------------------------------------------+
| Hello StarRocks                           |
+-------------------------------------------+
```

## キーワード

HEX_DECODE_STRING
