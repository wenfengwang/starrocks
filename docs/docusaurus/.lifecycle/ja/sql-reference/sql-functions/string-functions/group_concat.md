---
displayed_sidebar: "Japanese"
---

# group_concat

## 説明

グループ内の非NULL値を、`sep`引数を使用して1つの文字列に連結します。引数が指定されていない場合、デフォルトで`,`(カンマ)が使用されます。この関数は、列の複数の行から値を1つの文字列に連結するために使用できます。

group_concatは、3.0.6より後の3.0バージョンおよび3.1.3より後の3.1バージョンで、DISTINCTおよびORDER BYをサポートしています。

## 構文

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## パラメータ

- `expr`: 連結する値で、NULL値は無視されます。VARCHARに評価される必要があります。重複する値を出力文字列から排除するには、`DISTINCT`をオプションで指定できます。複数の`expr`を直接連結したい場合は、[concat](./concat.md)または[concat_ws](./concat_ws.md)を使用してフォーマットを指定します。
- ORDER BYのアイテムには、符号なし整数(1から始まる)、列名、または通常の式を使用できます。結果はデフォルトで昇順でソートされます。結果を降順でソートしたい場合は、ソートする列の名前にDESCキーワードを追加します。結果を降順でソートしたい場合は、ASCキーワードを明示的に指定することもできます。 
- `sep`: 異なる行の非NULL値を連結するために使用されるオプションのセパレータです。指定されていない場合は、デフォルトで`,`(カンマ)が使用されます。セパレータを除外するには、空の文字列`''`を指定します。

> **注意**
>
> v3.0.6およびv3.1.3以降、セパレータを指定する場合の動作が変更されます。たとえば、`select group_concat(name SEPARATOR '-') as res from ss;`としてセパレータを宣言するために`SEPARATOR`を使用する必要があります。

## 戻り値

各グループに対する文字列値を返し、非NULL値がない場合はNULLを返します。

group_concatによって返される文字列の長さを、デフォルト値が1024の[セッション変数](../../../reference/System_variable.md)`group_concat_max_len`を設定することで制限できます。最小値: 4。単位: 文字。

例:

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <値>;
```

## 例

1. 科目の成績が含まれる`ss`という名前のテーブルを作成します。

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

  例1: 名前をデフォルトのセパレータで文字列に連結し、NULL値を無視します。重複する名前は保持されます。

  ```sql
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
  ```

  例2: 名前を`-`で連結した文字列で、NULL値を無視します。重複する名前は保持されます。

  ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
  ```

  例3: 重複する名前を除去し、デフォルトのセパレータで一意の名前を文字列に連結します。

  ```sql
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
  ```

  例4: `score`の昇順で、同じIDの名前-科目文字列を連結します。たとえば、`TomMath`と`TomEnglish`はID1を共有し、`score`の昇順でカンマで連結されます。

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

  例5: group_concatをconcat()とネストして、`name`、`-`、`subject`を文字列として結合します。同じ行の文字列は`score`の昇順でソートされます。
  
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
  
  例6: 一致する結果が見つからず、NULLが返されます。

  ```sql
  select group_concat(distinct name) as res from ss where id < 0;
   +------+
   | res  |
   +------+
   | NULL |
   +------+
   ```

  例7: 返される文字列の長さを6文字に制限します。

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