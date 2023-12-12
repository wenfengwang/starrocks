---
displayed_sidebar: "Japanese"
---

# percentile_approx

## 説明

pの値が0から1の間にある場合、p番目のパーセンタイルの近似値を返します。

圧縮パラメータはオプションで、設定範囲は[2048、10000]です。値が大きいほど精度が高く、メモリの消費量が大きくなり、計算時間もかかります。指定されていない場合、または[2048、10000]の範囲外の場合、関数はデフォルトの圧縮パラメータ10000で実行されます。

この関数は固定サイズのメモリを使用するため、高い基数を持つ列のメモリを少なく使用でき、tp99などの統計を計算するために使用できます。

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