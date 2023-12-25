---
displayed_sidebar: Chinese
---

# curdate,current_date

## 機能

現在の日付を取得し、DATE 型で返します。

## 文法

```Haskell
DATE CURDATE()
```

## 例

```Plain Text
SELECT CURDATE();
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

SELECT CURDATE() + 0;
+-----------------+
| (curdate()) + 0 |
+-----------------+
|        20221220 |
+-----------------+
```
