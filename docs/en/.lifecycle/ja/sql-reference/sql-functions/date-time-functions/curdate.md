---
displayed_sidebar: "Japanese"
---

# curdate,current_date

## 説明

現在の日付を取得し、DATE型の値を返します。

## 構文

```Haskell
DATE CURDATE()
```

## 例

```Plain Text
MySQL > SELECT CURDATE();
+------------+
| curdate()  |
+------------+
| 2022-12-20 |
+------------+

SELECT CURRENT_DATE();
+----------------+
| current_date() |
+----------------+
| 2022-12-20     |
+----------------+


MySQL > SELECT CURDATE() + 0;
+---------------+
| CURDATE() + 0 |
+---------------+
|      20221220 |
+---------------+
```
