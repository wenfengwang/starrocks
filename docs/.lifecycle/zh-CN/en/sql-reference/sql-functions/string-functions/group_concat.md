---
displayed_sidebar: "Chinese"
---

# group_concat

## 描述

将组中的非空值连接成一个字符串，带有“sep”参数，默认情况下为`,`，如果未指定。此函数可用于将列的多行值连接成一个字符串。

group_concat支持3.0.6及更高版本，以及3.1.3及更高版本中的DISTINCT和ORDER BY。

## 语法

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## 参数

- `expr`：要连接的值，忽略空值。它必须求值为VARCHAR。您可以选择使用`DISTINCT`从输出字符串中排除重复值。如果要直接连接多个`expr`，请使用[concat](./concat.md)或[concat_ws](./concat_ws.md)指定格式。
- ORDER BY中的项目可以是无符号整数（从1开始）、列名称或正常表达式。默认情况下，结果按升序排序。您还可以明确指定ASC关键字。如果要按降序排序结果，请将DESC关键字添加到要排序的列的名称上。
- `sep`：连接来自不同行的非空值的可选分隔符。如果未指定，默认情况下使用`,`（逗号）。要消除分隔符，请指定一个空字符串`''`。

> **注意**
>
> 从v3.0.6和v3.1.3开始，当指定分隔符时有行为更改。必须使用`SEPARATOR`声明分隔符，例如`select group_concat(name SEPARATOR '-') as res from ss;`.

## 返回值

为每个组返回一个字符串值，如果没有非NULL值，则返回NULL。

您可以通过设置[session variable](../../../reference/System_variable.md)`group_concat_max_len`来限制group_concat返回的字符串长度，默认为1024。最小值：4。单位：字符。

示例：

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <value>;
```

## 示例

1. 创建一个包含科目分数的表`ss`。

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
  
  示例1：将名字连接成一个字符串，使用默认分隔符，并忽略空值。重复的名字会被保留。
  
  ```sql
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
  ```

  示例2：将名字连接成一个字符串，连接分隔符为“-”，并忽略空值。重复的名字会被保留。
  
  ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
  ```

  示例3：将不同的名字连接成一个字符串，使用默认分隔符，并忽略空值。重复的名字会被移除。

  ```sql
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
  ```

  示例4：按`score`的升序连接相同ID的名字-科目字符串。例如，`TomMath`和`TomEnglish`共享ID 1，并按照`score`的升序以逗号连接。
  
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

  示例5：group_concat与concat()嵌套使用，将`name`、`-`、`subject`组合成一个字符串。在`score`的升序中对同一行中的字符串进行排序。

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

## 关键词

GROUP_CONCAT,CONCAT,ARRAY_AGG