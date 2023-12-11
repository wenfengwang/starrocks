---
displayed_sidebar: "Japanese"
---

# ブラックリスト管理

場合によっては、管理者はクラスタのクラッシュや予期しない高並行クエリの発生を回避するために、特定の SQL パターンを無効にする必要があります。

StarRocks では、ユーザーが SQL ブラックリストを追加、表示、および削除できます。

## 構文

`enable_sql_blacklist` を介して SQL ブラックリストを有効にします。デフォルトは False です（オフ）。

~~~sql
admin set frontend config ("enable_sql_blacklist" = "true")
~~~

ADMIN_PRIV 権限を持つ admin ユーザーは、次のコマンドを実行することでブラックリストを管理できます。

~~~sql
ADD SQLBLACKLIST #sql# 
DELETE SQLBLACKLIST #sql# 
SHOW SQLBLACKLISTS  
~~~

* `enable_sql_blacklist` が true の場合、すべての SQL クエリは sqlblacklist によってフィルタリングする必要があります。一致する場合、ユーザーには SQL がブラックリストにあることが通知されます。それ以外の場合は、SQL は通常どおりに実行されます。SQL がブラックリストにある場合、メッセージは次のようになります。

`ERROR 1064 (HY000): アクセスが拒否されました; sql 'select count (*) from test_all_type_select_2556' はブラックリストにあります`

## ブラックリスト追加

~~~sql
ADD SQLBLACKLIST #sql#
~~~

**#sql#** は特定の種類の SQL に対する正規表現です。SQL 自体には正規表現の意味を持つ `(`、`)`、`*`、`.` などの一般的な文字が含まれているため、エスケープ文字を使用してそれらを区別する必要があります。たとえば、`(` と `)` は SQL で頻繁に使用されるため、エスケープ文字を使用する必要はありません。他の特殊文字は、エスケープ文字 `\` を接頭辞として使用する必要があります。例えば：

* `count(\*)` を禁止する場合：

~~~sql
ADD SQLBLACKLIST "select count(\\*) from .+"
~~~

* `count(distinct)` を禁止する場合：

~~~sql
ADD SQLBLACKLIST "select count(distinct .+) from .+"
~~~

* order by limit `x`, `y`, `1 <= x <=7`, `5 <=y とを制限する場合：

~~~sql
ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]"
~~~

* 複雑な SQL を禁止する場合：

~~~sql
ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable"
~~~

## ブラックリスト表示

~~~sql
SHOW SQLBLACKLIST
~~~

結果の形式: `インデックス | 禁止された SQL`

例：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| インデックス | 禁止された SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
| 3     | select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]                                                                                                                                                                                                                            |
| 4     | select count\(distinct .+\) from .+                                                                                                                                                                                                                                                                    |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
~~~

`禁止された SQL` に表示される SQL は、すべての SQL の意味をエスケープされています。

## ブラックリスト削除

~~~sql
DELETE SQLBLACKLIST #indexlist#
~~~

例えば、上記のブラックリストから sqlblacklist 3 および 4 を削除する場合：

~~~sql
delete sqlblacklist  3, 4;   -- #indexlist# はコンマ（,）で区切られた ID のリストです。
~~~

その後、残りの sqlblacklist は次のようになります：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| インデックス | 禁止された SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
~~~