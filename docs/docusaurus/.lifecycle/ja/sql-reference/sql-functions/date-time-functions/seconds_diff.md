---
displayed_sidebar: "Japanese"
---

# seconds_diff

## Description

2つの日付式（`expr1` - `expr2`）間の秒差を秒単位で返します。

## Syntax

```Haskell
BIGINT seconds_diff(DATETIME expr1, DATETIME expr2);
```

## Parameters

- `expr1`: 終了時刻。DATETIME型である必要があります。

- `expr2`: 開始時刻。DATETIME型である必要があります。

## Return value

BIGINT値を返します。

例えば、2022-02-29のような存在しない日付の場合、NULLが返されます。

## Examples

```Plain
select seconds_diff('2010-11-30 23:59:59', '2010-11-30 20:59:59');
+------------------------------------------------------------+
| seconds_diff('2010-11-30 23:59:59', '2010-11-30 20:59:59') |
+------------------------------------------------------------+
|                                                      10800 |
+------------------------------------------------------------+
```