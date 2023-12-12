---
displayed_sidebar: "Japanese"
---

# time_to_sec

## 説明

時間の値を秒数に変換します。変換に使用される式は次のとおりです。

時間 x 3600 + 分 x 60 + 秒

## 構文

```Haskell
BIGINT time_to_sec(TIME time)
```

## パラメータ

`time`: TIME型である必要があります。

## 戻り値

BIGINT型の値を返します。入力が無効な場合、NULLが返されます。

## 例

```plain text
select time_to_sec('12:13:14');
+-----------------------------+
| time_to_sec('12:13:14')     |
+-----------------------------+
|                        43994|
+-----------------------------+
```