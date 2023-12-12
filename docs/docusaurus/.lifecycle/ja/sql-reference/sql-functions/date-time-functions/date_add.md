```yaml
---
displayed_sidebar: "Japanese"
---

# date_add

## Description

指定された時間間隔を日付に追加します。

## Syntax

```Haskell
DATETIME DATE_ADD(DATETIME|DATE date, INTERVAL expr type)
```

## パラメータ

- `date`: 有効な日付または日時式でなければなりません。
- `expr`: 追加したい時間間隔です。INT型である必要があります。
- `type`: 時間間隔の単位です。次の値のいずれかにのみ設定できます: YEAR, MONTH, DAY, HOUR, MINUTE, SECOND。

## Return value

DATETIME 値を返します。たとえば、`2020-02-30` のような日付が存在しない場合、NULL が返されます。日付が DATE 値の場合、DATETIME 値に変換されます。

## Examples

```Plain Text
select date_add('2010-11-30 23:59:59', INTERVAL 2 DAY);
+-------------------------------------------------+
| date_add('2010-11-30 23:59:59', INTERVAL 2 DAY) |
+-------------------------------------------------+
| 2010-12-02 23:59:59                             |
+-------------------------------------------------+

select date_add('2010-12-03', INTERVAL 2 DAY);
+----------------------------------------+
| date_add('2010-12-03', INTERVAL 2 DAY) |
+----------------------------------------+
| 2010-12-05 00:00:00                    |
+----------------------------------------+
```