---
displayed_sidebar: English
---

# pow、power、dpow、fpow

## 説明

`x` を `y` の累乗にした結果を返します。

## 構文

```Haskell
POW(x,y); POWER(x,y);
```

## パラメーター

`x`: DOUBLE データ型をサポート。

`y`: DOUBLE データ型をサポート。

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
1 row in set (0.00 sec)

mysql> select power(4,3);
+-------------+
| power(4, 3) |
+-------------+
|          64 |
+-------------+
1 row in set (0.00 sec)
```
