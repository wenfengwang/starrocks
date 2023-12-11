---
displayed_sidebar: "Japanese"
---

# percentile_approx

## 説明

pが0から1の間の値であるpthパーセンタイルの近似値を返します。

圧縮パラメータはオプションで、[2048、10000]の範囲で設定できます。値が大きいほど精度が高くなり、メモリ消費量が多くなり、計算時間も長くなります。指定されていないか、[2048、10000]の範囲を超えていない場合、関数はデフォルトの圧縮パラメータ10000で実行されます。

この関数は固定サイズのメモリを使用するため、高いカーディナリティを持つ列では少ないメモリを使用し、tp99などの統計を計算するために使用できます。

## 構文

```Haskell
PERCENTILE_APPROX(expr、DOUBLE p[, DOUBLE compression])
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

PERCENTILE_APPROX, PERCENTILE, APPROX