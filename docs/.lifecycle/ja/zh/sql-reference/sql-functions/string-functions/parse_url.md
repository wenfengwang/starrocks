---
displayed_sidebar: Chinese
---

# parse_url

## 機能

ターゲットURLから情報の一部を抽出します。

## 文法

```Haskell
parse_url(expr1,expr2);
```

## パラメータ説明

`expr1`: ターゲットURL。サポートされるデータ型はVARCHARです。

`expr2`: 抽出する情報。サポートされるデータ型はVARCHARです。以下の値を取ることができますが、**大文字と小文字は区別されます**：

- PROTOCOL
- HOST
- PATH
- REF
- AUTHORITY
- FILE
- USERINFO
- QUERY（QUERY内の特定のパラメータを返すことはサポートされていません。特定のパラメータを返したい場合は、[trim](trim.md)関数を併用してください。例を参照。）

## 戻り値の説明

戻り値のデータ型はVARCHARです。入力されたURL文字列が無効な場合はエラーを返します。要求された情報が見つからない場合はNULLを返します。

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
