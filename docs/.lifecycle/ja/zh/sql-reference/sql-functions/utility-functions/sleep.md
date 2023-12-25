---
displayed_sidebar: Chinese
---

# sleep

## 機能

現在実行中のスレッドを `x` 秒間スリープさせます。BOOLEAN 型の値を返し、`1` は正常にスリープしたことを示し、`0` はスリープに失敗したことを示します。

## 文法

```Haskell
BOOLEAN sleep(INT x);
```

## パラメータ説明

`x`: 対応するデータ型は INT です。

## 戻り値の説明

BOOLEAN 型の値を返します。

## 例

```Plain Text
select sleep(3);
+----------+
| sleep(3) |
+----------+
|        1 |
+----------+
1行がセットされました (3.00 秒)

select sleep(NULL);
+-------------+
| sleep(NULL) |
+-------------+
|        NULL |
+-------------+
1行がセットされました (0.00 秒)
```
