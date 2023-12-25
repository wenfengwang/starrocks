---
displayed_sidebar: English
---

# covar_pop

## 説明

2つの式の母集団共分散を返します。この関数はv2.5.10からサポートされており、ウィンドウ関数としても使用できます。

## 構文

```Haskell
COVAR_POP(expr1, expr2)
```

## パラメーター

`expr1` と `expr2` は、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

`expr1` と `expr2` がテーブルの列である場合、この関数はこれら2つの列の母集団共分散を計算します。

## 戻り値

DOUBLE 値を返します。以下の式によって計算されます。ここで `n` はテーブルの行数を表します。

![covar_pop formula](../../../assets/covar_pop_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n}
$$-->

## 使用上の注意

- 2つの列が非NULL値のデータ行のみがカウントされます。NULL値がある行は結果から除外されます。

- いずれかの入力がNULLの場合、結果としてNULLが返されます。

## 例

`agg` テーブルに次のデータがあるとします。

```plaintext
mysql> select * from agg;
+------+-------+-------+
| no   | k     | v     |
+------+-------+-------+
|    1 | 10.00 |  NULL |
|    2 | 10.00 | 11.00 |
|    2 | 20.00 | 22.00 |
|    2 | 25.00 |  NULL |
|    2 | 30.00 | 35.00 |
+------+-------+-------+
```

`k` と `v` 列の母集団共分散を計算します：

```plaintext
mysql> select no,COVAR_POP(k,v) from agg group by no;
+------+-------------------+
| no   | COVAR_POP(k, v)   |
+------+-------------------+
|    1 |              NULL |
|    2 | 79.99999999999999 |
+------+-------------------+
```
