---
displayed_sidebar: "Japanese"
---

# COVAR_POP

## 説明

2つの式の母集団共分散を返します。この関数はv2.5.10からサポートされています。ウィンドウ関数としても使用することができます。

## 構文

```Haskell
COVAR_POP(expr1, expr2)
```

## パラメータ

`expr1`と`expr2`はTINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

`expr1`と`expr2`がテーブルの列である場合、この関数はこれら2つの列の母集団共分散を計算します。

## 戻り値

DOUBLE値を返します。以下の式が適用されます。ここで、`n`はテーブルの行数を表します。

![covar_pop formula](../../../assets/covar_pop_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n}
$$-->

## 使用上の注意

- データ行は、この行の2つの列が非NULL値である場合にのみカウントされます。それ以外の場合、このデータ行は結果から除外されます。

- どれかの入力がNULLの場合、NULLが返されます。

## 例

テーブル`agg`に次のデータがあるとします：

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

`k`列と`v`列の母集団共分散を計算します：

```plaintext
mysql> select no,COVAR_POP(k,v) from agg group by no;
+------+-------------------+
| no   | covar_pop(k, v)   |
+------+-------------------+
|    1 |              NULL |
|    2 | 79.99999999999999 |
+------+-------------------+
```
