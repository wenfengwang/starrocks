---
displayed_sidebar: "Japanese"
---

# char 

## 説明

CHAR（）は、与えられた整数値に対応する文字値をASCIIテーブルに従って返します。

## 構文

```Haskell
char(n)
```

## パラメータ

- `n`: 整数値

## 戻り値

VARCHAR値を返します。

## 例

```Plain Text
> select char(77);
+----------+
| char(77) |
+----------+
| M        |
+----------+
```

## キーワード

CHAR