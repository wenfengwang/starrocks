---
displayed_sidebar: English
---

# any_value

## 説明

集約された各グループから任意の行を取得します。この関数は、`GROUP BY` 句を含むクエリを最適化するために使用できます。

## 構文

```Haskell
ANY_VALUE(expr)
```

## パラメーター

`expr`: 集約される式です。

## 戻り値

集約された各グループから任意の行を返します。戻り値は非決定的です。

## 例

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

// ANY_VALUEを使用した後
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

## キーワード

ANY_VALUE
