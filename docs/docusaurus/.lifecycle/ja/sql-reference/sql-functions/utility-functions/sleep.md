---
displayed_sidebar: "Japanese"
---

# sleep

## Description

指定された時間（秒単位）の操作の実行を遅延させ、sleepが中断されずに完了したことを示すBOOLEAN値を返します。sleepが中断されずに完了した場合は`1`が返されます。それ以外の場合は`0`が返されます。

## Syntax

```Haskell
BOOLEAN sleep(INT x);
```

## Parameters

`x`: 操作の実行を遅延させたい時間。INT型である必要があります。単位：秒。入力がNULLの場合、即座にNULLが返されます。

## Return value

BOOLEAN型の値を返します。

## Examples

```Plain Text
select sleep(3);
+----------+
| sleep(3) |
+----------+
|        1 |
+----------+
1 row in set (3.00 sec)

select sleep(NULL);
+-------------+
| sleep(NULL) |
+-------------+
|        NULL |
+-------------+
1 row in set (0.00 sec)
```

## Keywords

SLEEP, sleep