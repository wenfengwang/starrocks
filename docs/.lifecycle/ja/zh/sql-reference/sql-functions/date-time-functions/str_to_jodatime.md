---
displayed_sidebar: Chinese
---

# str_to_jodatime

## 機能

Joda形式の文字列を指定されたJoda DateTime形式（例：`yyyy-MM-dd HH:mm:ss`）のDATETIME値に変換します。

## 文法

```Haskell
DATETIME str_to_jodatime(VARCHAR str, VARCHAR format)
```

## パラメータ説明

- `str`：変換対象の時間表現文字列で、VARCHARデータ型でなければなりません。
- `format`：変換後に生成されるDATETIME値のJoda DateTime形式です。[Joda DateTime](https://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)を参照してください。

## 戻り値の説明

- 入力された文字列が正しく解析された場合、DATETIME型の値を返します。
- 入力された文字列の解析に失敗した場合、`NULL`を返します。

## 例

例1：文字列 `2014-12-21 12:34:56` を `yyyy-MM-dd HH:mm:ss` 形式のDATETIME値に変換します。

```SQL
MySQL > select str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss');
+--------------------------------------------------------------+
| str_to_jodatime('2014-12-21 12:34:56', 'yyyy-MM-dd HH:mm:ss') |
+--------------------------------------------------------------+
| 2014-12-21 12:34:56                                          |
+--------------------------------------------------------------+
```

例2：文字列 `21/December/23 12:34:56`（月を単語で表した時間表現）を `dd/MMMM/yy HH:mm:ss` 形式のDATETIME値に変換します。

```SQL
MySQL > select str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss');
+------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56', 'dd/MMMM/yy HH:mm:ss') |
+------------------------------------------------------------------+
| 2023-12-21 12:34:56                                              |
+------------------------------------------------------------------+
```

例3：文字列 `21/December/23 12:34:56.123`（ミリ秒単位で精度を持つ時間表現）を `dd/MMMM/yy HH:mm:ss.SSS` 形式のDATETIME値に変換します。

```SQL
MySQL > select str_to_jodatime('21/December/23 12:34:56.123', 'dd/MMMM/yy HH:mm:ss.SSS');
+--------------------------------------------------------------------------+
| str_to_jodatime('21/December/23 12:34:56.123', 'dd/MMMM/yy HH:mm:ss.SSS') |
+--------------------------------------------------------------------------+
| 2023-12-21 12:34:56.123000                                               |
+--------------------------------------------------------------------------+
```

## キーワード

STR_TO_JODATIME, DATETIME
