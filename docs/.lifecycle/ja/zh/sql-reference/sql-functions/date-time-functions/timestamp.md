---
displayed_sidebar: Chinese
---

# タイムスタンプ

## 機能

時間表現 `expr` を DATETIME 値に変換します。

## 文法

```Haskell
DATETIME timestamp(DATETIME|DATE expr);
```

## 引数説明

`expr`: 変換する日付または日時の値で、サポートされるデータ型は DATETIME または DATE です。

## 戻り値の説明

戻り値のデータ型は DATETIME です。入力された日付が空または存在しない場合、例えば 2021-02-29 のような場合は、NULL を返します。

## 例

```Plain Text
select timestamp("2019-05-27");
+-------------------------+
| timestamp('2019-05-27') |
+-------------------------+
| 2019-05-27 00:00:00     |
+-------------------------+
1 row in set (0.00 sec)
```
