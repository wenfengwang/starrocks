---
displayed_sidebar: "Chinese"
---

# ADD SQLBLACKLIST

## 功能

Adds an SQL regular expression to the SQL blacklist. When the SQL blacklist feature is enabled, StarRocks compares all SQL statements that need to be executed with the SQL regular expressions in the blacklist. StarRocks will not execute any SQL that matches any regular expression in the blacklist and will return an error.

For more information about SQL blacklist, please refer to [Managing SQL Blacklist](../../../administration/Blacklist.md).

## 语法

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## 参数说明

`sql_reg_expr`: Specifies the regular expression of the SQL statement. In order to distinguish special characters in the SQL statement from those in the regular expression, you need to use the escape character (\) as the prefix for special characters in the SQL statement, such as `(`, `)`, and `+`. Since `(` and `)` are frequently used in SQL statements, StarRocks can directly recognize `(` and `)` in SQL statements. Therefore, you do not need to add escape characters for `(` and `)`.

## 示例

Example 1: Add `count(\*)` to the SQL blacklist.

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

Example 2: Add `count(distinct )` to the SQL blacklist.

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

Example 3: Add `order by limit x, y, 1 <= x <=7, 5 <=y <=7` to the SQL blacklist.

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

Example 4: Add a complex SQL regular expression to the SQL blacklist. This example is to demonstrate how to use escape characters to represent `*` and `-` in the SQL statement.

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