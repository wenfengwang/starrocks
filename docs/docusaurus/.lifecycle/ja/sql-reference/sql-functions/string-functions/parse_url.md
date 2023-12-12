---
displayed_sidebar: "Japanese"
---

# parse_url

## 説明

URLをパースし、このURLからコンポーネントを抽出します。

## 構文

```Haskell
parse_url(expr1,expr2);
```

## パラメータ

`expr1`: URL。サポートされるデータ型はVARCHARです。

`expr2`: このURLから抽出するコンポーネントです。サポートされるデータ型はVARCHARです。有効な値は以下の通りです：

- PROTOCOL
- HOST
- PATH
- REF
- AUTHORITY
- FILE
- USERINFO
- QUERY。QUERY内のパラメータは返されません。特定のパラメータを返す場合は、この実装を行うために[trim](trim.md)を使用したparse_urlを使用してください。詳細は[Examples](#examples)を参照してください。

`expr2` は **大文字と小文字を区別** します。

## 戻り値

VARCHAR型の値を返します。URLが無効な場合はエラーが返されます。要求された情報を見つけることができない場合は、NULLが返されます。

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