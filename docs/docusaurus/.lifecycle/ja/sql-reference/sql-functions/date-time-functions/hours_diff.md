---
displayed_sidebar: "Japanese"
---

# hours_diff

## Description

2つの日付式（`expr1` − `expr2`）間の時間の差を、時間単位で正確に返します。

## Syntax

```Haskell
BIGINT hours_diff(DATETIME expr1, DATETIME expr2);
```

## Parameters

- `expr1`: 終了時刻。DATETIME型である必要があります。

- `expr2`: 開始時刻。DATETIME型である必要があります。

## Return value

BIGINT値を返します。

例えば、日付が存在しない場合はNULLが返されます。たとえば、2022-02-29です。

## Examples

```Plain
select hours_diff('2010-11-30 23:59:59', '2010-11-30 20:58:59');
+----------------------------------------------------------+
| hours_diff('2010-11-30 23:59:59', '2010-11-30 20:58:59') |
+----------------------------------------------------------+
|                                                        3 |
+----------------------------------------------------------+
```