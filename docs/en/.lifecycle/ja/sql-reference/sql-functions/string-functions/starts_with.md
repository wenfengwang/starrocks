---
displayed_sidebar: "Japanese"
---

# starts_with

## 説明

この関数は、指定された接頭辞で文字列が始まる場合には1を返します。それ以外の場合は0を返します。引数がNULLの場合、結果はNULLです。

## 構文

```Haskell
BOOLEAN starts_with(VARCHAR str, VARCHAR prefix)
```

## 例

```Plain Text
mysql> select starts_with("hello world","hello");
+-------------------------------------+
|starts_with('hello world', 'hello')  |
+-------------------------------------+
| 1                                   |
+-------------------------------------+

mysql> select starts_with("hello world","world");
+-------------------------------------+
|starts_with('hello world', 'world')  |
+-------------------------------------+
| 0                                   |
+-------------------------------------+
```

## キーワード

START_WITH
