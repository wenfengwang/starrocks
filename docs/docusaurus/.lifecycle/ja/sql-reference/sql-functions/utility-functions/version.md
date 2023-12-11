---
displayed_sidebar: "Japanese"
---

# バージョン

## 説明

MySQLデータベースの現在のバージョンを返します。

StarRocksのバージョンをクエリするには[current_version](current_version.md)を使用できます。

## 構文

```Haskell
VARCHAR version();
```

## パラメーター

なし

## 戻り値

VARCHAR型の値を返します。

## 例

```Plain Text
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.1.0     |
+-----------+
1行がセットされました (0.00 秒)
```

## 参照

[current_version](../utility-functions/current_version.md)