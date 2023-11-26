---
displayed_sidebar: "Japanese"
---

# sleep

## 説明

指定された時間（秒単位）だけ操作の実行を遅延させ、中断なしでスリープが完了したかどうかを示すBOOLEAN値を返します。スリープが中断せずに完了した場合、`1`が返されます。それ以外の場合、`0`が返されます。

## 構文

```Haskell
BOOLEAN sleep(INT x);
```

## パラメータ

`x`: 操作の実行を遅延させる期間です。INT型である必要があります。単位: 秒。入力がNULLの場合、即座にNULLが返されます。

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
