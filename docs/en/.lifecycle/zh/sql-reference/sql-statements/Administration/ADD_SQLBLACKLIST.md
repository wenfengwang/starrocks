---
displayed_sidebar: English
---

# 添加 SQL 黑名单

## 描述

向 SQL 黑名单中添加正则表达式，以禁止特定的 SQL 模式。启用 SQL 黑名单功能后，StarRocks 会将待执行的所有 SQL 语句与黑名单中的 SQL 正则表达式进行比对。StarRocks 不会执行与黑名单中任何正则表达式匹配的 SQL，并返回错误。这可以防止某些 SQL 触发集群崩溃或出现意外行为。

有关 SQL 黑名单的更多信息，请参阅[管理 SQL 黑名单](../../../administration/Blacklist.md)。

:::提示

此操作需要 SYSTEM 级别的 BLACKLIST 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 语法

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## 参数

`sql_reg_expr`：用于指定特定 SQL 模式的正则表达式。为了区分 SQL 语句中的特殊字符和正则表达式中的特殊字符，您需要在 SQL 语句中使用转义字符 `\` 作为特殊字符的前缀，比如 `(`、 `)` 和 `+`。尽管 `(` 和 `)` 经常用于 SQL 语句中，但 StarRocks 可以直接识别 SQL 语句中的 `(` 和 `)`，因此您无需为 `(` 和 `)` 使用转义字符。

## 例子

示例1：将 `count(\*)` 添加到 SQL 黑名单中。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

示例2：将 `count(distinct )` 添加到 SQL 黑名单中。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

示例3：将 `order by limit x, y, 1 <= x <=7, 5 <=y <=7` 添加到 SQL 黑名单中。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

示例4：将复杂的 SQL 正则表达式添加到 SQL 黑名单中。此示例演示了如何在 SQL 语句中使用转义字符 `*` 和 `-`。

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