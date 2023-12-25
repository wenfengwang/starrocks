---
displayed_sidebar: English
---

# var_samp,variance_samp

## 説明

式の標本分散を返します。v2.5.10以降、この関数はウィンドウ関数としても使用できるようになりました。

## 構文

```Haskell
VAR_SAMP(expr)
```

## パラメーター

`expr`: 式です。テーブルのカラムである場合、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、またはDECIMALに評価される必要があります。

## 戻り値

DOUBLE型の値を返します。

## 例

```plaintext
MySQL > SELECT var_samp(scan_rows)
FROM log_statis
GROUP BY datetime;
+-----------------------+
| var_samp(`scan_rows`) |
+-----------------------+
|    5.6227132145741789 |
+-----------------------+
```

## キーワード

VAR_SAMP,VARIANCE_SAMP,VAR,SAMP,VARIANCE
