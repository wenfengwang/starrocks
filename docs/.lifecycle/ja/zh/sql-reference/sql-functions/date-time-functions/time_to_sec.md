---
displayed_sidebar: Chinese
---

# time_to_sec

## 機能

`time` 時間値を秒数に変換します。変換式は以下の通りです:

`${時間}\times{3600} + {分}\times{60} + 秒$`

## 文法

```Haskell
BIGINT time_to_sec(TIME time)
```

## パラメータ説明

`time`：サポートされるデータ型は TIME です。

## 戻り値の説明

BIGINT 型の値を返します。入力値の形式が不正な場合は、NULL を返します。

## 例

```plain text
select time_to_sec('12:13:14');
+-----------------------------+
| time_to_sec('12:13:14')     |
+-----------------------------+
|                        43994|
+-----------------------------+
```
