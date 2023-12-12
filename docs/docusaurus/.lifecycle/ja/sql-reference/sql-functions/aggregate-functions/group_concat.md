```yaml
---
displayed_sidebar: "Japanese"
---

# group_concat

## Description

グループ内の非NULL値を`sep`引数で1つの文字列に連結し、デフォルトでは`,`が指定されていない場合、デフォルトのセパレーターとして使用されます。この関数は、1つの文字列に複数の行の値を連結するために使用できます。

group_concatは、DISTINCTおよびORDER BYをサポートしています. バージョン3.0.6以降および3.1.3以降のバージョンです。

## Syntax

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## パラメータ

- `expr`: 連結する値で、NULL値は無視されます. VARCHARに評価する必要があります。重複する値を出力文字列から排除するにはオプションで`DISTINCT`を指定できます。複数の`expr`を直接連結する場合は、[concat](../string-functions/concat.md)または[concat_ws](../string-functions/concat_ws.md)を使用して書式を指定します。
- ORDER BYでの項目は、符号なし整数（1から始まる）、列名、または通常の式が指定できます。結果はデフォルトで昇順で並べ替えられます。昇順に並べ替える場合は、ASCキーワードを明示的に指定できます。結果を降順で並べ替える場合は、並べ替える列の名前にDESCキーワードを追加します。
- `sep`: 異なる行の非NULL値を連結するために使用されるオプションのセパレーターです。指定されていない場合、デフォルトで`,`（カンマ）が使用されます。セパレーターを除外するには、空の文字列`' '`を指定します。

> **注意**
>
> v3.0.6およびv3.1.3以降、セパレーターを指定する場合の動作が変更されます。セパレーターを宣言するために`SEPARATOR`を使用する必要があります。例：`select group_concat(name SEPARATOR '-') as res from ss;`。

## 戻り値

各グループに対する文字列値を返し、非NULL値がない場合はNULLを返します。

group_concatによって返される文字列の長さを[session variable](../../../reference/System_variable.md) `group_concat_max_len`を設定することで制限することができます。デフォルト値は1024で、最小値は4です。単位は文字です。

例：

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <value>;
```

## 例

1. 科目のスコアを含むテーブル`ss`を作成します。

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

2. group_concatを使用します。

  Example 1: デフォルトのセパレーターを使用し、NULL値を無視して名前を文字列に連結します。重複した名前は保持されます。

  ```sql
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
  ```

  Example 2: 名前をセパレーター`-`で接続し、NULL値を無視して名前を文字列に連結します。重複した名前は保持されます。

  ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
  ```

  Example 3: デフォルトのセパレーターを使用して重複する名前を排除し、名前を文字列に連結します。 

  ```sql
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
  ```

  Example 4: `score`の昇順でIDが同じ名前-科目の文字列を連結します。例えば、`TomMath`と`TomEnglish`はID 1を共有しており、`score`の昇順でカンマで連結されています。

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

  Example 5: group_concatはコンビネーション関数concat()とネストされ、`name`、`-`、`subject`を文字列として結合します。同じ行の文字列は`score`の昇順でソートされます。

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
  
  Example 6: 一致する結果が見つからず、NULLが返されます。

  ```sql
  select group_concat(distinct name) as res from ss where id < 0;
   +------+
   | res  |
   +------+
   | NULL |
   +------+
   ```

  Example 7: 返される文字列の長さを6文字に制限します。

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

## キーワード

GROUP_CONCAT,CONCAT,ARRAY_AGG
```