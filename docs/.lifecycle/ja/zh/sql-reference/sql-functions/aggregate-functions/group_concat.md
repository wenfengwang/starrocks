---
displayed_sidebar: Chinese
---

# group_concat

## 機能

グループ内の複数の非NULL値を一つの文字列に連結する。引数`sep`は文字列間の区切り文字で、この引数はオプションで、デフォルトは`,`です。この関数は連結時にNULL値を無視します。

バージョン3.0.6、3.1.3から、group_concatはDISTINCTとORDER BYを使用することができます。

## 文法

```Haskell
VARCHAR GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR sep])
```

## パラメータ説明

- `expr`: 連結される値で、VARCHAR型をサポートしています。DISTINCTキーワードを指定して、連結前にグループ内の重複値を除去することができます。複数の値を直接連結する場合は、[concat](../string-functions/concat.md)や[concat_ws](../string-functions/concat_ws.md)を使用して連結方法を指定できます。
- ORDER BYの後にはunsigned_integer（1から始まる）、列名、または一般的な式を続けることができます。ORDER BYは、連結される値を昇順または降順でソートするために使用されます。デフォルトは昇順です。降順でソートする場合はDESCを指定する必要があります。
- `sep`: 文字列間の区切り文字で、オプションです。指定しない場合は、デフォルトでコンマ`,`が区切り文字として使用されます。空文字で連結する場合は`''`を使用できます。

> **注記**
>
> バージョン3.0.6および3.1.3から、区切り文字は`SEPARATOR`キーワードを使用して宣言する必要があります。例えば、`select group_concat(name SEPARATOR '-') as res from ss;`のようにします。

## 戻り値の説明

戻り値のデータ型はVARCHARです。非NULL値がない場合はNULLを返します。

システム変数[group_concat_max_len](../../../reference/System_variable.md#group_concat_max_len)を使用して、返される最大文字長を制御できます。デフォルト値は1024です。最小値は4です。単位は文字です。

変数の設定方法：

```sql
SET [GLOBAL | SESSION] group_concat_max_len = <value>
```

## 例

1. テーブルを作成し、データを挿入します。

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

2. group_concatを使用して値を連結します。

   例1: `name`列の値を連結し、デフォルトの区切り文字を使用し、NULL値を無視します。データの重複は除去しません。

   ```plain
   select group_concat(name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Tom,Tom,Ti,Tom,Tom,May,Ti |
   +---------------------------+
   ```

   例2: `name`列の値を連結し、`SEPARATOR`を使用して区切り文字`-`を宣言し、NULL値を無視します。データの重複は除去しません。

   ```sql
   select group_concat(name SEPARATOR '-') as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti-May-Ti-Tom-Tom-Tom-Tom |
   +---------------------------+
   ```

   例3: `name`列の値を連結し、デフォルトの区切り文字を使用し、NULL値を無視します。DISTINCTを使用してデータの重複を除去します。

    ```plain
   select group_concat(distinct name) as res from ss;
   +---------------------------+
   | res                       |
   +---------------------------+
   | Ti,May,Tom                |
   +---------------------------+
   ```

   例4: 同じIDの`name`と`subject`を組み合わせて、`score`の昇順で連結します。例えば、返された例の2行目のデータ`TomMath,TomEnglish`は、IDが1の`TomMath`と`TomEnglish`を`score`で昇順にソートして連結したものです。

   ```plain
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

   例5: group_concatの中でconcat関数をネストします。concatは`name`と`subject`の連結形式を"name-subject"と指定します。クエリのロジックは例3と同じです。

   ```plain
   select id, group_concat(distinct concat(name,'-',subject) order by score) as res from ss group by id order by id;
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

   例6: 条件に合致する結果がない場合は、NULLを返します。

   ```plain
   select group_concat(distinct name) as res from ss where id < 0;
   +------+
   | res  |
   +------+
   | NULL |
   +------+
   ```

   例7: 返される文字列の最大長を6文字に制限します。クエリのロジックは例3と同じです。

   ```plain
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
