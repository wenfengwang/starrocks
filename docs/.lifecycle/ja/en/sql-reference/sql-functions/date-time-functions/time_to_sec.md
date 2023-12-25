---
displayed_sidebar: English
---

# time_to_sec

## 説明

時間値を秒数に変換します。変換に使用される式は次のとおりです:

時 x 3600 + 分 x 60 + 秒

## 構文

```Haskell
BIGINT time_to_sec(TIME time)
```

## パラメーター

`time`: TIME 型でなければなりません。

## 戻り値

BIGINT 型の値を返します。入力が無効な場合は、NULL が返されます。

## 例

```plain text
select time_to_sec('12:13:14');
+-----------------------------+
| time_to_sec('12:13:14')     |
+-----------------------------+
|                        43994|
+-----------------------------+
```
