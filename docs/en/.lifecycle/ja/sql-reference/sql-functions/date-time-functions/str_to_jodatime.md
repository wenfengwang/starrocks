---
displayed_sidebar: "Japanese"
---

# str_to_jodatime

## 説明

指定されたフォーマットに従って、Joda形式の文字列をDATETIME値に変換します。

フォーマットは[Joda DateTime](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)の形式であり、'yyyy-MM-dd HH:mm:ss'のようなものです。

## 構文

```Haskell
DATETIME str_to_jodatime(VARCHAR str, VARCHAR format)
```

## パラメータ

- `str`: 変換したい時間表現です。VARCHAR型である必要があります。
- `format`: Joda DateTimeのフォーマットです。

## 戻り値

- パースに成功した場合、`DATETIME`値を返します。
- パースに失敗した場合、`NULL`を返します。

## 例

例1: 入力をDATETIME値に変換します。

```Plain Text
MySQL > select str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss');
+--------------------------------------------------------------+
| str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss') |
+--------------------------------------------------------------+
| 2014-12-21 12:34:56                                          |
+--------------------------------------------------------------+
```


例2: 入力をテキスト形式の月を持つDATETIME値に変換します。

```Plain Text
MySQL > select str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss');
+------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss') |
+------------------------------------------------------------------+
| 2023-12-21 12:34:56                                              |
+------------------------------------------------------------------+
```


例3: 入力をミリ秒精度を持つDATETIME値に変換します。

```Plain Text
MySQL root@127.1:(none)> select str_to_jodatime('21/December/23 12:34:56.123', 'dd/MMMM/yy HH:mm:ss.SSS');
+--------------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56.123', 'dd/MMMM/yy HH:mm:ss.SSS') |
+--------------------------------------------------------------------------+
| 2023-12-21 12:34:56.123000                                               |
+--------------------------------------------------------------------------+
```


## キーワード

str_to_jodatime, DATETIME
