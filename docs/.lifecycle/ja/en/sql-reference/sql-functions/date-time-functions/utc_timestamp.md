---
displayed_sidebar: English
---

# utc_timestamp

## 説明

現在のUTC日付と時刻を、関数の使用法に応じて 'YYYY-MM-DD HH:MM:SS' または 'YYYYMMDDHHMMSS' 形式の値として返します。例えば、文字列や数値のコンテキストで使用される場合です。

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

-- v3.1以降、結果はマイクロ秒単位で正確です。
select utc_timestamp();
+----------------------------+
| utc_timestamp()            |
+----------------------------+
| 2023-11-18 04:59:14.561000 |
+----------------------------+
```

`utc_timestamp() + N`は、現在時刻に`N`秒を加えることを意味します。

## キーワード

UTC_TIMESTAMP,UTC,TIMESTAMP
