---
displayed_sidebar: "Japanese"
---

# ends_with

## 説明

指定された接尾辞で文字列が終わる場合は`true`を返します。それ以外の場合は`false`を返します。引数がNULLの場合、結果もNULLになります。

## 構文

```Haskell
BOOLEAN ENDS_WITH (VARCHAR str, VARCHAR suffix)
```

## 例

```Plain Text
MySQL > select ends_with("Hello starrocks", "starrocks");
+-----------------------------------+
| ends_with('Hello starrocks', 'starrocks') |
+-----------------------------------+
|                                 1 |
+-----------------------------------+

MySQL > select ends_with("Hello starrocks", "Hello");
+-----------------------------------+
| ends_with('Hello starrocks', 'Hello') |
+-----------------------------------+
|                                 0 |
+-----------------------------------+
```

## キーワード

ENDS_WITH
