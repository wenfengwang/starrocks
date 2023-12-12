---
displayed_sidebar: "Japanese"
---

# タイムスタンプ

## 説明

日付または日時の式のDATETIME値を返します。

## 構文

```Haskell
DATETIME timestamp(DATETIME|DATE expr);
```

## パラメータ

`expr`: 変換したい時刻の式。DATETIMEまたはDATE型である必要があります。

## 戻り値

DATETIME値を返します。入力された時間が空白であるか存在しない場合、例えば `2021-02-29`、NULLが返されます。

## 例

```Plain Text
select timestamp("2019-05-27");
+-------------------------+
| timestamp('2019-05-27') |
+-------------------------+
| 2019-05-27 00:00:00     |
+-------------------------+
1 行が返されました (0.00 秒)
```