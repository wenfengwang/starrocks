---
displayed_sidebar: English
---

# sleep

## 説明

指定された期間（秒単位）の操作の実行を遅延させ、スリープが中断されずに完了したかどうかを示すBOOLEAN値を返します。`1`はスリープが中断されずに完了した場合に返されます。それ以外の場合は`0`が返されます。

## 構文

```Haskell
BOOLEAN sleep(INT x);
```

## パラメーター

`x`: 操作の実行を遅延させたい期間です。INT型でなければなりません。単位は秒です。入力がNULLの場合は、スリープすることなくすぐにNULLが返されます。

## 戻り値

BOOLEAN型の値を返します。

## 例

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

## キーワード

SLEEP, sleep
