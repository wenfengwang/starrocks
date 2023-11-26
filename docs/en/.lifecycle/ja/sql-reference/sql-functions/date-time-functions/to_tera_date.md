---
displayed_sidebar: "Japanese"
---

# to_date

## 説明

VARCHARの値を入力形式から日付に変換します。

## 構文

```Haskell

VARCHARの値を入力形式から日付に変換します。

DATE to_tera_date(VARCHAR str, VARCHAR format)
```

## パラメータ

VARCHARの値を入力形式から日付に変換します。

- `str`: 変換したい時間表現です。VARCHAR型である必要があります。
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
VARCHARの値を入力形式から日付に変換します。

mysql> select to_date("1988/04/08","yyyy/mm/dd");
+-------------------------------------+
| to_date('1988/04/08', 'yyyy/mm/dd') |
+-------------------------------------+
| 1988-04-08                          |
+-------------------------------------+

```

## キーワード

TO_TERA_DATE
