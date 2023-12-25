---
displayed_sidebar: English
---

# space

## 説明

指定された数のスペースの文字列を返します。

## 構文

```Haskell
space(x);
```

## パラメーター

`x`: 返すスペースの数です。サポートされているデータ型は INT です。

## 戻り値

VARCHAR 型の値を返します。

## 例

```Plain Text
mysql> select space(6);
+----------+
| space(6) |
+----------+
|          |
+----------+
1 row in set (0.00 sec)
```

## キーワード

SPACE
