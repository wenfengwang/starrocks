---
displayed_sidebar: "Japanese"
---

# sleep（スリープ）

## Description（説明）

指定された期間（秒単位）操作の実行を遅らせ、スリープが中断されない場合はBOOLEAN値を返します。スリープが中断されなかった場合は`1`が返されます。それ以外の場合は`0`が返されます。

## Syntax（構文）

```Haskell
BOOLEAN sleep(INT x);
```

## Parameters（パラメータ）

`x`: 操作の実行を遅らせたい期間です。INT型でなければなりません。単位：秒。入力がNULLの場合、すぐにNULLが返されます。

## Return value（戻り値）

BOOLEAN型の値が返されます。

## Examples（例）

```Plain Text
select sleep(3);
+----------+
| sleep(3) |
+----------+
|        1 |
+----------+
1 行が返されました (3.00 秒)

select sleep(NULL);
+-------------+
| sleep(NULL) |
+-------------+
|        NULL |
+-------------+
1 行が返されました (0.00 秒)
```

## Keywords（キーワード）

SLEEP, sleep