---
displayed_sidebar: "Chinese"
---

# 黑名单管理

在某些情况下，管理员需要禁用某些SQL模式，以避免SQL触发集群崩溃或意外高并发查询。

StarRocks允许用户添加、查看和删除SQL黑名单。

## 语法

通过 `enable_sql_blacklist` 启用SQL黑名单。默认为False（关闭）。

~~~sql
admin set frontend config ("enable_sql_blacklist" = "true")
~~~

拥有ADMIN_PRIV权限的管理员用户可以通过执行以下命令来管理黑名单：

~~~sql
ADD SQLBLACKLIST #sql# 
DELETE SQLBLACKLIST #sql# 
SHOW SQLBLACKLISTS  
~~~

* 当 `enable_sql_blacklist` 为true时，每个SQL查询都需要通过sql黑名单进行过滤。如果匹配，用户将收到SQL在黑名单中的信息。否则，SQL将正常执行。当SQL被列入黑名单时，可能会出现以下消息：

`ERROR 1064 (HY000): Access denied; sql 'select count (*) from test_all_type_select_2556' is in blacklist`

## 添加黑名单

~~~sql
ADD SQLBLACKLIST #sql#
~~~

**#sql#** 是某种类型的SQL的正则表达式。由于SQL本身包含常见字符 `(`, `)`, `*`, `.` 可能与正则表达式的语义混淆，因此我们需要通过转义字符来区分它们。鉴于 `(` 和 `)` 在SQL中经常被使用，所以不需要使用转义字符。其他特殊字符需要使用转义字符 `\` 作为前缀。例如：

* 禁止 `count(\*)`:

~~~sql
ADD SQLBLACKLIST "select count(\\*) from .+"
~~~

* 禁止 `count(distinct)`:

~~~sql
ADD SQLBLACKLIST "select count(distinct .+) from .+"
~~~

* 禁止 order by limit `x`, `y`, `1 <= x <=7`, `5 <=y <=7`:

~~~sql
ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]"
~~~

* 禁止复杂SQL:

~~~sql
ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable"
~~~

## 查看黑名单

~~~sql
SHOW SQLBLACKLIST
~~~

结果格式: `索引 | 禁止的SQL`

例如:

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 索引  | 禁止的SQL                                                                                                                                                                                                                                                                                            |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
| 3     | select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]                                                                                                                                                                                                                            |
| 4     | select count\(distinct .+\) from .+                                                                                                                                                                                                                                                                    |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~

`禁止的SQL` 中显示的SQL已对所有SQL语义字符进行了转义。

## 删除黑名单

~~~sql
DELETE SQLBLACKLIST #indexlist#
~~~

例如，删除上述黑名单中的 sqlblacklist 3 和 4：

~~~sql
delete sqlblacklist  3, 4;   -- #indexlist# 是用逗号（,）分隔的ID列表。
~~~

然后，剩余的 sqlblacklist 如下:

~~~sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 索引  | 禁止的SQL                                                                                                                                                                                                                                                                                            |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

~~~