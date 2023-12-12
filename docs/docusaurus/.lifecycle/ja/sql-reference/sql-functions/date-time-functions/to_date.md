---
displayed_sidebar: "Japanese"
---

# to_date

## 説明

DATETIME値を日付に変換します。

## 構文

```Haskell
DATE TO_DATE(DATETIME datetime)
```

## 例

```Plain Text
MySQL > select to_date("2020-02-02 00:00:00");
+--------------------------------+
| to_date('2020-02-02 00:00:00') |
+--------------------------------+
| 2020-02-02                     |
+--------------------------------+
```

## キーワード

TO_DATE