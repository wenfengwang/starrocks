---
displayed_sidebar: English
---

# weeks_sub

## 説明

指定された日付から週数を減算した値を返します。

## 構文

```Haskell
DATETIME weeks_sub(DATETIME expr1, INT expr2);
```

## パラメーター

- `expr1`: 元の日付。`DATETIME` 型でなければなりません。

- `expr2`: 減算する週数。`INT` 型でなければなりません。

## 戻り値

`DATETIME` を返します。

日付が存在しない場合は `NULL` が返されます。

## 例

```Plain
select weeks_sub('2022-12-22',2);
+----------------------------+
| weeks_sub('2022-12-22', 2) |
+----------------------------+
|        2022-12-08 00:00:00 |
+----------------------------+
```
