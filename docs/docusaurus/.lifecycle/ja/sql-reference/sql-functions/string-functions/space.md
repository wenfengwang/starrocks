---
displayed_sidebar: "Japanese"
---

# スペース

## 説明

指定されたスペースの数の文字列を返します。

## 構文

```Haskell
space(x);
```

## パラメーター

`x`: 返されるスペースの数。サポートされるデータ型はINTです。

## 返り値

VARCHAR型の値を返します。

## 例

```Plain Text
mysql> select space(6);
+----------+
| space(6) |
+----------+
|          |
+----------+
1 行が選択されました (0.00 秒)
```

## キーワード

SPACE