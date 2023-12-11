---
displayed_sidebar: "Japanese"
---

# current_timestamp

## Description

現在の日付を取得し、DATETIME型の値を返します。

v3.1以降、結果はマイクロ秒まで正確です。

## Syntax

```Haskell
DATETIME CURRENT_TIMESTAMP()
```

## Examples

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

## keyword

CURRENT_TIMESTAMP,CURRENT,TIMESTAMP