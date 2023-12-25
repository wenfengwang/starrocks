---
displayed_sidebar: English
---

# convert_tz

## 説明

DATE 型または DATETIME 型の値をあるタイムゾーンから別のタイムゾーンに変換します。

この関数は、タイムゾーンによって異なる結果を返すことがあります。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。

## 構文

```Haskell
DATETIME CONVERT_TZ(DATETIME|DATE dt, VARCHAR from_tz, VARCHAR to_tz)
```

## パラメータ

- `dt`: 変換する DATE 型または DATETIME 型の値。

- `from_tz`: ソースタイムゾーン。VARCHAR がサポートされています。タイムゾーンは、タイムゾーンデータベース（例：Asia/Shanghai）または UTC オフセット（例：+08:00）のいずれかの形式で指定できます。

- `to_tz`: 目的タイムゾーン。VARCHAR がサポートされています。その形式は `from_tz` と同じです。

## 戻り値

DATETIME 型の値を返します。入力が DATE 型の場合は、DATETIME 型に変換されます。入力パラメータのいずれかが無効または NULL の場合、この関数は NULL を返します。

## 使用上の注意

タイムゾーンデータベースについては、[Wikipedia の tz データベースタイムゾーンの一覧](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)を参照してください。

## 例

例 1: 上海の DATETIME を Los_Angeles に変換します。

```plaintext
select convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles');
+---------------------------------------------------------------------------+
| convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles') |
+---------------------------------------------------------------------------+
| 2019-07-31 22:21:03                                                       |
+---------------------------------------------------------------------------+
1 row in set (0.00 sec)                                                       |
```

例 2: 上海の DATE を Los_Angeles に変換します。

```plaintext
select convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles');
+------------------------------------------------------------------+
| convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles') |
+------------------------------------------------------------------+
| 2019-07-31 09:00:00                                              |
+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

例 3: UTC+08:00 の DATETIME を Los_Angeles に変換します。

```plaintext
select convert_tz('2019-08-01 13:21:03', '+08:00', 'America/Los_Angeles');
+--------------------------------------------------------------------+
| convert_tz('2019-08-01 13:21:03', '+08:00', 'America/Los_Angeles') |
+--------------------------------------------------------------------+
| 2019-07-31 22:21:03                                                |
+--------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## キーワード

CONVERT_TZ, timezone, time zone
