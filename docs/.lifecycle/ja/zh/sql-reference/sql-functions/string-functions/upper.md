---
displayed_sidebar: Chinese
---

# upper

## 機能

文字列を大文字に変換します。

## 文法

```haskell
upper(str)
```

## 引数説明

- `str`：変換する文字列。`str` が文字列型でない場合は、暗黙の型変換を試みてから文字列に変換し、その後でこの関数を実行します。

## 戻り値の説明

大文字に変換された文字列を返します。

## 例

```plaintext
MySQL [test]> select C_String, upper(C_String) from ex_iceberg_tbl;
+---------------+-----------------+
| C_String      | upper(C_String) |
+---------------+-----------------+
| Hello, China! | HELLO, CHINA!   |
| Hello, World! | HELLO, WORLD!   |
+---------------+-----------------+
```
