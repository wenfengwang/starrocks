---
displayed_sidebar: "Chinese"
---

# group_concat

## 描述

将一个组中的非空值连接成一个字符串，可以使用 `sep` 参数来指定连接的分隔符，如果不指定，默认为逗号`,`。该函数可用于将同一列中多行的值连接成一个字符串。

group_concat 支持 DISTINCT 和 ORDER BY，要求版本不低于 3.0.6 的 3.0 版本和不低于 3.1.3 的 3.1 版本。

## 语法

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## 参数

- `expr`: 要连接的值，忽略空值。它必须评估为VARCHAR。您可以选择指定 `DISTINCT` 来消除输出字符串中的重复值。如果要直接连接多个`expr`，请使用 [concat](../string-functions/concat.md) 或 [concat_ws](../string-functions/concat_ws.md) 来指定格式。
- ORDER BY 中的项可以是无符号整数（从1开始）、列名或普通表达式。默认按升序排列结果。您也可以明确指定 ASC 关键字。如果要按降序排序结果，请在正在排序的列名后添加 DESC 关键字。
- `sep`：可选的分隔符，用于连接不同行的非空值。如果未指定，将默认使用逗号`,`。要消除分隔符，请指定空字符串`''`。

> **注意**
>
> 从3.0.6和3.1.3版本开始，在指定分隔符时有了行为变化。您必须使用 `SEPARATOR` 来声明分隔符，例如 `select group_concat(name SEPARATOR '-') as res from ss;`。

## 返回值

对于每个组返回一个字符串值，如果没有非NULL值，则返回NULL。

可以通过设置 [会话变量](../../../reference/System_variable.md) `group_concat_max_len` 来限制 group_concat 返回的字符串长度，默认为1024。最小值为4。单位：字符。

示例：

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <value>;
```

## 示例

1. 创建包含主题分数的表 `ss`。

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
  
  示例1：将姓名连接为一个字符串，默认使用分隔符并忽略空值。重复的姓名被保留。

  ```sql
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
  ```

  示例2：将姓名连接为一个字符串，以分隔符`-`连接并忽略空值。重复的姓名被保留。

  ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
  ```

  示例3：将不同的姓名连接为一个字符串，并忽略空值。重复的姓名被移除。

  ```sql
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
  ```

  示例4：按`score`的升序连接相同ID的姓名-学科字符串。例如，`TomMath` 和 `TomEnglish` 共享ID 1，并且它们按`score`的升序用逗号连接。

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

  示例5：group_concat 与 concat() 嵌套，用于将`name`、`-` 和 `subject` 组合为一个字符串。在同一行中的字符串按`score`的升序排序。
  
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