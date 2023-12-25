---
displayed_sidebar: Chinese
---

# timestampadd

## 機能

整数表現の間隔を日付または日時表現 `datetime_expr` に追加します。

`interval` の単位は `unit` パラメータによって指定され、以下のいずれかでなければなりません:

SECOND、MINUTE、HOUR、DAY、WEEK、MONTH、YEAR。

## 文法

```Haskell
DATETIME TIMESTAMPADD(unit, interval, DATETIME datetime_expr)
```

## 例

```plain text

SELECT TIMESTAMPADD(MINUTE,1,'2019-01-02');
+------------------------------------------------+
| timestampadd(MINUTE, 1, '2019-01-02 00:00:00') |
+------------------------------------------------+
| 2019-01-02 00:01:00                            |
+------------------------------------------------+

SELECT TIMESTAMPADD(WEEK,1,'2019-01-02');
+----------------------------------------------+
| timestampadd(WEEK, 1, '2019-01-02 00:00:00') |
+----------------------------------------------+
| 2019-01-09 00:00:00                          |
+----------------------------------------------+
```
