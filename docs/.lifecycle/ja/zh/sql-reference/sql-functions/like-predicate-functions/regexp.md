---
displayed_sidebar: Chinese
---

# 正規表現

## 機能

対象の文字列 `expr` が指定された正規表現 `pattern` に一致するかどうかを判断します。一致した場合は 1 を返し、そうでない場合は 0 を返します。いずれかの入力パラメータが NULL の場合は、NULL を返します。

## 構文

```Haskell
BOOLEAN regexp(VARCHAR expr, VARCHAR pattern);
```

## パラメータ説明

`expr`: 対象の文字列で、サポートされるデータ型は VARCHAR です。

`pattern`: 正規表現、つまり文字列が一致する必要があるパターンで、サポートされるデータ型は VARCHAR です。

## 戻り値の説明

戻り値のデータ型は BOOLEAN です。

## 例

```Plain Text
mysql> select regexp("abc123","abc*");
+--------------------------+
| regexp('abc123', 'abc*') |
+--------------------------+
|                        1 |
+--------------------------+
1行がセットされました (0.06 秒)
```

## キーワード

regexp, regular
