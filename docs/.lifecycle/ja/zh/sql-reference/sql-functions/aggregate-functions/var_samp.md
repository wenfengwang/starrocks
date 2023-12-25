---
displayed_sidebar: Chinese
---


# var_samp、variance_samp

## 機能

`expr` 式の標本分散を返します。バージョン 2.5.10 から、この関数はウィンドウ関数としても使用できます。

## 文法

```Haskell
VAR_SAMP(expr)
```

## パラメータ説明

`expr`: 選択された式。式がテーブルの列の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL。

## 戻り値の説明

戻り値は DOUBLE 型です。

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
