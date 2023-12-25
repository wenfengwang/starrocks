---
displayed_sidebar: English
---

# ブラックリスト管理

場合によっては、管理者はクラスターのクラッシュや予期しない高い同時クエリを引き起こすSQLを防ぐために、特定のSQLパターンを無効にする必要があります。

StarRocksでは、ユーザーはSQLブラックリストの追加、表示、削除ができます。

## 構文

`enable_sql_blacklist` を使用してSQLブラックリストを有効にします。デフォルトは False（オフ）です。

~~~sql
admin set frontend config ("enable_sql_blacklist" = "true")
~~~

ADMIN_PRIV権限を持つ管理者ユーザーは、以下のコマンドを実行してブラックリストを管理できます。

~~~sql
ADD SQLBLACKLIST #sql# 
DELETE SQLBLACKLIST #sql# 
SHOW SQLBLACKLISTS  
~~~

* `enable_sql_blacklist` が true の場合、すべてのSQLクエリはsqlblacklistによってフィルタリングされます。一致した場合、ユーザーにはSQLがブラックリストにあると通知されます。そうでなければ、SQLは通常通り実行されます。SQLがブラックリストにある場合のメッセージは以下のようになります：

`ERROR 1064 (HY000): Access denied; sql 'select count (*) from test_all_type_select_2556' is in blacklist`

## ブラックリストの追加

~~~sql
ADD SQLBLACKLIST #sql#
~~~

**#sql#** は特定のSQLタイプの正規表現です。SQL自体が `(`, `)`, `*`, `.` などの正規表現の意味と混同される一般的な文字を含むため、これらを区別するにはエスケープ文字の使用が必要です。ただし、`(` と `)` はSQLで頻繁に使用されるため、エスケープ文字は不要です。他の特殊文字にはエスケープ文字 `\` を接頭辞として使用します。例えば：

* `count(\*)` を禁止する：

~~~sql
ADD SQLBLACKLIST "select count(\\*) from .+"
~~~

* `count(distinct)` を禁止する：

~~~sql
ADD SQLBLACKLIST "select count(distinct .+) from .+"
~~~

* `x`, `y` による order by limit `1 <= x <=7`, `5 <=y <=7` を禁止する：

~~~sql
ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]"
~~~

* 複雑なSQLを禁止する：

~~~sql
ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable"
~~~

## ブラックリストの表示

~~~sql
SHOW SQLBLACKLIST
~~~

結果の形式は `Index | Forbidden SQL` です。

例：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \\* 9 \\- 8\) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
| 3     | select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]                                                                                                                                                                                                                            |
| 4     | select count\(distinct .+\) from .+                                                                                                                                                                                                                                                                    |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~

`Forbidden SQL` に示されるSQLは、すべてのSQLセマンティック文字に対してエスケープされています。

## ブラックリストの削除

~~~sql
DELETE SQLBLACKLIST #indexlist#
~~~

例えば、上記のブラックリストから sqlblacklist 3 と 4 を削除します。

~~~sql
delete sqlblacklist  3, 4;   -- #indexlist# はカンマ (,) で区切られたIDのリストです。
~~~

すると、残りのsqlblacklistは以下の通りです。

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \\* 9 \\- 8\) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~
