---
displayed_sidebar: "Japanese"
---

# PERCENTILE_APPROX

## 説明

pの値が0から1の間である場合、p番目のパーセンタイルの近似値を返します。

圧縮パラメータはオプションで、設定範囲は[2048, 10000]です。値が大きいほど精度が高くなり、メモリ消費量が大きくなり、計算時間が長くなります。指定されていない場合や[2048, 10000]の範囲外の場合、関数はデフォルトの圧縮パラメータ10000で実行されます。

この関数は固定サイズのメモリを使用するため、高い基数を持つ列ではより少ないメモリを使用することができ、tp99などの統計情報の計算に使用することができます。

## 構文

```Haskell
PERCENTILE_APPROX(expr, DOUBLE p[, DOUBLE compression])
```

## 例

```plain text
MySQL > select `table`, percentile_approx(cost_time,0.99)
from log_statis
group by `table`;
+----------+--------------------------------------+
| table    | percentile_approx(`cost_time`, 0.99) |
+----------+--------------------------------------+
| test     |                                54.22 |
+----------+--------------------------------------+

MySQL > select `table`, percentile_approx(cost_time,0.99, 4096)
from log_statis
group by `table`;
+----------+----------------------------------------------+
| table    | percentile_approx(`cost_time`, 0.99, 4096.0) |
+----------+----------------------------------------------+
| test     |                                        54.21 |
+----------+----------------------------------------------+
```

## キーワード

PERCENTILE_APPROX,PERCENTILE,APPROX
