---
displayed_sidebar: "Japanese"
---

# utc_timestamp

## 説明

現在のUTCの日付と時刻を、'YYYY-MM-DD HH:MM:SS'または'YYYYMMDDHHMMSS'の形式で返します。関数の使用方法によって異なります。例えば、文字列や数値のコンテキストで使用する場合です。

## 構文

```Haskell
DATETIME UTC_TIMESTAMP()
```

## 例

```Plain Text
MySQL > select utc_timestamp(),utc_timestamp() + 1;
+---------------------+---------------------+
| utc_timestamp()     | utc_timestamp() + 1 |
+---------------------+---------------------+
| 2019-07-10 12:31:18 |      20190710123119 |
+---------------------+---------------------+

-- バージョン3.1以降、結果はマイクロ秒まで正確です。
select utc_timestamp();
+----------------------------+
| utc_timestamp()            |
+----------------------------+
| 2023-11-18 04:59:14.561000 |
+----------------------------+
```

`utc_timestamp() + N` は現在の時刻に `N` 秒を加えることを意味します。

## キーワード

UTC_TIMESTAMP,UTC,TIMESTAMP
