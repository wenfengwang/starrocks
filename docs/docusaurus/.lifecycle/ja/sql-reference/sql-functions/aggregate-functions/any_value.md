---
displayed_sidebar: "Japanese"
---

# any_value

## Description

各集計グループから任意の行を取得します。この関数は、`GROUP BY` 句を持つクエリを最適化するために使用できます。

## Syntax

```Haskell
ANY_VALUE(expr)
```

## Parameters

`expr`: 集計される式。

## Return value

各集計グループから任意の行を返します。返り値は非決定的です。

## Examples

```plaintext
// 元のデータ
mysql> select * from any_value_test;
+------+------+------+
| a    | b    | c    |
+------+------+------+
|    1 |    1 |    1 |
|    1 |    2 |    1 |
|    2 |    1 |    1 |
|    2 |    2 |    2 |
|    3 |    1 |    1 |
+------+------+------+
5 rows in set (0.01 sec)

// ANY_VALUE の使用後
mysql> select a,any_value(b),sum(c) from any_value_test group by a;
+------+----------------+----------+
| a    | any_value(`b`) | sum(`c`) |
+------+----------------+----------+
|    1 |              1 |        2 |
|    2 |              1 |        3 |
|    3 |              1 |        1 |
+------+----------------+----------+
3 rows in set (0.01 sec)

mysql> select c,any_value(a),sum(b) from any_value_test group by c;
+------+----------------+----------+
| c    | any_value(`a`) | sum(`b`) |
+------+----------------+----------+
|    1 |              1 |        5 |
|    2 |              2 |        2 |
+------+----------------+----------+
2 rows in set (0.01 sec)
```

## Keywords

ANY_VALUE