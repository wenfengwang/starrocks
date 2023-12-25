---
displayed_sidebar: English
---

# curtime、current_time

## 説明

現在の時刻を取得し、TIME型の値を返します。

この関数は、タイムゾーンによって異なる結果を返すことがあります。詳細については、[タイムゾーンの設定](../../../administration/timezone.md)を参照してください。

## 構文

```Haskell
TIME CURTIME()
```

## 例

```Plain Text
MySQL > select current_time();
+----------------+
| current_time() |
+----------------+
| 15:25:47       |
+----------------+
```

## キーワード

CURTIME, CURRENT_TIME
