---
displayed_sidebar: Chinese
---

# right

## 機能

文字列の右側から指定された長さの文字を返します。長さの単位は「UTF-8 文字」です。関数の別名は [strright](strright.md) です。

## 文法

```Haskell
VARCHAR right(VARCHAR str, INT len)
```

## パラメータ説明

`str`: 処理対象の文字列で、サポートされるデータ型は VARCHAR です。

`len`: 返す文字の長さで、サポートされるデータ型は INT です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
MySQL > select right("Hello starrocks",5);
+-------------------------+
| right('Hello starrocks', 5) |
+-------------------------+
| rocks                   |
+-------------------------+
```
