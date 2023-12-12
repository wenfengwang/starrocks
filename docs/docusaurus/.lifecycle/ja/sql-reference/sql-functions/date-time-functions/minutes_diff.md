---
displayed_sidebar: "Japanese"
---

# minutes_diff

## Description

日付式（`expr1` - `expr2`）の間の分の差を分単位で返します。

## Syntax

```Haskell
BIGINT minutes_diff(DATETIME expr1,DATETIME expr2);
```

## Parameters

- `expr1`: 終了時刻。DATETIME型である必要があります。

- `expr2`: 開始時刻。DATETIME型である必要があります。

## Return value

BIGINT値を返します。

たとえば、2022-02-29のような日付が存在しない場合、NULLが返されます。

## Examples

```Plain
select minutes_diff('2010-11-30 23:59:59', '2010-11-30 20:58:59');
+------------------------------------------------------------+
| minutes_diff('2010-11-30 23:59:59', '2010-11-30 20:58:59') |
+------------------------------------------------------------+
|                                                        181 |
+------------------------------------------------------------+
```