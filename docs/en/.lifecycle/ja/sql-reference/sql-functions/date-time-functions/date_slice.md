---
displayed_sidebar: "Japanese"
---

# date_slice

## 説明

指定された時間を、指定された時間粒度に基づいて時間間隔の始まりまたは終わりに変換します。

この関数はv2.5からサポートされています。

## 構文

```Haskell
DATE date_slice(DATE dt, INTERVAL N type[, boundary])
```

## パラメータ

- `dt`: 変換する時間、DATE型。
- `INTERVAL N type`: 時間粒度、例えば `interval 5 day`。
  - `N` は時間間隔の長さです。INT値である必要があります。
  - `type` は単位で、YEAR、QUARTER、MONTH、WEEK、DAY のいずれかです。DATE値に対して `type` を HOUR、MINUTE、または SECOND に設定すると、エラーが返されます。
- `boundary`: オプションです。時間間隔の始まり (`FLOOR`) または終わり (`CEIL`) を返すかを指定するために使用されます。有効な値: FLOOR、CEIL。このパラメータが指定されていない場合、デフォルトは `FLOOR` です。

## 戻り値

DATE型の値を返します。

## 使用上の注意

時間間隔は西暦 `0001-01-01 00:00:00` から始まります。

## 例

例1: `boundary` パラメータを指定せずに、指定された時間を5年間隔の始まりに変換します。

```Plaintext
select date_slice('2022-04-26', interval 5 year);
+--------------------------------------------------+
| date_slice('2022-04-26', INTERVAL 5 year, floor) |
+--------------------------------------------------+
| 2021-01-01                                       |
+--------------------------------------------------+
```

例2: 指定された時間を5日間隔の終わりに変換します。

```Plaintext
select date_slice('0001-01-07', interval 5 day, CEIL);
+------------------------------------------------+
| date_slice('0001-01-07', INTERVAL 5 day, ceil) |
+------------------------------------------------+
| 0001-01-11                                     |
+------------------------------------------------+
```

以下の例は、`test_all_type_select` テーブルを基にしています。

```Plaintext
select * from test_all_type_select order by id_int;
+------------+---------------------+--------+
| id_date    | id_datetime         | id_int |
+------------+---------------------+--------+
| 2052-12-26 | 1691-12-23 04:01:09 |      0 |
| 2168-08-05 | 2169-12-18 15:44:31 |      1 |
| 1737-02-06 | 1840-11-23 13:09:50 |      2 |
| 2245-10-01 | 1751-03-21 00:19:04 |      3 |
| 1889-10-27 | 1861-09-12 13:28:18 |      4 |
+------------+---------------------+--------+
5 rows in set (0.06 sec)
```

例3: 指定されたDATE値を5秒間隔の始まりに変換します。

```Plaintext
select date_slice(id_date, interval 5 second, FLOOR)
from test_all_type_select
order by id_int;
ERROR 1064 (HY000): can't use date_slice for date with time(hour/minute/second)
```

DATE値の秒部分が見つからないため、エラーが返されます。

例4: 指定されたDATE値を5日間隔の始まりに変換します。

```Plaintext
select date_slice(id_date, interval 5 day, FLOOR)
from test_all_type_select
order by id_int;
+--------------------------------------------+
| date_slice(id_date, INTERVAL 5 day, floor) |
+--------------------------------------------+
| 2052-12-24                                 |
| 2168-08-03                                 |
| 1737-02-04                                 |
| 2245-09-29                                 |
| 1889-10-25                                 |
+--------------------------------------------+
5 rows in set (0.14 sec)
```

例5: 指定されたDATE値を5日間隔の終わりに変換します。

```Plaintext
select date_slice(id_date, interval 5 day, CEIL)
from test_all_type_select
order by id_int;
+-------------------------------------------+
| date_slice(id_date, INTERVAL 5 day, ceil) |
+-------------------------------------------+
| 2052-12-29                                |
| 2168-08-08                                |
| 1737-02-09                                |
| 2245-10-04                                |
| 1889-10-30                                |
+-------------------------------------------+
5 rows in set (0.17 sec)
```
