---
displayed_sidebar: Chinese
---

# ceil、dceil

## 機能

`x` 以上で最小の整数を返します。

## 文法

```Haskell
CEIL(x);
```

## パラメータ説明

`x`: 対応するデータ型は DOUBLE です。

## 戻り値の説明

戻り値のデータ型は BIGINT です。

## 例

```Plain Text
mysql> select ceil(3.14);
+------------+
| ceil(3.14) |
+------------+
|          4 |
+------------+
1行がセットされました (0.15秒)
```
