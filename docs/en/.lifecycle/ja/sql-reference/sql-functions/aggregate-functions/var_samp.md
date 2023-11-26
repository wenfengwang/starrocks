---
displayed_sidebar: "Japanese"
---

# VAR_SAMP,VARIANCE_SAMP

## 説明

式のサンプル分散を返します。v2.5.10以降、この関数はウィンドウ関数としても使用できます。

## 構文

```Haskell
VAR_SAMP(expr)
```

## パラメータ

`expr`: 式です。テーブルの列である場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

## 戻り値

DOUBLE値を返します。

## 例

```plaintext
MySQL > select var_samp(scan_rows)
from log_statis
group by datetime;
+-----------------------+
| var_samp(`scan_rows`) |
+-----------------------+
|    5.6227132145741789 |
+-----------------------+
```

## キーワード

VAR_SAMP,VARIANCE_SAMP,VAR,SAMP,VARIANCE
