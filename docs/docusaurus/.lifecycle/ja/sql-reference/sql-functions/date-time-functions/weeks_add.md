---
displayed_sidebar: "Japanese"
---

# weeks_add

## Description

日付に週数を追加した値を返します。

## Syntax

```Haskell
DATETIME weeks_add(DATETIME expr1, INT expr2);
```

## Parameters

- `expr1`: 元の日付。`DATETIME`型でなければなりません。

- `expr2`: 週数。`INT`型でなければなりません。

## Return value

`DATETIME`を返します。

日付が存在しない場合は`NULL`が返されます。

## Examples

```Plain
select weeks_add('2022-12-20',2);
+----------------------------+
| weeks_add('2022-12-20', 2) |
+----------------------------+
|        2023-01-03 00:00:00 |
+----------------------------+
```