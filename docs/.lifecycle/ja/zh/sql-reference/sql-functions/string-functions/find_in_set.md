---
displayed_sidebar: Chinese
---

# find_in_set

## 機能

`strlist` 中で `str` が初めて現れる位置を返します（1から数え始めます）。見つからない場合は 0 を返し、いずれかの引数が NULL の場合は NULL を返します。

## 文法

```Haskell
find_in_set(str, strlist)
```

## 引数説明

`str`: 対応するデータ型は VARCHAR です。

`strlist`: コンマで区切られた文字列で、対応するデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は INT です。

## 例

```Plain Text
MySQL > select find_in_set("b", "a,b,c");
+---------------------------+
| find_in_set('b', 'a,b,c') |
+---------------------------+
|                         2 |
+---------------------------+
```
