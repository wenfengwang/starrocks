---
displayed_sidebar: Chinese
---

# space

## 機能

指定された数のスペースで構成される文字列を返します。

## 文法

```Haskell
space(x);
```

## 引数の説明

`x`: サポートされるデータ型は INT です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

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
