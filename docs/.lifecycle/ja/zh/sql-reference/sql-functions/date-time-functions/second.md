---
displayed_sidebar: Chinese
---

# second

## 機能

日付から秒を取得し、返り値の範囲は 0~59 です。

引数は DATE または DATETIME 型です。

### 構文

```Haskell
INT SECOND(DATETIME date)
```

## 例

```Plain Text
select second('2018-12-31 23:59:59');
+-----------------------------+
|second('2018-12-31 23:59:59')|
+-----------------------------+
|                          59 |
+-----------------------------+
```
