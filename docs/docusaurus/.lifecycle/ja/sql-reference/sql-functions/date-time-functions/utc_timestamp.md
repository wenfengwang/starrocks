---
displayed_sidebar: "Japanese"
---

# utc_timestamp

## 説明

現在のUTC日時を 'YYYY-MM-DD HH:MM:SS' または 'YYYYMMDDHHMMSS' 形式の値として返します。関数の使用方法に応じて、文字列または数値のコンテキストで返されます。

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

-- 結果はv3.1以降マイクロ秒単位で正確です。
select utc_timestamp();
+----------------------------+
| utc_timestamp()            |
+----------------------------+
| 2023-11-18 04:59:14.561000 |
+----------------------------+
```

`utc_timestamp() + N` は、現在時刻に `N` 秒を追加することを意味します。

## キーワード

UTC_TIMESTAMP, UTC, TIMESTAMP