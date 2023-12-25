---
displayed_sidebar: Chinese
---

# divide

## 機能

除算関数で、`x` を `y` で割った結果を返します。`y` が 0 の場合はnullを返します。

## 文法

```Haskell
divide(x, y);
```

## 引数説明

`x`: 対応するデータ型は DOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128 です。

`y`: 対応するデータ型は DOUBLE、FLOAT、LARGEINT、BIGINT、INT、SMALLINT、TINYINT、DECIMALV2、DECIMAL32、DECIMAL64、DECIMAL128 です。

## 戻り値の説明

戻り値のデータ型は DOUBLE です。

## 例

```Plain Text
mysql> select divide(3, 2);
+--------------+
| divide(3, 2) |
+--------------+
|          1.5 |
+--------------+
1 row in set (0.00 sec)
```
