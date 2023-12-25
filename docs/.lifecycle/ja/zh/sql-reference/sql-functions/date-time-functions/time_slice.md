---
displayed_sidebar: Chinese
---

# time_slice

## 機能

指定された時間粒度周期に基づき、与えられた時間をその時間粒度周期の開始または終了時刻に変換します。

この関数はバージョン2.3からサポートされています。バージョン2.5からは終了時刻への変換がサポートされています。

## 文法

```Haskell
DATETIME time_slice(DATETIME dt, INTERVAL N type[, boundary])
```

## 引数説明

- `dt`：変換する時間。サポートされるデータ型はDATETIMEです。

- `INTERVAL N type`：時間粒度周期。例えば `interval 5 second` は5秒の時間粒度を意味します。
  - `N` はINT型で、時間粒度周期の長さです。
  - `type` は時間粒度周期の単位で、YEAR、QUARTER、MONTH、WEEK、DAY、HOUR、MINUTE、SECONDが可能です。

- `boundary`：オプションで、時間周期の開始時刻（`FLOOR`）か終了時刻（`CEIL`）を指定します。値の範囲はFLOOR、CEILです。指定しない場合のデフォルトは `FLOOR` です。このパラメータはバージョン2.5からサポートされています。

## 戻り値説明

戻り値のデータ型はDATETIMEです。

## 注意事項

時間粒度周期は西暦 `0001-01-01 00:00:00` から始まります。

## 例

`test_all_type_select` というテーブルがあり、`id_int` でソートされたデータが以下の通りです：

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

例1：与えられたDATETIME `id_datetime` を5秒の時間粒度周期の開始時刻に変換します（`boundary` パラメータは指定せず、デフォルトで `FLOOR`）。

```Plaintext
select time_slice(id_datetime, interval 5 second)
from test_all_type_select
order by id_int;

+---------------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 second, FLOOR) |
+---------------------------------------------------+
| 1691-12-23 04:01:05                               |
| 2169-12-18 15:44:30                               |
| 1840-11-23 13:09:50                               |
| 1751-03-21 00:19:00                               |
| 1861-09-12 13:28:15                               |
+---------------------------------------------------+
5 rows in set (0.16 sec)
```

例2：与えられたDATETIME `id_datetime` を5日の時間粒度周期の開始時刻に変換します（`boundary` を `FLOOR` に設定）。

```Plaintext
select time_slice(id_datetime, interval 5 day, FLOOR)
from test_all_type_select
order by id_int;

+------------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 day, FLOOR) |
+------------------------------------------------+
| 1691-12-22 00:00:00                            |
| 2169-12-16 00:00:00                            |
| 1840-11-21 00:00:00                            |
| 1751-03-18 00:00:00                            |
| 1861-09-12 00:00:00                            |
+------------------------------------------------+
5 rows in set (0.15 sec)
```

例3：与えられたDATETIME `id_datetime` を5日の時間粒度周期の終了時刻に変換します。

```Plaintext
select time_slice(id_datetime, interval 5 day, CEIL)
from test_all_type_select
order by id_int;

+-----------------------------------------------+
| time_slice(id_datetime, INTERVAL 5 day, CEIL) |
+-----------------------------------------------+
| 1691-12-27 00:00:00                           |
| 2169-12-21 00:00:00                           |
| 1840-11-26 00:00:00                           |
| 1751-03-23 00:00:00                           |
| 1861-09-17 00:00:00                           |
+-----------------------------------------------+
5 rows in set (0.12 sec)
```
