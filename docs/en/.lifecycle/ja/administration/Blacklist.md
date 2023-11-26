---
displayed_sidebar: "Japanese"
---

# ブラックリスト管理

一部の場合、管理者はクラスタのクラッシュや予期しない高並行クエリの発生を避けるために、特定のSQLパターンを無効にする必要があります。

StarRocksでは、ユーザーがSQLのブラックリストを追加、表示、削除することができます。

## 構文

`enable_sql_blacklist`を使用してSQLブラックリストを有効にします。デフォルトはFalse（オフ）です。

~~~sql
admin set frontend config ("enable_sql_blacklist" = "true")
~~~

ADMIN_PRIV権限を持つ管理者ユーザーは、次のコマンドを実行することでブラックリストを管理できます。

~~~sql
ADD SQLBLACKLIST #sql# 
DELETE SQLBLACKLIST #sql# 
SHOW SQLBLACKLISTS  
~~~

* `enable_sql_blacklist`がtrueの場合、すべてのSQLクエリはsqlblacklistによってフィルタリングされる必要があります。一致する場合、ユーザーにはSQLがブラックリストにあることが通知されます。それ以外の場合、SQLは通常通り実行されます。SQLがブラックリストにある場合、次のようなメッセージが表示される場合があります：

`ERROR 1064 (HY000): Access denied; sql 'select count (*) from test_all_type_select_2556' is in blacklist`

## ブラックリストの追加

~~~sql
ADD SQLBLACKLIST #sql#
~~~

**#sql#**は特定のタイプのSQLの正規表現です。SQL自体には`(`、`)`、`*`、`.`といった共通の文字が含まれているため、正規表現の意味と混同される可能性があるため、エスケープ文字を使用してそれらを区別する必要があります。`(`と`)`はSQLで頻繁に使用されるため、エスケープ文字を使用する必要はありません。その他の特殊文字は、エスケープ文字`\`を接頭辞として使用する必要があります。例：

* `count(\*)`を禁止する場合：

~~~sql
ADD SQLBLACKLIST "select count(\\*) from .+"
~~~

* `count(distinct)`を禁止する場合：

~~~sql
ADD SQLBLACKLIST "select count(distinct .+) from .+"
~~~

* `order by limit x`, `y`, `1 <= x <=7`, `5 <=y <=7`を禁止する場合：

~~~sql
ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]"
~~~

* 複雑なSQLを禁止する場合：

~~~sql
ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable"
~~~

## ブラックリストの表示

~~~sql
SHOW SQLBLACKLIST
~~~

結果の形式：`インデックス | 禁止されたSQL`

例：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| インデックス | 禁止されたSQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
| 3     | select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]                                                                                                                                                                                                                            |
| 4     | select count\(distinct .+\) from .+                                                                                                                                                                                                                                                                    |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~

`禁止されたSQL`に表示されるSQLは、すべてのSQLセマンティック文字に対してエスケープされています。

## ブラックリストの削除

~~~sql
DELETE SQLBLACKLIST #indexlist#
~~~

例えば、上記のブラックリストからsqlblacklist 3と4を削除する場合：

~~~sql
delete sqlblacklist  3, 4;   -- #indexlist#はカンマ（,）で区切られたIDのリストです。
~~~

その後、残りのsqlblacklistは次のようになります：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| インデックス | 禁止されたSQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~
