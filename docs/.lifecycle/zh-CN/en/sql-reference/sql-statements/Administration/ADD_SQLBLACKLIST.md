---
displayed_sidebar: "Chinese"
---

# 添加SQL黑名单

## 描述

向SQL黑名单中添加正则表达式以禁止特定的SQL模式。当启用SQL黑名单功能时，StarRocks会将所有要执行的SQL语句与黑名单中的SQL正则表达式进行比较。StarRocks不会执行与黑名单中任何正则表达式匹配的SQL，并返回错误。这可以防止某些SQL触发集群崩溃或出现意外行为。

有关SQL黑名单的更多信息，请参见[管理SQL黑名单](../../../administration/Blacklist.md)。

> **注意**
>
> 只有具有ADMIN特权的用户可以向SQL黑名单中添加SQL正则表达式。

## 语法

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## 参数

`sql_reg_expr`：用于指定特定SQL模式的正则表达式。为了区分SQL语句中的特殊字符和正则表达式中的特殊字符，您需要使用转义字符`\`作为SQL语句中特殊字符的前缀，例如`(`、`)`和`+`。虽然`(`和`)`经常在SQL语句中使用，但StarRocks可以直接识别SQL语句中的`(`和`)`。因此，对于`(`和`)`，您无需使用转义字符。

## 示例

示例1：将`count(\*)`添加到SQL黑名单中。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

示例2：将`count(distinct )`添加到SQL黑名单中。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

示例3：将`order by limit x, y, 1 <= x <=7, 5 <=y <=7`添加到SQL黑名单中。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

示例4：将复杂的SQL正则表达式添加到SQL黑名单中。该示例演示了如何在SQL语句中使用转义字符`*`和`-`。

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