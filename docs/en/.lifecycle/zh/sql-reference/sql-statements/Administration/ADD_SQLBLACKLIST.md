---
displayed_sidebar: English
---

# 添加 SQLBLACKLIST

## 描述

添加一个正则表达式到 SQL 黑名单，以禁止某些 SQL 模式。当 SQL 黑名单功能被启用时，StarRocks 会将所有待执行的 SQL 语句与黑名单中的 SQL 正则表达式进行比较。StarRocks 不会执行任何与黑名单中的正则表达式相匹配的 SQL，并会返回错误信息。这可以防止某些 SQL 触发集群崩溃或不预期的行为。

更多关于 SQL 黑名单的信息，请参见[管理 SQL 黑名单](../../../administration/Blacklist.md)。

:::tip

此操作需要 SYSTEM 级别的 BLACKLIST 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的指南来授予此权限。

:::

## 语法

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## 参数

`sql_reg_expr`：用于指定特定 SQL 模式的正则表达式。为了区分 SQL 语句中的特殊字符和正则表达式中的特殊字符，您需要使用转义字符 `\` 作为 SQL 语句中特殊字符的前缀，例如 `(`、`)` 和 `+`。由于 `(` 和 `)` 在 SQL 语句中经常使用，StarRocks 可以直接识别 SQL 语句中的 `(` 和 `)`，因此您不需要为 `(` 和 `)` 使用转义字符。

## 示例

示例 1：将 `count(*)` 添加到 SQL 黑名单。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

示例 2：将 `count(distinct )` 添加到 SQL 黑名单。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

示例 3：将 `order by limit x, y, 1 <= x <=7, 5 <=y <=7` 添加到 SQL 黑名单。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

示例 4：将一个复杂的 SQL 正则表达式添加到 SQL 黑名单。本示例演示了如何对 SQL 语句中的 `*` 和 `-` 使用转义字符。

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