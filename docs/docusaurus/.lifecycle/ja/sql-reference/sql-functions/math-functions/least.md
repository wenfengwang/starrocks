---
displayed_sidebar: "Japanese"
---

# least

## 説明

1つ以上のパラメータからなるリストの中で最小の値を返します。

一般的に、戻り値のデータ型は入力と同じです。

比較ルールは[greatest](greatest.md) 関数と同じです。

## 構文

```Haskell
LEAST(expr1,...);
```

## パラメータ

`expr1`: 比較する式。以下のデータ型がサポートされています:

- SMALLINT

- TINYINT

- INT

- BIGINT

- LARGEINT

- FLOAT

- DOUBLE

- DECIMALV2

- DECIMAL32

- DECIMAL64

- DECIMAL128

- DATETIME

- VARCHAR

## 例

Example 1: 単一の入力値の最小値を返す。

```Plain
select least(3);
+----------+
| least(3) |
+----------+
|        3 |
+----------+
1 row in set (0.00 sec)
```

Example 2: 値のリストから最小値を返す。

```Plain
select least(3,4,5,5,6);
+----------------------+
| least(3, 4, 5, 5, 6) |
+----------------------+
|                    3 |
+----------------------+
1 row in set (0.01 sec)
```

Example 3: 1つのパラメータがDOUBLE型でDOUBLEの値が返される。

```Plain
select least(4,4.5,5.5);
+--------------------+
| least(4, 4.5, 5.5) |
+--------------------+
|                4.0 |
+--------------------+
```

Example 4: 入力パラメータが数字と文字列の混合だが、文字列は数値に変換できる。パラメータは数字として比較される。

```Plain
select least(7,'5');
+---------------+
| least(7, '5') |
+---------------+
| 5             |
+---------------+
1 row in set (0.01 sec)
```

Example 5: 入力パラメータが数字と文字列の混合だが、文字列は数値に変換できない。パラメータは文字列として比較される。文字列 '1' は 'at' よりも小さい。

```Plain
select least(1,'at');
+----------------+
| least(1, 'at') |
+----------------+
| 1              |
+----------------+
```

Example 6: 入力パラメータは文字です。

```Plain
mysql> select least('A','B','Z');
+----------------------+
| least('A', 'B', 'Z') |
+----------------------+
| A                    |
+----------------------+
1 row in set (0.00 sec)
```

## キーワード

LEAST, least