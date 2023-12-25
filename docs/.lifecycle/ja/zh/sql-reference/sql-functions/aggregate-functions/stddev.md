---
displayed_sidebar: Chinese
---


# stddev、stddev_pop、std

## 機能

`expr` 式の母集団標準偏差を返します。バージョン 2.5.10 から、この関数はウィンドウ関数としても使用できます。

### 文法

```Haskell
STDDEV(expr)
```

## パラメータ説明

`expr`: 選択された式。式がテーブルの列の場合、以下のデータ型をサポートしています: TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL。

## 戻り値の説明

戻り値は DOUBLE 型です。

## 例

```plaintext
mysql> SELECT stddev(lo_quantity), stddev_pop(lo_quantity) from lineorder;
+---------------------+-------------------------+
| stddev(lo_quantity) | stddev_pop(lo_quantity) |
+---------------------+-------------------------+
|   14.43100708360797 |       14.43100708360797 |
+---------------------+-------------------------+
```
