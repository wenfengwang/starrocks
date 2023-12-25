---
displayed_sidebar: Chinese
---

# concat

## 機能

複数の文字列を連結します。引数のいずれかが NULL の場合、結果は NULL となります。

## 文法

```Haskell
concat(str,...)
```

## 引数説明

`str`: 対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

## 例

```Plain Text
MySQL > select concat("a", "b");
+------------------+
| concat('a', 'b') |
+------------------+
| ab               |
+------------------+

MySQL > select concat("a", "b", "c");
+-----------------------+
| concat('a', 'b', 'c') |
+-----------------------+
| abc                   |
+-----------------------+

MySQL > select concat("a", null, "c");
+------------------------+
| concat('a', NULL, 'c') |
+------------------------+
| NULL                   |
+------------------------+
```
