---
displayed_sidebar: English
---

# 黑名单管理

在某些情况下，管理员需要禁用某些 SQL 模式，以避免 SQL 触发集群崩溃或意外的高并发查询。

StarRocks 支持添加、查看和删除 SQL 黑名单。

## 语法

通过 `enable_sql_blacklist` 启用 SQL 黑名单。默认值为 False（关闭）。

~~~sql
admin set frontend config ("enable_sql_blacklist" = "true")
~~~

具有 ADMIN_PRIV 权限的管理员用户可以通过执行以下命令来管理黑名单：

~~~sql
ADD SQLBLACKLIST #sql# 
DELETE SQLBLACKLIST #sql# 
SHOW SQLBLACKLISTS  
~~~

* 当 `enable_sql_blacklist` 为 true 时，每个 SQL 查询都需要通过 sqlblacklist 进行筛选。如果匹配，则通知用户该 SQL 在黑名单中。否则，SQL 将正常执行。当 SQL 被列入黑名单时，该消息可能如下所示：

`ERROR 1064 (HY000): Access denied; sql 'select count (*) from test_all_type_select_2556' is in blacklist`

## 添加黑名单

~~~sql
ADD SQLBLACKLIST #sql#
~~~

**#sql#** 是某种类型 SQL 的正则表达式。由于 SQL 本身包含通用字符 `(`, `)`, `*`, `.` 可能会与正则表达式的语义混淆，因此我们需要使用转义字符来区分它们。鉴于 `(` 和 `)` 在 SQL 中使用得太频繁，因此没有必要使用转义字符。其他特殊字符需要使用转义字符 `\` 作为前缀。例如：

* 禁止 `count(\*)`:

~~~sql
ADD SQLBLACKLIST "select count(\\*) from .+"
~~~

* 禁止 `count(distinct)`:

~~~sql
ADD SQLBLACKLIST "select count(distinct .+) from .+"
~~~

* 禁止按照限制排序 `x`，`y`，`1 <= x <=7`，`5 <=y <=7`:

~~~sql
ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]"
~~~

* 禁止复杂的 SQL:

~~~sql
ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable"
~~~

## 查看黑名单

~~~sql
SHOW SQLBLACKLIST
~~~

结果格式： `Index | Forbidden SQL`

例如：

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

`Forbidden SQL` 中显示的 SQL 对于所有 SQL 语义字符都进行了转义。

## 删除黑名单

~~~sql
DELETE SQLBLACKLIST #indexlist#
~~~

例如，删除上述黑名单中的 sqlblacklist 3 和 4：

~~~sql
delete sqlblacklist  3, 4;   -- #indexlist# 是由逗号（,）分隔的 ID 列表。
~~~

然后，剩余的 sqlblacklist 如下所示：

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~
