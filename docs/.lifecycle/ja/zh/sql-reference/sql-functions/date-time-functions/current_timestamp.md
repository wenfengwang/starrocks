---
displayed_sidebar: Chinese
---

# current_timestamp

## 機能

現在の時刻を取得し、DATETIME 型で返します。バージョン 3.1 から、この関数の戻り値はマイクロ秒単位になります。

## 文法

```Haskell
DATETIME CURRENT_TIMESTAMP()
```

## 例

```Plain Text
select current_timestamp();
+---------------------+
| current_timestamp() |
+---------------------+
| 2019-05-27 15:59:33 |
+---------------------+

-- バージョン 3.1 以降の戻り値はマイクロ秒単位です。
select current_timestamp();
+----------------------------+
| current_timestamp()        |
+----------------------------+
| 2023-11-18 12:58:05.375000 |
+----------------------------+
```
