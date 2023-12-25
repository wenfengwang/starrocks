---
displayed_sidebar: Chinese
---

# weeks_sub

## 機能

指定された日付から数週間を差し引いた日付を返します。

## 構文

```Haskell
DATETIME weeks_sub(DATETIME|DATE expr1, INT expr2)
```

## 引数説明

`expr1`: 元の日付で、サポートされるデータ型は DATETIME または DATE です。DATE 型の入力は DATETIME に暗黙的に変換されます。

`expr2`: 差し引かれる週の数で、データ型は `INT` です。

## 戻り値の説明

戻り値のデータ型は `DATETIME` です。存在しない日付の場合は NULL を返します。

## 例

```Plain Text
select weeks_sub('2022-12-22',2);
+----------------------------+
| weeks_sub('2022-12-22', 2) |
+----------------------------+
|        2022-12-08 00:00:00 |
+----------------------------+
```
