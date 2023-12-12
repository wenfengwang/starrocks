```sql
---
displayed_sidebar: "Japanese"
---

# left

## 説明

この関数は、指定された文字列から左側の指定された数の文字を返します。長さの単位：utf8 文字。
注意：この関数は [strleft](strleft.md) としても呼び出すことができます。

## 構文

```SQL
VARCHAR left(VARCHAR str,INT len)
```

## 例

```SQL
MySQL > select left("Hello starrocks",5);
+----------------------------+
| left('Hello starrocks', 5) |
+----------------------------+
| Hello                      |
+----------------------------+
```

## キーワード

LEFT
```