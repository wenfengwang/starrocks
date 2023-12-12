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

## パラメータ

`x`: 返すスペースの数です。サポートされているデータ型はINTです。

## 戻り値

VARCHAR型の値を返します。

## 例

```Plain Text
mysql> select space(6);
+----------+
| space(6) |
+----------+
|          |
+----------+
1行が選択されました（0.00秒）
```

## キーワード

SPACE