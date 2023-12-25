---
displayed_sidebar: Chinese
---

# date_slice

## 機能

指定された時間粒度周期に基づき、与えられた時間をその時間粒度周期の開始または終了時刻に変換します。

この関数はバージョン2.5からサポートされています。

## 文法

```Haskell
DATE date_slice(DATE dt, INTERVAL N type[, boundary])
```

## パラメータ説明

- `dt`：変換する時間。サポートされるデータ型は DATE です。
- `INTERVAL N type`：時間粒度周期。例えば `interval 5 day` は時間粒度が5日間であることを意味します。
  - `N` は INT 型の時間周期の長さです。
  - `type` は時間粒度周期の単位で、YEAR、QUARTER、MONTH、WEEK、DAY が使用できます。DATE 型の入力値に対して、`type` が時分秒である場合はエラーが返されます。
- `boundary`：オプションで、時間周期の開始時刻（FLOOR）または終了時刻（CEIL）を指定します。選択肢は FLOOR、CEIL です。指定しない場合のデフォルトは FLOOR です。

## 戻り値の説明

戻り値のデータ型は DATE です。

## 注意事項

時間粒度周期は西暦`0001-01-01`から始まります。

## 例

例1：与えられた時間を5年の時間粒度周期の開始時刻に変換します（`boundary` パラメータは指定せず、デフォルトで `FLOOR`）。

```Plaintext
select date_slice('2022-04-26', interval 5 year);
+--------------------------------------------------+
| date_slice('2022-04-26', INTERVAL 5 year, floor) |
+--------------------------------------------------+
| 2021-01-01                                       |
+--------------------------------------------------+
```

例2：与えられた時間を5日間の時間粒度周期の終了時刻に変換します。

```Plaintext
select date_slice('0001-01-07', interval 5 day, CEIL);
+------------------------------------------------+
| date_slice('0001-01-07', INTERVAL 5 day, ceil) |
+------------------------------------------------+
| 0001-01-11                                     |
+------------------------------------------------+
```

以下の例では `test_all_type_select` テーブルを使用し、`id_int` でソートしています：

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

例3：`id_date` の時間を5秒間の時間粒度周期の開始時刻に変換します。

```Plaintext
select date_slice(id_date, interval 5 second, FLOOR)
from test_all_type_select
order by id_int;
ERROR 1064 (HY000): can't use date_slice for date with time(hour/minute/second)
```

DATE 型の時間値に対して時間部分（時/分/秒）を使用することはできないというエラーメッセージが表示されます。

例4：`id_date` の時間を5日間の時間粒度周期の開始時刻に変換します。

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

例5：`id_date` の時間を5日間の時間粒度周期の終了時刻に変換します。

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
