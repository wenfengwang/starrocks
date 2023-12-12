---
displayed_sidebar: "Japanese"
---

# current_timestamp（現在のタイムスタンプ）

## 説明

現在の日付を取得し、DATETIME型の値で返します。

v3.1以降、結果はマイクロ秒まで正確です。

## 構文

```Haskell
DATETIME CURRENT_TIMESTAMP()
```

## 例

```Plain Text
MySQL > select current_timestamp();
+---------------------+
| current_timestamp() |
+---------------------+
| 2019-05-27 15:59:33 |
+---------------------+

-- v3.1以降、結果はマイクロ秒まで正確です。
MySQL > select current_timestamp();
+----------------------------+
| current_timestamp()        |
+----------------------------+
| 2023-11-18 12:58:05.375000 |
+----------------------------+
```

## キーワード

CURRENT_TIMESTAMP, CURRENT, TIMESTAMP