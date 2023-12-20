---
displayed_sidebar: English
---

# 黑名单管理

在特定情况下，管理员需要禁用特定的SQL模式，以防止SQL触发集群崩溃或出现意料之外的高并发查询。

StarRocks允许用户添加、查看和删除SQL黑名单。

## 语法

通过enable_sql_blacklist启用SQL黑名单功能。默认值为False（关闭状态）。

```sql
admin set frontend config ("enable_sql_blacklist" = "true")
```

拥有ADMIN_PRIV权限的管理员用户可以通过执行以下命令来管理黑名单：

```sql
ADD SQLBLACKLIST #sql# 
DELETE SQLBLACKLIST #sql# 
SHOW SQLBLACKLISTS  
```

* 当enable_sql_blacklist设置为true时，每条SQL查询都需要经过sqlblacklist的过滤。如果匹配到，用户将被告知该SQL已被列入黑名单。否则，SQL将正常执行。当SQL被加入黑名单时，可能会出现以下信息提示：

错误 1064 (HY000)：访问被拒绝；SQL 'select count(*) from test_all_type_select_2556' 已在黑名单中

## 添加黑名单

```sql
ADD SQLBLACKLIST #sql#
```

**#sql#**是针对某种类型的SQL的正则表达式。鉴于SQL本身包含了常见的字符`(`, `)`, `*`, `.`，这些字符可能与正则表达式的语义混淆，所以我们需要使用转义字符进行区分。由于`(`和`)`在SQL中使用非常频繁，因此不需要使用转义字符。其他特殊字符需要在前面加上转义字符`\`。例如：

* 禁止count(\*)：

```sql
ADD SQLBLACKLIST "select count(\\*) from .+"
```

* 禁止count(distinct)：

```sql
ADD SQLBLACKLIST "select count(distinct .+) from .+"
```

* 禁止按limit x, y排序，其中1 <= x <= 7, 5 <= y <= 7：

```sql
ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]"
```

* 禁止复杂的SQL：

```sql
ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable"
```

## 查看黑名单

```sql
SHOW SQLBLACKLIST
```

结果格式：索引 | 禁止的SQL

例如：

```sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
| 3     | select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]                                                                                                                                                                                                                            |
| 4     | select count\(distinct .+\) from .+                                                                                                                                                                                                                                                                    |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

显示在“禁止的SQL”中的SQL已对所有SQL语义字符进行了转义处理。

## 删除黑名单

```sql
DELETE SQLBLACKLIST #indexlist#
```

例如，删除上述黑名单中的第3条和第4条sqlblacklist：

```sql
delete sqlblacklist  3, 4;   -- #indexlist# is a list of IDs separated by comma (,).
```

然后，剩余的sqlblacklist如下：

```sql
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```
