---
displayed_sidebar: English
---

# timestamp

## 説明

日付式または日時式のDATETIME値を返します。

## 構文

```Haskell
DATETIME timestamp(DATETIME|DATE expr);
```

## パラメータ

`expr`: 変換したい時間表現です。DATETIME型またはDATE型でなければなりません。

## 戻り値

DATETIME値を返します。入力された時間が空であるか存在しない場合（例：`2021-02-29`）、NULLが返されます。

## 例

```Plain Text
select timestamp("2019-05-27");
+-------------------------+
| timestamp('2019-05-27') |
+-------------------------+
| 2019-05-27 00:00:00     |
+-------------------------+
1行がセットされました (0.00秒)
```
