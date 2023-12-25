---
displayed_sidebar: Chinese
---

# stddev_samp

## 機能

`expr` 式の標本標準偏差を返します。バージョン 2.5.10 から、この関数はウィンドウ関数としても使用できます。

## 文法

```Haskell
STDDEV_SAMP(expr)
```

## パラメータ説明

`expr`: 選択された式。式がテーブルの列の場合、次のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL。

## 戻り値の説明

戻り値は DOUBLE 型です。

## 例

```plaintext
MySQL > select stddev_samp(scan_rows)
from log_statis
group by datetime;
+--------------------------+
| stddev_samp(`scan_rows`) |
+--------------------------+
|        2.372044195280762 |
+--------------------------+
```
