---
displayed_sidebar: Chinese
---

# strleft

## 機能

文字列の左側から指定された長さの文字を返します。長さの単位は「UTF-8 文字」です。関数の別名は [left](left.md) です。

## 構文

```Haskell
VARCHAR strleft(VARCHAR str, INT len)
```

## 引数説明

`str`: 処理対象の文字列で、サポートされるデータ型は VARCHAR です。

`len`: 返すべき文字の長さで、サポートされるデータ型は INT です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
MySQL > select strleft("Hello starrocks",5);
+-----------------------------+
|strleft('Hello starrocks', 5)|
+-----------------------------+
| Hello                       |
+-----------------------------+
```
