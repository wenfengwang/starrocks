---
displayed_sidebar: Chinese
---

# std

## 機能

`expr` 式の標準偏差を返します。バージョン 2.5.10 から、この関数はウィンドウ関数としても使用できます。

## 文法

```Haskell
STD(expr)
```

## パラメータ説明

`expr`: 選択された式。式がテーブルの列である場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL。

## 戻り値の説明

DOUBLE 型の値を返します。

## 例

サンプルデータセット：

```plaintext
MySQL [test]> select * from std_test;
+------+------+
| col0 | col1 |
+------+------+
|    0 |    0 |
|    1 |    2 |
|    2 |    4 |
|    3 |    6 |
|    4 |    8 |
+------+------+
```

`col0` と `col1` の標準偏差を計算します。

```plaintext
MySQL > select std(col0) as std_of_col0, std(col1) as std_of_col1 from std_test;
+--------------------+--------------------+
| std_of_col0        | std_of_col1        |
+--------------------+--------------------+
| 1.4142135623730951 | 2.8284271247461903 |
+--------------------+--------------------+
```

## キーワード

STD
