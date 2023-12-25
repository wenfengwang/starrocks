---
displayed_sidebar: English
---

# covar_samp

## 説明

2つの式の標本共分散を返します。この関数はv2.5.10からサポートされています。ウィンドウ関数としても使用可能です。

## 構文

```Haskell
COVAR_SAMP(expr1, expr2)
```

## パラメーター

`expr1` と `expr2` は、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、または DECIMAL に評価される必要があります。

`expr1` と `expr2` がテーブルの列である場合、この関数はこれら2つの列の標本共分散を計算します。

## 戻り値

DOUBLE 値を返します。以下の式によって計算されます。ここで `n` はテーブルの行数を表します。

![covar_samp formula](../../../assets/covar_samp_formula.png)

<!--$$
\frac{\sum_{i=1}^{n} (x_i - \bar{x})(y_i - \bar{y})}{n-1}
$$-->

## 使用上の注意

- 2つの列が非NULL値である行のみがカウントされます。それ以外の場合、その行は結果から除外されます。

- `n` が1の場合、0が返されます。

- いずれかの入力がNULLの場合、NULLが返されます。

## 例

テーブル `agg` に以下のデータがあるとします。

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

`k` と `v` の列の標本共分散を計算します：

```plaintext
mysql> select no,COVAR_SAMP(k,v) from agg group by no;
+------+--------------------+
| no   | COVAR_SAMP(k, v)   |
+------+--------------------+
|    1 |               NULL |
|    2 | 119.99999999999999 |
+------+--------------------+
```
