```yaml
---
displayed_sidebar: "Japanese"
---

# weeks_sub

## 説明

日付から週数を減算した値を返します。

## 構文

```Haskell
DATETIME weeks_sub(DATETIME expr1, INT expr2);
```

## パラメーター

- `expr1`: 元の日付です。`DATETIME` 型でなければなりません。

- `expr2`: 週数です。`INT` 型でなければなりません。

## 返り値

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