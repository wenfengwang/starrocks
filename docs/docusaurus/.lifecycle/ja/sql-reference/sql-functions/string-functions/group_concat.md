---
displayed_sidebar: "Japanese"
---

# group_concat

## 説明

グループ内の非NULL値を、`sep`引数で指定された``,`（デフォルト）を使用して単一の文字列に連結します。この機能は、複数の行の値を1つの文字列に連結するために使用できます。

group_concatは、3.0.6より後の3.0バージョンおよび3.1.3より後の3.1バージョンで、DISTINCTおよびORDER BYをサポートしています。

## 構文

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## パラメーター

- `expr`: NULL値を無視して連結する値を指定します。VARCHARに評価する必要があります。出力文字列から重複値を除外するには、オプションで`DISTINCT`を指定できます。複数の`expr`を直接連結する場合は、[concat](./concat.md)または[concat_ws](./concat_ws.md)を使用して書式を指定します。
- ORDER BYのアイテムには、符号なし整数（1から始まる）、列名、または通常の式を指定できます。結果はデフォルトでは昇順で並べ替えられます。ASCキーワードを明示的に指定することもできます。結果を降順でソートする場合は、ソートしている列名にDESCキーワードを追加します。
- `sep`: 異なる行の非NULL値を連結するために使用するオプションのセパレータです。指定されていない場合は、デフォルトで`,`（カンマ）が使用されます。セパレータを除外するには、空の文字列`''`を指定します。

> **注記**
>
> v3.0.6およびv3.1.3以降では、セパレータを指定する際の動作が変更されています。例えば、`select group_concat(name SEPARATOR '-') as res from ss;`のように、セパレータを宣言するために`SEPARATOR`を使用する必要があります。

## 戻り値

各グループに対する文字列値を返し、非NULL値がない場合はNULLを返します。

group_concatによって返される文字列の長さを、デフォルトで1024のセッション変数`group_concat_max_len`を設定することによって制限できます。最小値: 4、単位: 文字数。

例:

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <value>;
```

## 例

1. 科目の得点を含む`ss`という名前のテーブルを作成します。

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
  
  例1: デフォルトのセパレータとNULL値を無視して名前を文字列に連結します。重複した名前は保持されます。

  ```sql
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
  ```

  例2: デフォルトのセパレータとNULL値を無視して名前を文字列に連結し、セパレータ`-`で接続します。重複した名前は保持されます。

  ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
  ```

  例3: デフォルトのセパレータとNULL値を無視して重複しない名前を文字列に連結します。

  ```sql
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
  ```

  例4: `score`の昇順で同じIDの名前-科目文字列を連結します。例えば、`TomMath`と`TomEnglish`はID 1を共有し、`score`の昇順でカンマで連結されます。

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

  例5: group_concatはconcat()とネストされており、`name`、`-`、`subject`を文字列として結合するために使用されます。同じ行の文字列は`score`の昇順で並べ替えられます。
  
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