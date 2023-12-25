---
displayed_sidebar: Chinese
---

# weeks_add

## 機能

指定された週数を加算した後の元の日付を返します。

## 文法

```Haskell
DATETIME weeks_add(DATETIME|DATE expr1, INT expr2)
```

## パラメータ説明

`expr1`: 元の日付で、サポートされているデータ型は DATETIME または DATE です。

`expr2`: 加算したい週数で、サポートされているデータ型は `INT` です。

## 戻り値の説明

戻り値のデータ型は DATETIME です。日付が存在しない場合は NULL を返します。

## 例

```Plain Text
select weeks_add('2022-12-20',2);
+----------------------------+
| weeks_add('2022-12-20', 2) |
+----------------------------+
|        2023-01-03 00:00:00 |
+----------------------------+
```
