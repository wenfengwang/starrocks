---
displayed_sidebar: English
---

# HEX

## 説明

`x` が数値である場合、この関数はその値の16進数の文字列表現を返します。

`x` が文字列である場合、この関数は文字列内の各文字を2桁の16進数に変換した文字列の16進数表現を返します。

## 構文

```Haskell
HEX(x);
```

## パラメーター

`x`: 変換する文字列または数値です。サポートされているデータ型はBIGINT、VARCHAR、およびVARBINARY（v3.0以降）です。

## 戻り値

VARCHAR型の値を返します。

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

-- 入力がバイナリ値です。

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
