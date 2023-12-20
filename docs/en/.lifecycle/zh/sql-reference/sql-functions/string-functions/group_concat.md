---
displayed_sidebar: English
---

# group_concat

## 描述

使用`sep`参数将组中的非空值连接成单个字符串，如果未指定，则默认为`,`。此函数可用于将一列中多行的值连接成一个字符串。

group_concat在3.0.6之后的3.0版本和3.1.3之后的3.1版本中支持DISTINCT和ORDER BY。

## 语法

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## 参数

- `expr`: 要连接的值，忽略空值。它必须评估为VARCHAR。您可以选择指定`DISTINCT`以消除输出字符串中的重复值。如果您想直接连接多个`expr`，请使用[concat](./concat.md)或[concat_ws](./concat_ws.md)来指定格式。
- ORDER BY中的项可以是无符号整数（从1开始）、列名或普通表达式。默认情况下，结果按升序排序。您还可以显式指定ASC关键字。如果您想按降序对结果进行排序，请将DESC关键字添加到您要排序的列名中。
- `sep`: 可选的分隔符，用于连接不同行中的非空值。如果未指定，则默认使用`,`（逗号）。要消除分隔符，请指定空字符串`''`。

> **注意**
> 从v3.0.6和v3.1.3开始，当您指定分隔符时行为有所变化。您必须使用`SEPARATOR`来声明分隔符，例如，`select group_concat(name SEPARATOR '-') as res from ss;`。

## 返回值

为每个组返回一个字符串值，如果没有非NULL值，则返回NULL。

您可以通过设置[会话变量](../../../reference/System_variable.md) `group_concat_max_len`来限制group_concat返回的字符串长度，默认为1024。最小值：4。单位：字符。

示例：

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <value>;
```

## 示例

1. 创建一个表`ss`，其中包含科目分数。

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

2. 使用group_concat。

示例1：使用默认分隔符将名称连接成一个字符串，并忽略空值。重复的名称被保留。

```sql
 select group_concat(name) as res from ss;
 +---------------------------+
 | res                       |
 +---------------------------+
 | Tom,Tom,Ti,Tom,Tom,May,Ti |
 +---------------------------+
```

示例2：将名称连接成一个字符串，通过分隔符`-`连接，并忽略空值。重复的名称被保留。

```sql
 select group_concat(name SEPARATOR '-') as res from ss;
 +---------------------------+
 | res                       |
 +---------------------------+
 | Ti-May-Ti-Tom-Tom-Tom-Tom |
 +---------------------------+
```

示例3：使用默认分隔符将不同的名称连接成一个字符串，并忽略空值。重复的名称被移除。

```sql
 select group_concat(distinct name) as res from ss;
 +---------------------------+
 | res                       |
 +---------------------------+
 | Ti,May,Tom                |
 +---------------------------+
```

示例4：按`score`升序将相同ID的name-subject字符串连接。例如，`TomMath`和`TomEnglish`共享ID 1，并且它们按`score`升序用逗号连接。

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

示例5：group_concat与concat()嵌套使用，将`name`、`-`和`subject`组合成一个字符串。同一行中的字符串按`score`升序排序。

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

示例6：未找到匹配结果，返回NULL。

```sql
select group_concat(distinct name) as res from ss where id < 0;
 +------+
 | res  |
 +------+
 | NULL |
 +------+
```

示例7：将返回的字符串长度限制为六个字符。

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

## 关键字

GROUP_CONCAT, CONCAT, ARRAY_AGG