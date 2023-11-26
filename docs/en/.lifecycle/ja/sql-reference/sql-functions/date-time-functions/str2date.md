---
displayed_sidebar: "Japanese"
---

# str2date

## 説明

指定された形式に従って、文字列をDATE値に変換します。変換に失敗した場合、NULLが返されます。

形式は[date_format](./date_format.md)で説明されているものと一致している必要があります。

この関数は[str_to_date](../date-time-functions/str_to_date.md)と同等ですが、異なる戻り値の型を持っています。

## 構文

```Haskell
DATE str2date(VARCHAR str, VARCHAR format);
```

## パラメータ

`str`: 変換したい時間表現です。VARCHAR型である必要があります。

`format`: 値を返すために使用される形式です。サポートされている形式については[date_format](./date_format.md)を参照してください。

## 戻り値

DATE型の値が返されます。

`str`または`format`がNULLの場合は、NULLが返されます。

## 例

```Plain
select str2date('2010-11-30 23:59:59', '%Y-%m-%d %H:%i:%s');
+------------------------------------------------------+
| str2date('2010-11-30 23:59:59', '%Y-%m-%d %H:%i:%s') |
+------------------------------------------------------+
| 2010-11-30                                           |
+------------------------------------------------------+
1 row in set (0.01 sec)
```
