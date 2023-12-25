---
displayed_sidebar: English
---

# parse_url

## 説明

URL を解析し、この URL からコンポーネントを抽出します。

## 構文

```Haskell
parse_url(expr1,expr2);
```

## パラメーター

`expr1`: URL です。サポートされているデータ型は VARCHAR です。

`expr2`: この URL から抽出するコンポーネント。サポートされているデータ型は VARCHAR です。有効値:

- PROTOCOL
- HOST
- PATH
- REF
- AUTHORITY
- FILE
- USERINFO
- QUERY。QUERY のパラメーターは返せません。特定のパラメーターを返したい場合は、[trim](trim.md) を使用して parse_url でこの実装を行います。詳細は[例](#examples)を参照してください。

`expr2` は **大文字と小文字を区別します**。

## 戻り値

VARCHAR 型の値を返します。URL が無効な場合はエラーが返されます。要求された情報が見つからない場合は NULL が返されます。

## 例

```Plain Text
select parse_url('http://facebook.com/path/p1.php?query=1', 'HOST');
+--------------------------------------------------------------+
| parse_url('http://facebook.com/path/p1.php?query=1', 'HOST') |
+--------------------------------------------------------------+
| facebook.com                                                 |
+--------------------------------------------------------------+

select parse_url('http://facebook.com/path/p1.php?query=1', 'AUTHORITY');
+-------------------------------------------------------------------+
| parse_url('http://facebook.com/path/p1.php?query=1', 'AUTHORITY') |
+-------------------------------------------------------------------+
| facebook.com                                                      |
+-------------------------------------------------------------------+

select parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY');
+---------------------------------------------------------------+
| parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY') |
+---------------------------------------------------------------+
| query=1                                                       |
+---------------------------------------------------------------+

select trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'),'query='); 
+-------------------------------------------------------------------------------+
| trim(parse_url('http://facebook.com/path/p1.php?query=1', 'QUERY'), 'query=') |
+-------------------------------------------------------------------------------+
| 1                                                                             |
+-------------------------------------------------------------------------------+
```
