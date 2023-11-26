---
displayed_sidebar: "Japanese"
---

# parse_url

## 説明

URLを解析し、このURLからコンポーネントを抽出します。

## 構文

```Haskell
parse_url(expr1,expr2);
```

## パラメータ

`expr1`: URL。サポートされるデータ型はVARCHARです。

`expr2`: このURLから抽出するコンポーネント。サポートされるデータ型はVARCHARです。有効な値:

- PROTOCOL
- HOST
- PATH
- REF
- AUTHORITY
- FILE
- USERINFO
- QUERY。QUERY内のパラメータは返されません。特定のパラメータを返す場合は、[trim](trim.md)を使用してこの実装を行います。詳細については、[例](#examples)を参照してください。

`expr2`は**大文字と小文字を区別**します。

## 戻り値

VARCHAR型の値を返します。URLが無効な場合はエラーが返されます。要求された情報が見つからない場合はNULLが返されます。

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
