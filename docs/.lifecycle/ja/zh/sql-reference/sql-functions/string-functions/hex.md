---
displayed_sidebar: Chinese
---

# hex

## 機能

引数 `x` が数値の場合は、その十六進数値の文字列表現を返します。引数 `x` が文字列の場合は、各文字を二つの十六進数の文字に変換し、変換後の全ての文字を連結して文字列を出力します。

## 文法

```Haskell
HEX(x);
```

## 引数説明

`x`: 対応するデータ型は BIGINT、VARCHAR、VARBINARY (v3.0 以降)。

## 戻り値説明

戻り値のデータ型は VARCHAR です。

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

-- 入力値が VARBINARY 型の場合。
mysql> select hex(x'abab');
+-------------+
| hex('ABAB') |
+-------------+
| ABAB        |
+-------------+
```
