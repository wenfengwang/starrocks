---
displayed_sidebar: Chinese
---

# any_value

## 機能

`GROUP BY` を含む集約クエリで、各集約グループから**ランダムに**1行を選択して返すために使用される関数です。

## 文法

```Haskell
ANY_VALUE(expr)
```

## パラメータ説明

`expr`: 選択される式です。

## 戻り値の説明

各集約されたグループから**ランダムに**行を選択して結果を返しますが、結果は不定です。

## 例

以下のようなデータがテーブルにあるとします。

```plain text
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

ANY_VALUE を使用した結果は以下の通りです。

```plain text
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
