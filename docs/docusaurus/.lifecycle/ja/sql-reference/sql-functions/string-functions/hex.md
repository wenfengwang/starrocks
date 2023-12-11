---
displayed_sidebar: "Japanese"
---

# hex（16進数）

## 説明

`x` が数値の場合、この関数はその値の16進数表現の文字列を返します。

`x` が文字列の場合、この関数は文字列内の各文字が2桁の16進数に変換された16進数表現の文字列を返します。

## 構文

```Haskell
HEX(x);
```

## パラメーター

`x`: 変換する文字列または数値。サポートされているデータ型はBIGINT、VARCHAR、VARBINARYです（v3.0以降）。

## 戻り値

VARCHAR 型の値を返します。

## 例

```Plain Text
mysql> select hex(3);
+--------+
| hex(3) |
+--------+
| 3      |
+--------+
1 row in set (0.00 sec)

mysql> select hex('3');
+----------+
| hex('3') |
+----------+
| 33       |
+----------+
1 row in set (0.00 sec)

mysql> select hex('apple');
+--------------+
| hex('apple') |
+--------------+
| 6170706C65   |
+--------------+

-- 入力がバイナリ値である場合。

mysql> select hex(x'abab');
+-------------+
| hex('ABAB') |
+-------------+
| ABAB        |
+-------------+
1 row in set (0.01 sec)
```

## キーワード

HEX