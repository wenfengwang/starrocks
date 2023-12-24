---
displayed_sidebar: English
---

# group_concat

## 描述

将组中的非空值连接成一个字符串，使用 `sep` 参数，默认情况下为`,`。此函数可用于将列的多行值连接成一个字符串。

group_concat 在 3.0.6 版本之后和 3.1.3 版本之后支持 DISTINCT 和 ORDER BY。

## 语法

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## 参数

- `expr`：要连接的值，忽略 null 值。它必须计算为 VARCHAR。您可以选择指定 `DISTINCT` 以消除输出字符串中的重复值。如果要直接连接多个 `expr`，请使用 [concat](../string-functions/concat.md) 或 [concat_ws](../string-functions/concat_ws.md) 来指定格式。
- ORDER BY 中的项可以是无符号整数（从 1 开始）、列名或普通表达式。默认情况下，结果按升序排序。您也可以显式指定 ASC 关键字。如果要按降序对结果进行排序，请在要排序的列名后添加 DESC 关键字。
- `sep`：用于连接不同行中的非 null 值的可选分隔符。如果未指定，默认使用`,`（逗号）。要消除分隔符，请指定一个空字符串 `''`。

> **注意**
>
> 从 v3.0.6 和 v3.1.3 开始，当您指定分隔符时会发生行为更改。您必须使用 `SEPARATOR` 来声明分隔符，例如，`select group_concat(name SEPARATOR '-') as res from ss;`。

## 返回值

对于每个组返回一个字符串值，如果没有非 NULL 值，则返回 NULL。

您可以通过设置 [session variable](../../../reference/System_variable.md) `group_concat_max_len` 来限制 group_concat 返回的字符串长度，默认为 1024。最小值：4。单位：字符。

例：

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <value>;
```

## 例子

1. 创建一个包含科目分数的表 `ss`。

   ```sql
   CREATE TABLE `ss` (
     `id` int(11) NULL COMMENT "",
     `name` varchar(255) NULL COMMENT "",
     `subject` varchar(255) NULL COMMENT "",
     `score` int(11) NULL COMMENT ""
   ) ENGINE=OLAP
   DUPLICATE KEY(`id`)
   DISTRIBUTED BY HASH(`id`) BUCKETS 4
   PROPERTIES (
   "replication_num" = "1"
   );

   insert into ss values (1,"Tom","English",90);
   insert into ss values (1,"Tom","Math",80);
   insert into ss values (2,"Tom","English",NULL);
   insert into ss values (2,"Tom",NULL,NULL);
   insert into ss values (3,"May",NULL,NULL);
   insert into ss values (3,"Ti","English",98);
   insert into ss values (4,NULL,NULL,NULL);
   insert into ss values (NULL,"Ti","Phy",98);

   select * from ss order by id;
   +------+------+---------+-------+
   | id   | name | subject | score |
   +------+------+---------+-------+
   | NULL | Ti   | Phy     |    98 |
   |    1 | Tom  | English |    90 |
   |    1 | Tom  | Math    |    80 |
   |    2 | Tom  | English |  NULL |
   |    2 | Tom  | NULL    |  NULL |
   |    3 | May  | NULL    |  NULL |
   |    3 | Ti   | English |    98 |
   |    4 | NULL | NULL    |  NULL |
   +------+------+---------+-------+
   ```

2. 使用 group_concat。
  
  示例 1：将名称连接成一个字符串，使用默认分隔符并忽略 null 值。保留重复的名称。

  ```sql
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
  ```

  示例 2：将名称连接成一个字符串，用分隔符 `-` 连接，并忽略 null 值。保留重复的名称。

  ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
  ```

  示例 3：将不同的名称连接成一个字符串，使用默认分隔符并忽略 null 值。删除重复的名称。

  ```sql
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
  ```

  示例 4：按 `score` 的升序连接相同 ID 的名称-主题字符串。例如，`TomMath` 和 `TomEnglish` 共享 ID 1，它们按 `score` 的升序用逗号连接。

  ```sql
   select id, group_concat(distinct name,subject order by score) as res from ss group by id order by id;
   +------+--------------------+
   | id   | res                |
   +------+--------------------+
   | NULL | TiPhy              |
   |    1 | TomMath,TomEnglish |
   |    2 | TomEnglish         |
   |    3 | TiEnglish          |
   |    4 | NULL               |
   +------+--------------------+
   ```

  示例 5：group_concat 嵌套在 concat() 中，用于将 `name`、`-` 和 `subject` 组合为字符串。同一行中的字符串按 `score` 的升序排序。
  
  ```sql
   select id, group_concat(distinct concat(name, '-',subject) order by score) as res from ss group by id order by id;
   +------+----------------------+
   | id   | res                  |
   +------+----------------------+
   | NULL | Ti-Phy               |
   |    1 | Tom-Math,Tom-English |
   |    2 | Tom-English          |
   |    3 | Ti-English           |
   |    4 | NULL                 |
   +------+----------------------+
   ```
  
  示例 6：未找到匹配结果，返回 NULL。

  ```sql
  select group_concat(distinct name) as res from ss where id < 0;
   +------+
   | res  |
   +------+
   | NULL |
   +------+
   ```

  示例 7：将返回的字符串的长度限制为六个字符。

  ```sql
   set group_concat_max_len = 6;

   select id, group_concat(distinct name,subject order by score) as res from ss group by id order by id;
   +------+--------+
   | id   | res    |
   +------+--------+
   | NULL | TiPhy  |
   |    1 | TomMat |
   |    2 | NULL   |
   |    3 | TiEngl |
   |    4 | NULL   |
   +------+--------+
   ```

## 关键词

GROUP_CONCAT，CONCAT，ARRAY_AGG
