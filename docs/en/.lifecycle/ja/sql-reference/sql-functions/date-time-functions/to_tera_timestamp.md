---
displayed_sidebar: "Japanese"
---

# to_tera_timestamp

## 説明

VARCHARの値を入力形式からDATETIMEに変換します。

## 構文

```Haskell

DATETIME to_tera_timestamp(VARCHAR str, VARCHAR format)
```

## パラメーター
- `str`: 変換したい時刻表現です。VARCHAR型である必要があります。
- `format`: 以下のDateTime形式です。

```
[ \r \n \t - / , . ;] : 句読点は無視されます
dd	                  : 月の日 (1-31)
hh	                  : 時間 (1-12)
hh24                  : 時間 (0-23)
mi                    : 分 (0-59)
mm                    : 月 (01-12)
ss                    : 秒 (0-59)
yyyy                  : 4桁の年
yy                    : 2桁の年
am                    : 午前指示子
pm                    : 午後指示子
```

## 例

```Plain Text
mysql> select to_tera_timestamp("1988/04/08 2:3:4","yyyy/mm/dd hh24:mi:ss");
+-----------------------------------------------------------+
| to_tera_timestamp('1988/04/08 2:3:4', 'yyyy/mm/dd hh24:mi:ss') |
+-----------------------------------------------------------+
| 1988-04-08 02:03:04                                       |
+-----------------------------------------------------------+
```

## キーワード

to_tera_timestamp
