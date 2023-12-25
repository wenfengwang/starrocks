---
displayed_sidebar: Chinese
---

# SPARK LOAD

## 機能

Spark Loadは外部のSparkリソースを利用して、データの事前処理を行い、StarRocksの大量データのインポート性能を向上させ、StarRocksクラスターの計算リソースを節約します。主に初回の移行や、大量データをStarRocksにインポートするシナリオで使用されます。

Spark Loadは非同期インポート方式で、ユーザーはMySQLプロトコルを通じてSparkタイプのインポートタスクを作成し、`SHOW LOAD`でインポート結果を確認する必要があります。

> **注意**
>
> - Spark Load操作には対象テーブルのINSERT権限が必要です。ユーザーアカウントにINSERT権限がない場合は、[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。
> - Spark Loadを使用してデータをStarRocksテーブルにインポートする際、テーブルのバケット列のデータタイプがDATE、DATETIME、またはDECIMALに対応していないことに注意してください。

## 文法

```sql
LOAD LABEL load_label
(
data_desc1[, data_desc2, ...]
)
WITH RESOURCE resource_name
[resource_properties]
[opt_properties]
```

1.load_label

現在のインポートバッチのラベル。データベース内で一意です。

文法：

```sql
[database_name.]your_label
```

2.data_desc

一連のインポートデータを記述するために使用されます。

文法：

```sql
DATA INFILE
(
"file_path1"[, file_path2, ...]
)
[NEGATIVE]
INTO TABLE `table_name`
[PARTITION (p1, p2)]
[COLUMNS TERMINATED BY "column_separator"]
[FORMAT AS "file_type"]
[(column_list)]
[COLUMNS FROM PATH AS (col2, ...)]
[SET (k1 = func(k2))]
[WHERE predicate]

DATA FROM TABLE hive_external_tbl
[NEGATIVE]
INTO TABLE tbl_name
[PARTITION (p1, p2)]
[SET (k1=f1(xx), k2=f2(xx))]
[WHERE predicate]
```

説明：

```plain text
file_path:

ファイルパスは、単一のファイルを指定することも、*ワイルドカードを使用して特定のディレクトリ内のすべてのファイルを指定することもできます。ワイルドカードはファイルに一致しなければならず、ディレクトリには一致しません。

hive_external_tbl:

Hiveの外部テーブル名。
インポートされるStarRocksテーブルの列は、Hiveの外部テーブルに存在する必要があります。
各インポートタスクは、一つのHive外部テーブルからのインポートのみをサポートします。
file_path方式と同時に使用することはできません。

PARTITION:

このパラメータを指定すると、指定されたパーティションのみがインポートされ、パーティション外のデータはフィルタリングされます。
指定しない場合は、デフォルトでテーブルのすべてのパーティションがインポートされます。

NEGATIVE：

このパラメータを指定すると、「負」のデータをインポートすることになり、以前にインポートされた同じバッチのデータを相殺するために使用されます。
このパラメータは、value列が存在し、そのvalue列の集約タイプがSUMのみである場合にのみ適用されます。

column_separator：

インポートファイル内の列の区切り文字を指定するために使用されます。デフォルトは\tです。
不可視文字の場合は、\\xをプレフィックスとして追加し、16進数で区切り文字を表現する必要があります。
例えば、Hiveファイルの区切り文字\x01は"\\x01"として指定します。

file_type：

インポートファイルのタイプを指定するために使用されます。現在、csv、orc、parquetのみをサポートしています。

column_list：

インポートファイル内の列とテーブル内の列の対応関係を指定するために使用されます。
インポートファイル内の特定の列をスキップする必要がある場合は、その列をテーブル内に存在しない列名として指定します。

文法：
(col_name1, col_name2, ...)

SET:

このパラメータを指定すると、ソースファイルの特定の列を関数で変換し、その結果をテーブルにインポートすることができます。文法は`column_name` = expressionです。
Spark SQLの組み込み関数のみをサポートしており、詳細は https://spark.apache.org/docs/2.4.6/api/sql/index.html を参照してください。
いくつかの例を挙げて理解を助けます。
例1: テーブルには「c1, c2, c3」という3つの列があり、ソースファイルの最初の2列は(c1,c2)に対応し、次の2列の合計がc3に対応します。その場合、columns (c1,c2,tmp_c3,tmp_c4) SET (c3=tmp_c3+tmp_c4)と指定する必要があります。
例2: テーブルには「year, month, day」という3つの列があり、ソースファイルには「2018-06-01 01:02:03」という形式の1つの日付列のみがあります。
その場合、columns(tmp_time) set (year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))を指定してインポートを完了することができます。

WHERE:

変換後のデータに対してフィルタリングを行い、WHERE条件に合致するデータのみがインポートされます。WHERE文では、テーブル内の列名のみを参照できます。
```

3.resource_name

使用するSparkリソースの名前で、`SHOW RESOURCES`コマンドで確認できます。

4.resource_properties

[Sparkリソースの設定](../data-definition/CREATE_RESOURCE.md#spark-リソース)。ユーザーが一時的なニーズがある場合、例えばタスクに使用するリソースを増やすためにSparkとHDFSの設定を変更する場合、ここで設定できます。設定はこのタスクにのみ有効で、StarRocksクラスター内の既存の設定には影響しません。

5.opt_properties

特別なパラメータを指定するために使用されます。

文法：

```sql
[PROPERTIES ("key"="value", ...)]
```

以下のパラメータを指定できます：

```plain text
timeout：         インポート操作のタイムアウト時間を指定します。デフォルトのタイムアウトは4時間です。単位は秒です。
max_filter_ratio：フィルタリング可能な（データが不規則などの理由で）データの最大比率を指定します。デフォルトはゼロトレランスです。
strict_mode：     データに対する厳格な制限を行うかどうかを指定します。デフォルトはfalseです。
timezone:         strftime/alignment_timestamp/from_unixtimeなど、タイムゾーンに影響される関数のタイムゾーンを指定します。詳細は[タイムゾーン](../../../administration/timezone.md)のドキュメントを参照してください。指定しない場合は、"Asia/Shanghai"のタイムゾーンが使用されます。
```

6.インポートデータの形式例

```plain text
1.整数型（TINYINT/SMALLINT/INT/BIGINT/LARGEINT）：1, 1000, 1234。

2.浮動小数点型（FLOAT/DOUBLE/DECIMAL）：1.1, 0.23, .356。

3.日付型（DATE/DATETIME）：2017-10-03, 2017-06-13 12:34:03。（注：他の日付形式の場合は、インポートコマンドでstrftimeまたはtime_format関数を使用して変換できます）

4.文字列型（CHAR/VARCHAR）："I am a student", "a"。
NULL値：\N
```

## 例

### HDFSからデータをインポート

HDFSから一連のデータをインポートし、タイムアウト時間とフィルター比率を指定します。my_sparkという名前のSparkリソースを使用します。

```sql
LOAD LABEL example_db.label1
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
INTO TABLE `my_table`
)
WITH RESOURCE 'my_spark'
PROPERTIES
(
    "timeout" = "3600",
    "max_filter_ratio" = "0.1"
);
```

ここで、hdfs_hostはnamenodeのホスト、hdfs_portはfs.defaultFSのポート（デフォルトは9000）です。

### HDFSから「負」のデータをインポート

HDFSから一連の「負」のデータをインポートし、カンマを区切り文字として指定し、ワイルドカード*を使用してディレクトリ内のすべてのファイルを指定し、Sparkリソースの一時的なパラメータを指定します。「負」のデータの詳細な意味は上記の文法パラメータの説明部分を参照してください。

```sql
LOAD LABEL example_db.label3
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/*")
NEGATIVE
INTO TABLE `my_table`
COLUMNS TERMINATED BY ","
)
WITH RESOURCE 'my_spark'
(
    "spark.executor.memory" = "3g",
    "broker.username" = "hdfs_user",
    "broker.password" = "hdfs_passwd"
);
```

### HDFSから指定されたパーティションにデータをインポートし、列を変換

HDFSから一連のデータをインポートし、指定されたパーティションを指定し、インポートファイルの列を変換します。以下のように：

```plain text
テーブル構造：
k1 varchar(20)
k2 int

データファイルには1行のデータのみがあります：

Adele,1,1

データファイル内の各列は、インポートステートメントで指定された各列に対応します：
k1,tmp_k2,tmp_k3

変換は以下の通りです：

1. k1: 変換なし
2. k2：tmp_k2とtmp_k3のデータの合計です

LOAD LABEL example_db.label6
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
INTO TABLE `my_table`
PARTITION (p1, p2)
COLUMNS TERMINATED BY ","
(k1, tmp_k2, tmp_k3)
SET (
k2 = tmp_k2 + tmp_k3
)
)
WITH RESOURCE 'my_spark';
```

### ファイルパスからパーティションフィールドを抽出

必要に応じて、テーブルで定義されたフィールドタイプに基づいてファイルパスからパーティションフィールドを解析します。これはSparkのPartition Discovery機能に似ています。

```sql
LOAD LABEL example_db.label10
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/*/*")
INTO TABLE `my_table`
(k1, k2, k3)
COLUMNS FROM PATH AS (city, utc_date)
SET (uniq_id = md5sum(k1, city))
)
WITH RESOURCE 'my_spark';
```

`hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing`ディレクトリには、以下のファイルが含まれています：

`[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

これにより、ファイルパスから`city`と`utc_date`フィールドが抽出されます。

### インポートデータをフィルタリング

インポートされるデータをフィルタリングし、k1の値が10より大きい列のみがインポートされます。

```sql
LOAD LABEL example_db.label10
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
INTO TABLE `my_table`
WHERE k1 > 10
)
WITH RESOURCE 'my_spark';
```

### Hive外部テーブルからインポートしてグローバル辞書を構築

Hiveの外部テーブルからインポートし、ソーステーブルのuuid列をグローバル辞書を使用してbitmapタイプに変換します。

```sql
LOAD LABEL db1.label1
(
DATA FROM TABLE hive_t1
INTO TABLE tbl1
SET
(
uuid=bitmap_dict(uuid)
)
)
WITH RESOURCE 'my_spark';
```
