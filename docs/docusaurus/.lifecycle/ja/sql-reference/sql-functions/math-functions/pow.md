---
displayed_sidebar: "英語"
---

# pow, power, dpow, fpow

## 説明

`x` の `y` 乗の結果を返します。

## 構文

```Haskell
POW(x,y);POWER(x,y);
```

## パラメーター

`x`: DOUBLE データ型をサポートします。

`y`: DOUBLE データ型をサポートします。

## 戻り値

DOUBLE データ型の値を返します。

## 例

```Plain
mysql> select pow(2,2);
+-----------+
| pow(2, 2) |
+-----------+
|         4 |
+-----------+
1 行がセット (0.00 秒)

mysql> select power(4,3);
+-------------+
| power(4, 3) |
+-------------+
|          64 |
+-------------+
1 行がセット (0.00 秒)
```