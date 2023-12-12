---
displayed_sidebar: "Japanese"
---

# floor, dfloor

## 説明

`x` より大きくない最大の整数を返します。

## 構文

```SQL
FLOOR(x);
```

## パラメーター

`x`: DOUBLE がサポートされています。

## 戻り値

BIGINT データ型の値を返します。

## 例

```Plaintext
mysql> select floor(3.14);
+-------------+
| floor(3.14) |
+-------------+
|           3 |
+-------------+
1 行が選択されました (0.01 秒)
```