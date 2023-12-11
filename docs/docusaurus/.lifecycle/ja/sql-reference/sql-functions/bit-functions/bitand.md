```yaml
---
displayed_sidebar: "Japanese"
---

# ビットアンド

## 説明

2つの数値式のビットごとの論理積を返します。

## 構文

```Haskell
BITAND(x,y);
```

## パラメーター

- `x`: この式は次のいずれかのデータ型に評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

- `y`: この式は次のいずれかのデータ型に評価される必要があります：TINYINT、SMALLINT、INT、BIGINT、LARGEINT。

> `x` と `y` はデータ型で一致している必要があります。

## 戻り値

戻り値の型は `x` と同じです。どちらかの値がNULLの場合、結果もNULLです。

## 例

```Plain Text
mysql> select bitand(3,0);
+--------------+
| bitand(3, 0) |
+--------------+
|            0 |
+--------------+
1 row in set (0.01 sec)
```