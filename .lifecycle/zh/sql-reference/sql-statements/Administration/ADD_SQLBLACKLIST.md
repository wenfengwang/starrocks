---
displayed_sidebar: English
---

# 向SQL黑名单中添加规则

## 说明

向SQL黑名单中添加正则表达式，以禁止特定的SQL模式。启用SQL黑名单功能后，StarRocks会将所有待执行的SQL语句与黑名单中的正则表达式进行对比。匹配黑名单中任一正则表达式的SQL将不会被执行，并会返回错误信息。这样做可以防止某些SQL语句引发集群崩溃或其他意外行为。

更多关于SQL黑名单的信息，请参阅[管理SQL黑名单](../../../administration/Blacklist.md)。

:::提示

此操作需要SYSTEM-level的**BLACKLIST**权限。您可以按照[GRANT](../account-management/GRANT.md)中的说明来授予此权限。

:::

## 语法

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## 参数

sql_reg_expr：用于指定特定SQL模式的正则表达式。为了区分SQL语句和正则表达式中的特殊字符，您需要使用转义字符\作为SQL语句中特殊字符的前缀，如(、)和+。由于(和)在SQL语句中经常使用，StarRocks能够直接识别SQL语句中的(和)。您不需要为(和)使用转义字符。

## 示例

示例1：将count(\*)添加到SQL黑名单。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

示例2：将count(distinct )添加到SQL黑名单。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

示例3：将order by limit x, y, 1 <= x <=7, 5 <=y <=7添加到SQL黑名单。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

示例4：向SQL黑名单添加一个复杂的SQL正则表达式。此示例展示了如何在SQL语句中为*和-使用转义字符。

```Plain
mysql> ADD SQLBLACKLIST 
    "select id_int \\* 4, id_tinyint, id_varchar 
        from test_all_type_nullable 
    except select id_int, id_tinyint, id_varchar 
        from test_basic 
    except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar 
        from test_all_type_nullable2 
    except select id_int, id_tinyint, id_varchar 
        from test_basic_nullable";
```
