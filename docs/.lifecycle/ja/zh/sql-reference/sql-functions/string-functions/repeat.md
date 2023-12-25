---
displayed_sidebar: Chinese
---

# repeat

## 機能

文字列をcount回繰り返して出力します。countが1未満の場合は空文字列を返します。strまたはcountがNULLの場合は、NULLを返します。

## 文法

```Haskell
repeat(str, count)
```

## 引数説明

`str`: 対応するデータ型はVARCHARです。

`count`: 対応するデータ型はINTです。

## 戻り値説明

戻り値のデータ型はVARCHARです。

## 例

```Plain Text
MySQL > SELECT repeat("a", 3);
+----------------+
| repeat('a', 3) |
+----------------+
| aaa            |
+----------------+

MySQL > SELECT repeat("a", -1);
+-----------------+
| repeat('a', -1) |
+-----------------+
|                 |
+-----------------+
```
