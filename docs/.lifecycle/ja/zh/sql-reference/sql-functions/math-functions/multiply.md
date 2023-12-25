---
displayed_sidebar: Chinese
---

# 掛け算

## 機能

引数 `arg1` と `arg2` の積を計算します。

## 構文

```Haskell
MULTIPLY(arg1, arg2);
```

## 引数説明

`arg1`: 対応するデータ型は数値列またはリテラルです。
`arg2`: 対応するデータ型は数値列またはリテラルです。

## 戻り値の説明

戻り値のデータ型は `arg1` と `arg2` に依存します。

## 例

```Plain Text
MySQL > select multiply(10,2);
+-----------------+
| multiply(10, 2) |
+-----------------+
|              20 |
+-----------------+
1 row in set (0.01 sec)

MySQL > select multiply(1,2.1);
+------------------+
| multiply(1, 2.1) |
+------------------+
|              2.1 |
+------------------+
1 row in set (0.01 sec)

-- テーブルのデータに対して multiply を実行します。
MySQL > select * from t;
+------+------+------+------+
| id   | name | job1 | job2 |
+------+------+------+------+
|    2 |    2 |    2 |    2 |
+------+------+------+------+
1 row in set (0.08 sec)

MySQL > select multiply(1.0,id) from t;
+-------------------+
| multiply(1.0, id) |
+-------------------+
|                 2 |
+-------------------+
1 row in set (0.01 sec)
```
