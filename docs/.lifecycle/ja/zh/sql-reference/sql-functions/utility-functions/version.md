---
displayed_sidebar: Chinese
---

# バージョン

## 機能

現在の MySQL データベースのバージョンを返します。`current_version` 関数を使用して StarRocks の現在のバージョンを照会できます。

## 文法

```Haskell
version();
```

## 引数説明

なし。この関数はいかなる引数も受け付けません。

## 戻り値の説明

VARCHAR 型の値を返します。

## 例

```Plain Text
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.1.0     |
+-----------+
1行がセットされました (0.00秒)
```

## 関連文書

[current_version](./current_version.md)
