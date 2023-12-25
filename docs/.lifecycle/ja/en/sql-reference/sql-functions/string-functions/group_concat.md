---
displayed_sidebar: English
---

# group_concat

## 説明

グループ内のnullでない値を単一の文字列に連結します。`sep` 引数はデフォルトで `,` ですが、指定されていない場合は変更されます。この関数は、列の複数の行からの値を一つの文字列に連結するために使用できます。

group_concatは、バージョン3.0.6以降の3.0系とバージョン3.1.3以降の3.1系でDISTINCTとORDER BYをサポートしています。

## 構文

```SQL
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## パラメーター

- `expr`: 連結する値で、null値は無視されます。VARCHARとして評価される必要があります。出力文字列から重複する値を排除するために`DISTINCT`を指定することもできます。複数の`expr`を直接連結したい場合は、[concat](./concat.md)または[concat_ws](./concat_ws.md)を使用してフォーマットを指定してください。
- ORDER BYには符号なし整数（1から始まる）、列名、または通常の式を指定できます。結果はデフォルトで昇順にソートされます。昇順を明示的に指定するにはASCキーワードを使用します。結果を降順でソートしたい場合は、ソートする列名にDESCキーワードを追加してください。
- `sep`: 異なる行からのnullでない値を連結するために使用されるオプションのセパレータです。指定されていない場合はデフォルトで`,`（カンマ）が使用されます。セパレータを省略するには、空文字列`''`を指定してください。

> **注記**
>
> v3.0.6およびv3.1.3以降、セパレータを指定する際の挙動が変更されました。セパレータを宣言するには`SEPARATOR`を使用する必要があります。例えば、`select group_concat(name SEPARATOR '-') as res from ss;`のようにします。

## 戻り値

各グループに対して文字列値を返し、nullでない値がない場合はNULLを返します。

group_concatによって返される文字列の長さを制限するには、[セッション変数](../../../reference/System_variable.md) `group_concat_max_len`を設定します。デフォルトは1024です。最小値：4。単位は文字です。

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
  
  例 1: デフォルトのセパレータを使用して、null値を無視して名前を文字列に連結します。重複する名前は保持されます。

  ```sql
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
  ```

  例 2: セパレータ`-`を使用して、null値を無視して名前を文字列に連結します。重複する名前は保持されます。

  ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
  ```

  例 3: デフォルトのセパレータを使用して、null値を無視して異なる名前を文字列に連結します。重複する名前は削除されます。

  ```sql
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
  ```

  例 4: 同じIDの名前と科目の文字列を`score`の昇順で連結します。例えば、`TomMath`と`TomEnglish`はID 1を共有し、`score`の昇順でカンマで連結されます。

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

 例 5: group_concatはconcat()とネストされ、`name`、`-`、`subject`を文字列に組み合わせます。同じ行の文字列は`score`の昇順でソートされます。
  
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
  
  例 6: 一致する結果が見つからず、NULLが返されます。

  ```sql
  select group_concat(distinct name) as res from ss where id < 0;
   +------+
   | res  |
   +------+
   | NULL |
   +------+
   ```

  例 7: 返される文字列の長さを6文字に制限します。

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

GROUP_CONCAT, CONCAT, ARRAY_AGG
