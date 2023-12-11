---
displayed_sidebar: "Japanese"
---

# microseconds_add

## Description

日付値に時間間隔を追加します。時間間隔はマイクロ秒単位です。

## Syntax

```Haskell
DATETIME microseconds_add(DATETIME expr1,INT expr2);
```

## Parameters

`expr1`: 時間式です。DATETIMEタイプでなければなりません。

`expr2`: 追加したい時間間隔、マイクロ秒単位で表されます。INTタイプでなければなりません。

## Return value

DATETIMEタイプの値を返します。入力値がDATEタイプの場合、時、分、秒の部分は `00:00:00` として処理されます。

## Examples

```Plain Text
select microseconds_add('2010-11-30 23:50:50', 2);
+--------------------------------------------+
| microseconds_add('2010-11-30 23:50:50', 2) |
+--------------------------------------------+
| 2010-11-30 23:50:50.000002                 |
+--------------------------------------------+
1 row in set (0.00 sec)

select microseconds_add('2010-11-30', 2);
+-----------------------------------+
| microseconds_add('2010-11-30', 2) |
+-----------------------------------+
| 2010-11-30 00:00:00.000002        |
+-----------------------------------+
```