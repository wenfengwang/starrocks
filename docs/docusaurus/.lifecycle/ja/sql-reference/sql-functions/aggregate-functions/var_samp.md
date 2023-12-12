---
displayed_sidebar: "Japanese"
---

# var_samp, variance_samp

## Description

式の標本分散を返します。v2.5.10以降、この関数はウィンドウ関数としても使用できます。

## Syntax

```Haskell
VAR_SAMP(expr)
```

## Parameters

`expr`: 式。テーブルの列の場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価する必要があります。

## Return value

DOUBLE値を返します。

## Examples

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

## keyword

VAR_SAMP, VARIANCE_SAMP, VAR, SAMP, VARIANCE