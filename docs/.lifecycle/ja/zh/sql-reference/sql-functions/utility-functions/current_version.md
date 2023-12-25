---
displayed_sidebar: Chinese
---

# current_version

## 機能

現在のStarRocksのバージョンを取得します。異なるクライアントに対応するため、二つの構文を提供しています。

## 文法

```Haskell
current_version();

@@version_comment;
```

## パラメータ説明

なし

## 戻り値の説明

戻り値のデータ型はVARCHARです。

## 例

```Plain Text
mysql> select current_version();
+-------------------+
| current_version() |
+-------------------+
| 2.1.2 0782ad7     |
+-------------------+
1行がセットされました (0.00秒)

mysql> select @@version_comment;
+-------------------------+
| @@version_comment       |
+-------------------------+
| StarRocks version 2.1.2 |
+-------------------------+
1行がセットされました (0.01秒)
```
