```yaml
---
displayed_sidebar: "Japanese"
---

# convert_tz

## 説明

あるタイムゾーンから別のタイムゾーンへのDATEまたはDATETIMEの値を変換します。

この関数は異なるタイムゾーンで異なる結果を返す可能性があります。詳細については[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。

## 構文

```Haskell
DATETIME CONVERT_TZ(DATETIME|DATE dt, VARCHAR from_tz, VARCHAR to_tz)
```

## パラメータ

- `dt`: 変換するDATEまたはDATETIMEの値です。

- `from_tz`: 元のタイムゾーンです。VARCHARがサポートされています。タイムゾーンは2つの形式で表されることができます。1つはタイムゾーンデータベース（例: Asia/Shanghai）、もう1つはUTCオフセット（例: +08:00）です。

- `to_tz`: 変換先のタイムゾーンです。VARCHARがサポートされています。その形式は`from_tz`と同じです。

## 戻り値

DATETIMEデータ型の値を返します。入力がDATE値の場合、DATETIME値に変換されます。この関数は、入力パラメータのいずれかが無効またはNULLの場合はNULLを返します。

## 使用上の注意

タイムゾーンデータベースについては、[tzデータベースタイムゾーンのリスト](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)（Wikipedia）を参照してください。

## 例

例1: ShanghaiのdatetimeをLos_Angelesに変換する。

```plaintext
select convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles');
+---------------------------------------------------------------------------+
| convert_tz('2019-08-01 13:21:03', 'Asia/Shanghai', 'America/Los_Angeles') |
+---------------------------------------------------------------------------+
| 2019-07-31 22:21:03                                                       |
+---------------------------------------------------------------------------+
1 row in set (0.00 sec)                                                       |
```

例2: ShanghaiのdateをLos_Angelesに変換する。

```plaintext
select convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles');
+------------------------------------------------------------------+
| convert_tz('2019-08-01', 'Asia/Shanghai', 'America/Los_Angeles') |
+------------------------------------------------------------------+
| 2019-07-31 09:00:00                                              |
+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

例3: UTC+08:00のdatetimeをLos_Angelesに変換する。

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

CONVERT_TZ, タイムゾーン, タイムゾーン