---
displayed_sidebar: "英語"
---

# ブラックリスト管理

場合によっては、管理者がクラスタのクラッシュや予期せぬ高並行クエリのトリガーを回避するために、特定のSQLパターンを無効にする必要があることがあります。

StarRocksを使用すると、ユーザーはSQLブラックリストの追加、閲覧、削除が可能です。

## 文法

`enable_sql_blacklist`を通じてSQLブラックリスティングを有効にします。デフォルトはFalse（オフ）です。

~~~sql
admin set frontend config ("enable_sql_blacklist" = "true")
~~~

ADMIN_PRIV権限を持つ管理ユーザーは、以下のコマンドを実行することでブラックリストを管理できます：

~~~sql
ADD SQLBLACKLIST #sql# 
DELETE SQLBLACKLIST #sql# 
SHOW SQLBLACKLISTS  
~~~

* `enable_sql_blacklist`がtrueの場合、すべてのSQLクエリはsqlblacklistによってフィルタリングされる必要があります。一致する場合は、ユーザーはそのSQLがブラックリストにあることを通知されます。それ以外の場合は、SQLは通常通り実行されます。SQLがブラックリストに入っている場合のメッセージは以下のようになるかもしれません。

`ERROR 1064 (HY000): Access denied; sql 'select count (*) from test_all_type_select_2556' is in blacklist`

## ブラックリストの追加

~~~sql
ADD SQLBLACKLIST #sql#
~~~

**#sql#**は特定のSQLの正規表現です。SQLには`(`、`)`、`*`、`.`などの正規表現の意味合いと混同されがちな一般的な文字が含まれているため、これらをエスケープ文字を使用して区別する必要があります。ただし、`(`及び`)`はSQLで非常に頻繁に使用されるため、エスケープ文字を使用する必要はありません。その他の特殊文字はエスケープ文字`\`を接頭語として使用する必要があります。例えば：

* `count(\*)`を禁止する：

~~~sql
ADD SQLBLACKLIST "select count(\\*) from .+"
~~~

* `count(distinct)`を禁止する：

~~~sql
ADD SQLBLACKLIST "select count(distinct .+) from .+"
~~~

* order by limit `x`、`y`、`1 <= x <=7`、`5 <=y <=7`を禁止する：

~~~sql
ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]"
~~~

* 複雑なSQLを禁止する：

~~~sql
ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable"
~~~

## ブラックリストの閲覧

~~~sql
SHOW SQLBLACKLIST
~~~

結果フォーマット：`Index | Forbidden SQL`

例えば：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
| 3     | select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]                                                                                                                                                                                                                            |
| 4     | select count\(distinct .+\) from .+                                                                                                                                                                                                                                                                    |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~

`Forbidden SQL`にはSQLのセマンティック文字がすべてエスケープされて表示されます。

## ブラックリストの削除

~~~sql
DELETE SQLBLACKLIST #indexlist#
~~~

例として、上記のブラックリストのsqlblacklist 3および4を削除するとします：

~~~sql
delete sqlblacklist  3, 4;   -- #indexlist#はカンマ（,）で区切られたIDのリストです。
~~~

すると、残ったsqlblacklistは以下の通りです：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~