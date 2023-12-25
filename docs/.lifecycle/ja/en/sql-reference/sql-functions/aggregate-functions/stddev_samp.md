---
displayed_sidebar: English
---

# stddev_samp

## 説明

式の標本標準偏差を返します。v2.5.10以降、この関数はウィンドウ関数としても使用できます。

## 構文

```Haskell
STDDEV_SAMP(expr)
```

## パラメーター

`expr`: 式です。テーブル列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

## 戻り値

DOUBLE型の値を返します。

## 例

```plain text
MySQL > select stddev_samp(scan_rows)
from log_statis
group by datetime;
+--------------------------+
| stddev_samp(`scan_rows`) |
+--------------------------+
|        2.372044195280762 |
+--------------------------+
```

## キーワード

STDDEV_SAMP,STDDEV,SAMP
