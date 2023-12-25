---
displayed_sidebar: Chinese
---

# INSERT文を使用してデータをインポートする

import InsertPrivNote from '../assets/commonMarkdown/insertPrivNote.md'

この記事では、INSERT文を使用してStarRocksにデータをインポートする方法について説明します。

MySQLなどのデータベースシステムと同様に、StarRocksはINSERT文を使用してデータをインポートすることをサポートしています。INSERT INTO VALUES文を使用して直接テーブルにデータを挿入することができますし、INSERT INTO SELECT文を使用して他のStarRocksテーブルのデータを新しいStarRocksテーブルにインポートすることもできます。また、[外部テーブル機能](../data_source/External_table.md)を通じて他のデータソースのデータをStarRocksの内部テーブルにインポートすることも可能です。v3.1以降では、INSERT文と[FILES()](../sql-reference/sql-functions/table-functions/files.md)関数を使用して、クラウドストレージやHDFS内のファイルを直接インポートすることができます。

バージョン2.4では、StarRocksはINSERT OVERWRITE文を使用して、目標テーブルに対して**上書き書き込み**をバッチでサポートするようになりました。INSERT OVERWRITE文は、以下の3つの操作を統合して上書き書き込みを実現します：

1. 目標パーティションに[一時パーティションを作成する](../table_design/Temporary_partition.md#一時パーティションの作成)
2. [一時パーティションにデータを書き込む](../table_design/Temporary_partition.md#一時パーティションへのデータインポート)
3. [一時パーティションを使用して目標パーティションを原子的に置き換える](../table_design/Temporary_partition.md#一時パーティションを使用した置き換え)

データを置き換える前に検証したい場合は、上記の手順に従ってデータの上書き書き込みを自分で実装することができます。

## 注意事項

- MySQLクライアントで`Ctrl` + `C`キーを使用して同期INSERTインポートタスクを強制的にキャンセルすることができます。
- [SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md)を使用して非同期INSERTインポートタスクを作成することができます。
- 現在のバージョンでは、StarRocksがINSERT文を実行する際に、データが目標テーブルのフォーマットに合致しない場合（例えば文字列が長すぎるなど）、INSERT操作はデフォルトで失敗します。セッション変数`enable_insert_strict`を`false`に設定することで、目標テーブルのフォーマットに合致しないデータをフィルタリングし、INSERT操作を続行することができます。
- 小規模なデータをINSERT文で頻繁にインポートすると、多くのデータバージョンが発生し、クエリのパフォーマンスに影響を与えるため、INSERT文を頻繁に使用してデータをインポートすることや、それを生産環境の日常的なルーチンインポートジョブとして使用することはお勧めしません。ビジネスシナリオがストリーミングインポートや小規模なデータを複数回インポートすることを要求する場合は、Apache Kafka®をデータソースとして使用し、[Routine Load](../loading/RoutineLoad.md)を通じてインポートジョブを行うことをお勧めします。
- INSERT OVERWRITE文を実行した後、システムは目標パーティションに対応する一時パーティションを作成し、データを一時パーティションに書き込み、最後に一時パーティションを使用して目標パーティションを[原子的に置き換える](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#一時パーティションを使用した元のパーティションの置き換え)ことで上書き書き込みを実現します。この全プロセスはLeader FEノードで実行されます。したがって、Leader FEノードが上書き書き込みプロセス中にダウンした場合、そのINSERT OVERWRITEインポートは失敗し、プロセス中に作成された一時パーティションも削除されます。

## 準備

### 権限を確認する

<InsertPrivNote />

### オブジェクトを作成する

StarRocksに`load_test`データベースを作成し、その中にインポート対象のテーブル`insert_wiki_edit`とデータソーステーブル`source_wiki_edit`を作成します。

> **説明**
>
> この記事で示されている操作例はすべて、テーブル`insert_wiki_edit`とデータソーステーブル`source_wiki_edit`に基づいています。自分のテーブルとデータを使用する場合は、このステップをスキップし、使用シナリオに応じてインポートするデータを変更してください。

```SQL
CREATE DATABASE IF NOT EXISTS load_test;
USE load_test;
CREATE TABLE insert_wiki_edit
(
    event_time      DATETIME,
    channel         VARCHAR(32)      DEFAULT '',
    user            VARCHAR(128)     DEFAULT '',
    is_anonymous    TINYINT          DEFAULT '0',
    is_minor        TINYINT          DEFAULT '0',
    is_new          TINYINT          DEFAULT '0',
    is_robot        TINYINT          DEFAULT '0',
    is_unpatrolled  TINYINT          DEFAULT '0',
    delta           INT              DEFAULT '0',
    added           INT              DEFAULT '0',
    deleted         INT              DEFAULT '0'
)
DUPLICATE KEY(
    event_time,
    channel,
    user,
    is_anonymous,
    is_minor,
    is_new,
    is_robot,
    is_unpatrolled
)
PARTITION BY RANGE(event_time)(
    PARTITION p06 VALUES LESS THAN ('2015-09-12 06:00:00'),
    PARTITION p12 VALUES LESS THAN ('2015-09-12 12:00:00'),
    PARTITION p18 VALUES LESS THAN ('2015-09-12 18:00:00'),
    PARTITION p24 VALUES LESS THAN ('2015-09-13 00:00:00')
)
DISTRIBUTED BY HASH(user);

CREATE TABLE source_wiki_edit
(
    event_time      DATETIME,
    channel         VARCHAR(32)      DEFAULT '',
    user            VARCHAR(128)     DEFAULT '',
    is_anonymous    TINYINT          DEFAULT '0',
    is_minor        TINYINT          DEFAULT '0',
    is_new          TINYINT          DEFAULT '0',
    is_robot        TINYINT          DEFAULT '0',
    is_unpatrolled  TINYINT          DEFAULT '0',
    delta           INT              DEFAULT '0',
    added           INT              DEFAULT '0',
    deleted         INT              DEFAULT '0'
)
DUPLICATE KEY(
    event_time,
    channel,user,
    is_anonymous,
    is_minor,
    is_new,
    is_robot,
    is_unpatrolled
)
PARTITION BY RANGE(event_time)(
    PARTITION p06 VALUES LESS THAN ('2015-09-12 06:00:00'),
    PARTITION p12 VALUES LESS THAN ('2015-09-12 12:00:00'),
    PARTITION p18 VALUES LESS THAN ('2015-09-12 18:00:00'),
    PARTITION p24 VALUES LESS THAN ('2015-09-13 00:00:00')
)
DISTRIBUTED BY HASH(user);
```

> **注意**
>
> バージョン2.5.7以降、StarRocksはテーブル作成とパーティション追加時に自動的にバケット数（BUCKETS）を設定することをサポートしています。手動でバケット数を設定する必要はありません。詳細は[バケット数の決定](../table_design/Data_distribution.md#バケット数の決定)を参照してください。

## INSERT INTO VALUES文を使用してデータをインポートする

INSERT INTO VALUES文を使用して、指定されたテーブルに直接データをインポートすることができます。このインポート方法では、複数のデータはカンマ（,）で区切られます。詳細な使用方法については、[SQLリファレンス - INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。詳細なパラメータ情報については、[INSERTのパラメータ説明](../sql-reference/sql-statements/data-manipulation/INSERT.md#パラメータ説明)を参照してください。

> **注意**
>
> INSERT INTO VALUES文によるインポート方法は、少量のデータを検証用のDEMOとしてインポートする場合にのみ適しており、大規模なテストや生産環境には適していません。大規模なデータをインポートする必要がある場合は、他のインポート方法を選択してください。

以下の例では、`insert_load_wikipedia`というLabelを使用して、データソーステーブル`source_wiki_edit`に2つのデータをインポートします。Labelはインポートジョブの識別子であり、データベース内で一意です。

```SQL
INSERT INTO source_wiki_edit
WITH LABEL insert_load_wikipedia
VALUES
    ("2015-09-12 00:00:00","#en.wikipedia","AustinFF",0,0,0,0,0,21,5,0),
    ("2015-09-12 00:00:00","#ca.wikipedia","helloSR",0,1,0,1,0,3,23,0);
```

| パラメータ   | 説明                                                         |
| ---------- | ------------------------------------------------------------ |
| table_name | データをインポートする対象のテーブル。`db_name.table_name`の形式を使用することができます。       |
| label      | インポートジョブの識別子であり、データベース内で一意です。指定されていない場合、StarRocksは自動的にジョブに対してLabelを生成します。Labelを指定することをお勧めします。そうでないと、現在のインポートジョブがネットワークエラーのために結果を返すことができない場合、そのインポート操作が成功したかどうかを知ることができません。Labelが指定されている場合は、SQLコマンド`SHOW LOAD WHERE label="label";`を使用してジョブの結果を確認することができます。 |
| values     | VALUES構文を使用して1つまたは複数のデータを挿入します。複数のデータはカンマ（,）で区切られます。 |

## INSERT INTO SELECT文を使用してデータをインポートする

INSERT INTO SELECT文を使用して、ソーステーブルのデータをターゲットテーブルにインポートすることができます。INSERT INTO SELECTは、ソーステーブルのデータをETL変換した後、StarRocksの内部テーブルにインポートします。ソーステーブルは、内部テーブルまたは外部テーブルの1つまたは複数、あるいはクラウドストレージやHDFS内のデータファイルでも構いません。ターゲットテーブルはStarRocksの内部テーブルでなければなりません。この文を実行すると、システムはSELECT文の結果をターゲットテーブルにインポートします。詳細な使用方法については、[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。詳細なパラメータ情報については、[INSERTのパラメータ](../sql-reference/sql-statements/data-manipulation/INSERT.md#パラメータ説明)を参照してください。

### INSERT INTO SELECTを使用して内部および外部テーブルのデータを内部テーブルにインポートする

> 説明
>
> 以下の例は内部テーブルのデータをインポートすることを示していますが、操作プロセスは外部テーブルのデータをインポートする場合と同じですので、外部テーブルのデータをインポートするプロセスは再度示しません。

- 以下の例では、`insert_load_wikipedia_1`というLabelを使用して、ソーステーブルのデータをターゲットテーブルにインポートします。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_1
SELECT * FROM source_wiki_edit;
```

- 以下の例では、`insert_load_wikipedia_2`というLabelを使用して、ソーステーブルのデータをターゲットテーブルの`p06`および`p12`パーティションにインポートします。ターゲットパーティションを指定しない場合は、データは全テーブルにインポートされます。ターゲットパーティションを指定した場合は、データは指定されたパーティションにのみインポートされます。


```SQL
INSERT INTO insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_2
SELECT * FROM source_wiki_edit;
```

`p06` と `p12` パーティションをクリアすると、以前に対応するパーティションに挿入されたデータはクエリできません。

```Plain
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.00 sec)

MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 次の例では、`insert_load_wikipedia_3` をラベルとして、ソーステーブルの `event_time` と `channel` 列のデータをターゲットテーブルの対応する列にインポートします。インポートされなかった列にはデフォルト値が設定されます。

```SQL
INSERT INTO insert_wiki_edit
WITH LABEL insert_load_wikipedia_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

| パラメータ        | 説明                                                         |
| ----------- | ------------------------------------------------------------ |
| table_name  | データをインポートするターゲットテーブル。`db_name.table_name` 形式も可能です。         |
| partitions  | インポートするターゲットパーティション。このパラメータは、ターゲットテーブルに存在するパーティションでなければなりません。複数のパーティション名はコンマ（`,`）で区切ります。このパラメータを指定した場合、データは指定されたパーティション内にのみインポートされます。指定しない場合は、デフォルトでターゲットテーブルのすべてのパーティションにデータがインポートされます。 |
| label       | インポートジョブの識別子で、データベース内で一意です。指定しない場合、StarRocks はジョブに自動的にラベルを生成します。ラベルを指定することをお勧めします。そうでないと、現在のインポートジョブがネットワークエラーで結果を返せない場合、インポート操作が成功したかどうかを知ることができません。ラベルを指定した場合は、SQL コマンド `SHOW LOAD WHERE label="label"` でジョブの結果を確認できます。 |
| column_name | インポートするターゲット列で、ターゲットテーブルに存在する列でなければなりません。このパラメータは、インポートするデータの列名と異なる場合がありますが、順序は一致している必要があります。ターゲット列を指定しない場合、デフォルトでターゲットテーブルのすべての列が対象になります。ソーステーブルのある列がターゲット列に存在しない場合は、デフォルト値が書き込まれます。現在の列にデフォルト値がない場合、インポートジョブは失敗します。クエリ結果の列の型がターゲット列の型と一致しない場合は、暗黙の型変換が行われます。変換できない場合は、INSERT INTO ステートメントで構文解析エラーが発生します。 |
| query       | クエリステートメントで、その結果はターゲットテーブルにインポートされます。クエリステートメントは、StarRocks がサポートする任意の SQL クエリ構文をサポートしています。 |

### INSERT INTO SELECT およびテーブル関数 FILES() を使用して外部データファイルをインポートする

v3.1 以降、StarRocks は INSERT ステートメントと [FILES()](../sql-reference/sql-functions/table-functions/files.md) テーブル関数を使用して、クラウドストレージまたは HDFS 内のファイルを直接インポートすることをサポートしています。これにより、External Catalog やファイル外部テーブルを事前に作成する必要がなくなります。さらに、FILES() は Table Schema を自動的に推測し、インポートプロセスを大幅に簡素化します。

以下の例では、AWS S3 バケット `inserttest` 内の Parquet ファイル **parquet/insert_wiki_edit_append.parquet** からデータを `insert_wiki_edit` テーブルにインポートしています：

```Plain
INSERT INTO insert_wiki_edit
    SELECT * FROM FILES(
        "path" = "s3://inserttest/parquet/insert_wiki_edit_append.parquet",
        "format" = "parquet",
        "aws.s3.access_key" = "XXXXXXXXXX",
        "aws.s3.secret_key" = "YYYYYYYYYY",
        "aws.s3.region" = "us-west-2"
);
```

## INSERT OVERWRITE VALUES ステートメントでデータを上書きインポートする

INSERT OVERWRITE VALUES ステートメントを使用して、指定されたテーブルにデータを上書きインポートすることができます。このインポート方法では、複数のデータはコンマ（,）で区切られます。使用方法の詳細については、[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。パラメータの詳細については、[INSERT パラメータ説明](../sql-reference/sql-statements/data-manipulation/INSERT.md#パラメータ説明) を参照してください。

> **注意**
>
> INSERT OVERWRITE VALUES ステートメントによるインポートは、少量のデータを検証するデモ目的にのみ適しており、大規模なテストや本番環境には適していません。大量のデータをインポートする必要がある場合は、他のインポート方法を選択してください。

ソーステーブルとターゲットテーブルをクエリして、既存のデータを確認します。

```Plain
MySQL > SELECT * FROM source_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.02 sec)
 
MySQL > SELECT * FROM insert_wiki_edit;
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user     | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| 2015-09-12 00:00:00 | #ca.wikipedia | helloSR  |            0 |        1 |      0 |        1 |              0 |     3 |    23 |       0 |
| 2015-09-12 00:00:00 | #en.wikipedia | AustinFF |            0 |        0 |      0 |        0 |              0 |    21 |     5 |       0 |
+---------------------+---------------+----------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)
```

以下の例では、`insert_load_wikipedia_ow` をラベルとして、ソーステーブル `source_wiki_edit` に2つのデータを上書きインポートしています。

```SQL
INSERT OVERWRITE source_wiki_edit
WITH LABEL insert_load_wikipedia_ow
VALUES
    ("2015-09-12 00:00:00","#cn.wikipedia","GELongstreet",0,0,0,0,0,36,36,0),
    ("2015-09-12 00:00:00","#fr.wikipedia","PereBot",0,1,0,1,0,17,17,0);
```

## INSERT OVERWRITE SELECT ステートメントでデータを上書きインポートする

INSERT OVERWRITE SELECT ステートメントを使用して、ソーステーブルのデータをターゲットテーブルに上書きインポートすることができます。INSERT OVERWRITE SELECT は、ソーステーブルのデータを ETL 変換した後、StarRocks の内部テーブルに上書きインポートします。ソーステーブルは、内部テーブルまたは外部テーブルのいずれか一つまたは複数にすることができます。ターゲットテーブルは、StarRocks の内部テーブルでなければなりません。このステートメントを実行すると、システムは SELECT ステートメントの結果でターゲットテーブルのデータを上書きします。使用方法の詳細については、[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md) を参照してください。パラメータの詳細については、[INSERT パラメータ](../sql-reference/sql-statements/data-manipulation/INSERT.md#パラメータ説明) を参照してください。

> 説明
>
> 以下の例は、内部テーブルデータのインポートのみを示しており、外部テーブルデータのインポートプロセスは同じであるため、外部テーブルデータのインポートプロセスは再度示しません。

- 以下の例では、`insert_load_wikipedia_ow_1` をラベルとして、ソーステーブルのデータをターゲットテーブルに上書きインポートしています。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_1
SELECT * FROM source_wiki_edit;
```

- 以下の例では、`insert_load_wikipedia_ow_2` をラベルとして、ソーステーブルのデータをターゲットテーブルの `p06` と `p12` パーティションに上書きインポートしています。ターゲットパーティションを指定しない場合は、全表にデータが上書きされます。指定した場合は、指定されたパーティションにのみデータが上書きされます。

```SQL
INSERT OVERWRITE insert_wiki_edit PARTITION(p06, p12)
WITH LABEL insert_load_wikipedia_ow_2
SELECT * FROM source_wiki_edit;
```

`p06` と `p12` パーティションをクリアすると、以前に上書きされた対応するパーティションのデータはクエリできません。

```Plain
MySQL > select * from insert_wiki_edit;
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
| event_time          | channel       | user         | is_anonymous | is_minor | is_new | is_robot | is_unpatrolled | delta | added | deleted |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+

| 2015-09-12 00:00:00 | #fr.wikipedia | PereBot      |            0 |        1 |      0 |        1 |              0 |    17 |    17 |       0 |
| 2015-09-12 00:00:00 | #cn.wikipedia | GELongstreet |            0 |        0 |      0 |        0 |              0 |    36 |    36 |       0 |
+---------------------+---------------+--------------+--------------+----------+--------+----------+----------------+-------+-------+---------+
2 rows in set (0.01 sec)

MySQL > TRUNCATE TABLE insert_wiki_edit PARTITION(p06, p12);
Query OK, 0 rows affected (0.01 sec)

MySQL > select * from insert_wiki_edit;
Empty set (0.00 sec)
```

- 次の例では、`insert_load_wikipedia_ow_3` を Label として使用し、ソーステーブルの `event_time` と `channel` 列のデータを、ターゲットテーブルの対応する列に上書きします。インポートされなかった列にはデフォルト値が設定されます。

```SQL
INSERT OVERWRITE insert_wiki_edit
WITH LABEL insert_load_wikipedia_ow_3 
(
    event_time, 
    channel
)
SELECT event_time, channel FROM source_wiki_edit;
```

## INSERT 文を使用して生成列にデータをインポートする

生成列（Generated Columns）は、列定義にある式に基づいて自動的に計算される特別な列です。生成列に直接データを書き込んだり、更新したりすることはできません。例えば、JSON 型のフィールドをクエリしたり、ARRAY データに対して計算を行うなど、式の計算がクエリリクエストに関係する場合に生成列が特に有用です。データインポート時に StarRocks は式を計算し、その結果を生成列に保存します。これにより、クエリ処理中に式を計算する必要がなくなり、クエリパフォーマンスが向上します。

生成列を含むテーブルにデータをインポートするには、INSERT 文を使用します。

以下の例では、`insert_generated_columns` テーブルを作成し、1行のデータを挿入します。このテーブルには2つの生成列、`avg_array` と `get_string` が含まれています。`avg_array` は `data_array` の ARRAY データの平均値を計算し、`get_string` は `data_json` から JSON パス `$.a` の文字列を抽出します。

```SQL
CREATE TABLE insert_generated_columns (
  id           INT(11)           NOT NULL    COMMENT "ID",
  data_array   ARRAY<INT(11)>    NOT NULL    COMMENT "ARRAY",
  data_json    JSON              NOT NULL    COMMENT "JSON",
  avg_array    DOUBLE            NULL 
      AS array_avg(data_array)               COMMENT "Get the average of ARRAY",
  get_string   VARCHAR(65533)    NULL 
      AS get_json_string(json_string(data_json), '$.a') COMMENT "Extract JSON string"
) ENGINE=OLAP 
PRIMARY KEY(id)
DISTRIBUTED BY HASH(id);

INSERT INTO insert_generated_columns 
VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
```

> **説明**
>
> 生成列に直接データをインポートすることはサポートされていません。

テーブルをクエリしてデータを確認します。

```Plain
mysql> SELECT * FROM insert_generated_columns;
+------+------------+------------------+-----------+------------+
| id   | data_array | data_json        | avg_array | get_string |
+------+------------+------------------+-----------+------------+
|    1 | [1,2]      | {"a": 1, "b": 2} |       1.5 | 1          |
+------+------------+------------------+-----------+------------+
1 row in set (0.02 sec)
```

## INSERT 文を使用してデータを非同期にインポートする

INSERT 文を使用して作成された同期インポートタスクは、セッションの中断やタイムアウトにより失敗する可能性があります。[SUBMIT TASK](../sql-reference/sql-statements/data-manipulation/SUBMIT_TASK.md) 文を使用して、非同期の INSERT タスクを提出することができます。この機能は StarRocks v2.5 からサポートされています。

- 次の例では、ソーステーブルのデータをターゲットテーブルに非同期でインポートします。

```SQL
SUBMIT TASK AS INSERT INTO insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータをターゲットテーブルに非同期で上書きします。

```SQL
SUBMIT TASK AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータをターゲットテーブルに非同期で上書きし、Hint を使用して Query Timeout を `100000` 秒に設定します。

```SQL
SUBMIT /*+set_var(query_timeout=100000)*/ TASK AS
INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

- 次の例では、ソーステーブルのデータをターゲットテーブルに非同期で上書きし、タスクを `async` として名付けます。

```SQL
SUBMIT TASK async
AS INSERT OVERWRITE insert_wiki_edit
SELECT * FROM source_wiki_edit;
```

Information Schema のメタデータビュー `task_runs` をクエリして、非同期 INSERT タスクの状態を確認できます。

次の例では、非同期 INSERT タスク `async` の状態を確認します。

```SQL
SELECT * FROM information_schema.task_runs WHERE task_name = 'async';
```

## インポートジョブの状態を確認する

### 結果から状態を確認する

同期 INSERT インポートジョブは、実行結果に応じて以下の2つの状態を返します：

- **成功した場合**

インポートが成功した場合、StarRocks の返り値は以下のようになります：

```Plain
Query OK, 2 rows affected, 2 warnings (0.05 sec)
{'label':'insert_load_wikipedia', 'status':'VISIBLE', 'txnId':'1006'}
```

| 返り値          | 説明                                                         |
| ------------- | ------------------------------------------------------------ |
| 影響を受ける行 | インポートされたデータの総行数です。`warnings` はフィルタリングされた行数を示します。 |
| ラベル         | ユーザーが指定したか自動生成された Label。Label はその INSERT インポートジョブの識別子で、現在のデータベース内で一意です。 |
| 状態        | インポートされたデータが表示可能かどうかを示します。VISIBLE は表示可能、COMMITTED はコミットされたが一時的に表示されないことを意味します。 |
| txnId         | その INSERT インポートに対応するトランザクション ID。                            |

- **失敗した場合**

すべてのデータがインポートできなかった場合、インポートは失敗と見なされ、StarRocks は関連するエラーと `tracking_url` を返します。`tracking_url` を使用して、エラーに関連するログ情報を確認し、問題を調査できます。

```Plain
ERROR 1064 (HY000): Insert has filtered data in strict mode, tracking_url=http://x.x.x.x:yyyy/api/_load_error_log?file=error_log_9f0a4fd0b64e11ec_906bbede076e9d08
```

### Information Schema で確認する

`information_schema` データベースの `loads` テーブルから [SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 文を使用して、INSERT INTO ジョブの結果を確認できます。この機能はバージョン 3.1 からサポートされています。

例1：`load_test` データベースのインポートジョブの実行状況を確認し、結果を作成時間 (`CREATE_TIME`) で降順に並べ替え、最大1件の結果を表示します：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test'
ORDER BY create_time DESC
LIMIT 1\G
```

例2：`load_test` データベースで Label が `insert_load_wikipedia` のインポートジョブの実行状況を確認します：

```SQL
SELECT * FROM information_schema.loads
WHERE database_name = 'load_test' and label = 'insert_load_wikipedia'\G
```

上記の例の結果は以下の通りです：

```Plain
*************************** 1. row ***************************
              JOB_ID: 21319
               LABEL: insert_load_wikipedia
       DATABASE_NAME: load_test
               STATE: FINISHED
            PROGRESS: ETL:100%; LOAD:100%
                TYPE: INSERT
            PRIORITY: NORMAL
           SCAN_ROWS: 0
       FILTERED_ROWS: 0
     UNSELECTED_ROWS: 0
           SINK_ROWS: 2
            ETL_INFO: 
           TASK_INFO: resource:N/A; timeout(s):300; max_filter_ratio:0.0
         CREATE_TIME: 2023-08-09 10:42:23
      ETL_START_TIME: 2023-08-09 10:42:23
     ETL_FINISH_TIME: 2023-08-09 10:42:23
     LOAD_START_TIME: 2023-08-09 10:42:23
    LOAD_FINISH_TIME: 2023-08-09 10:42:24
         JOB_DETAILS: {"All backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[10006]},"FileNumber":0,"FileSize":0,"InternalTableLoadBytes":175,"InternalTableLoadRows":2,"ScanBytes":0,"ScanRows":0,"TaskNumber":1,"Unfinished backends":{"5ebf11b5-365e-11ee-9e4a-7a563fb695da":[]}}
           ERROR_MSG: NULL
        TRACKING_URL: NULL
        TRACKING_SQL: NULL
REJECTED_RECORD_PATH: NULL
1 row in set (0.01 sec)
```

返り値のフィールドについての説明は、[`information_schema.loads`](../reference/information_schema/loads.md) を参照してください。

## 関連する設定項目

INSERT インポートジョブに対して、以下の設定項目を設定できます：

- **FE 設定項目**

| FE 設定項目                          | 説明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| insert_load_default_timeout_second | INSERT インポートジョブのタイムアウト時間（秒単位）。このパラメータで設定された時間内に完了しない場合、システムによってジョブはキャンセルされ、状態は CANCELLED になります。現在、このパラメータを使用して、すべての INSERT インポートジョブに対して一律にタイムアウト時間を設定することはできますが、個別のインポートジョブに対してタイムアウト時間を設定することはできません。デフォルトは 3600 秒（1 時間）です。インポートジョブが指定された時間内に完了できない場合は、このパラメータを調整してタイムアウト時間を延長できます。 |

- **セッション変数**

| セッション変数         | 説明                                                         |
| -------------------- | ------------------------------------------------------------ |

| enable_insert_strict | INSERT時のエラーデータ行の許容に関する設定です。`true` に設定すると、1行でもデータにエラーがある場合は、インポート失敗として返されます。`false` に設定すると、少なくとも1行のデータが正しくインポートされていれば、インポート成功として返され、Labelが返されます。このパラメータのデフォルトは `true` です。`SET enable_insert_strict = {true or false};` コマンドでこのパラメータを設定できます。 |
| query_timeout        | SQLコマンドのタイムアウト時間で、単位は秒です。INSERT文もSQLコマンドとして、このセッション変数の制限を受けます。`SET query_timeout = xxx;` コマンドでこのパラメータを設定できます。 |
