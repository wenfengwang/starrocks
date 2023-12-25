---
displayed_sidebar: Chinese
---

# left

## 機能

文字列の左側から指定された長さの文字を返します。長さの単位は「UTF-8 文字」です。関数の別名は [strleft](strleft.md) です。

## 文法

```Haskell
VARCHAR left(VARCHAR str, INT len)
```

## 引数説明

`str`: 処理対象の文字列で、サポートされているデータ型は VARCHAR です。

`len`: 返すべき文字の長さで、サポートされているデータ型は INT です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
MySQL > select left("Hello starrocks",5);
+----------------------------+
| left('Hello starrocks', 5) |
+----------------------------+
| Hello                      |
+----------------------------+
```
