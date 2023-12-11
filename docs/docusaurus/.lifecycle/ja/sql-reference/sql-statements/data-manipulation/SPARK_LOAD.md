---
displayed_sidebar: "Japanese"
---

# SPARK LOAD

## Description

Sparkの読み込みは、外部のSparkリソースを介してインポートされたデータを前処理し、大量のStarRocksデータのインポートパフォーマンスを向上させ、StarRocksクラスタの計算リソースを保存します。これは、初期移行のシナリオやStarRocksへの大量のデータインポートに主に使用されます。

Sparkの読み込みは非同期のインポート方法です。ユーザーはMySQLプロトコルを通じてSparkタイプのインポートタスクを作成し、`SHOW LOAD`を使ってインポート結果を表示する必要があります。

> **注意**
>
> - StarRocksテーブルにデータをロードできるのは、それらのStarRocksテーブルにINSERT権限を持っているユーザーのみです。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md) で接続するStarRocksクラスタに権限を付与するよう指示されます。
> - Spark Loadを使用してStarRocksテーブルにデータをロードする場合、StarRocksテーブルのバケティング列はDATE、DATETIME、またはDECIMALタイプであってはなりません。

構文

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

現在インポートされているバッチのラベル。データベース内で一意です。

構文:

```sql
[database_name.]your_label
```

2.data_desc

インポートされたデータのバッチを記述するために使用されます。

構文:

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

注意

```plain text
file_path:

ファイルパスは、1つのファイルに指定することもできますし、ディレクトリ内のすべてのファイルを指定するために*ワイルドカードを使用することもできます。ワイルドカードはディレクトリではなくファイルに一致する必要があります。

hive_external_tbl:

Hive外部テーブルの名前です。
インポートされたstarrocksテーブルの列がhive外部テーブル内に存在していることが必要です。
各ロードタスクは、1つのHive外部テーブルからのみの読み込みをサポートします。
file_ pathモードと同時に使用することはできません。

PARTITION:

このパラメータが指定されている場合、指定されたパーティションのみがインポートされ、インポートされたパーティションの外のデータはフィルタリングされます。
指定されていない場合、デフォルトでテーブルのすべてのパーティションがインポートされます。

NEGATIVE:

このパラメータが指定されている場合、これは以前にインポートされたデータのバッチを "負の" データとしてロードすることと等価です。
このパラメータは、値の列が存在し、値の列の集計タイプがSUMのみの場合にのみ適用されます。

column_separator:

インポートファイル内の列の区切り記号を指定します。デフォルトは\ tです。
不可視文字の場合は、\ \ xを接頭辞として使用して16進数で区切り記号を表す必要があります。
例えば、hiveファイル\ x01のセパレータは "\ \ x01" と指定されます。

file_type:

インポートされたファイルのタイプを指定するために使用されます。現在、サポートされるファイルタイプはcsv、orc、およびparquetです。

column_list:

インポートファイル内の列とテーブル内の列との対応を指定するために使用されます。
インポートファイル内の列をスキップする必要がある場合は、テーブルに存在しない列名を指定してください。

構文:
(col_name1, col_name2, ...)

SET:

このパラメータを指定すると、ソースファイルの列を関数に従って変換し、変換された結果をテーブルにインポートすることができます。構文はcolumn_name = expressionです。
Spark SQLのビルドイン関数のみがサポートされています。https://spark.apache.org/docs/2.4.6/api/sql/index.html を参照してください。
理解を助けるためにいくつかの例を挙げます。
例1: テーブルには「c1、c2、c3」という3つの列があり、ソースファイルの最初の2つの列が（c1、c2）に対応し、最後の2つの列の合計がC3に対応しているとします。
その場合は、列（c1、c2、tmp_c3、tmp_c4）を指定してset(c3 = tmp_c3 + tmp_c4)が指定される必要があります。
例2: テーブルには「year、month、day」という3つの列があり、ソースファイルには「2018-06-01 01:02:03」の形式で1つの時刻列だけがあります。
この場合は、カラム(tmp_time) set(year = year(tmp_time), month = month(tmp_time), day = day(tmp_time)) を指定してインポートを完了します。

WHERE:

変換されたデータをフィルタリングし、where条件を満たすデータのみをインポートできます。WHEREステートメントではテーブル内の列名のみを参照できます
```

3.resource_name

使用されるSparkリソースの名前は、`SHOW RESOURCES`コマンドを通じて表示できます。

4.resource_properties

SparkやHDFSの設定を変更するなどの一時的な必要がある場合に、ここでパラメータを設定することができます。これはこの特定のSparkローディングジョブにのみ影響し、StarRocksクラスタ内の既存の構成に影響を与えることはありません。

5.opt_properties

いくつかの特別なパラメータを指定するために使用されます。

構文:

```sql
[PROPERTIES ("key" = "value", ...)]
```

以下のパラメータを指定することができます:
timeout: インポート操作のタイムアウトを指定します。デフォルトのタイムアウトは4時間です。単位は秒です。
max_filter_ratio:（非標準のデータなどの理由で）フィルタリングできる最大のデータ割合を指定します。デフォルトはゼロ許容です。
strict mode: データを厳密に制限するかどうかを指定します。デフォルトはfalseです。
timezone: strftime / alignment_timestamp / from_unixtimeなど、タイムゾーンの影響を受ける関数のタイムゾーンを指定します。詳細については[タイムゾーン]ドキュメントを参照してください。指定されていない場合は、「Asia / Shanghai」タイムゾーンが使用されます。

6.データのインポート形式の例

int (TINYINT/SMALLINT/INT/BIGINT/LARGEINT): 1, 1000, 1234
float (FLOAT/DOUBLE/DECIMAL): 1.1, 0.23, .356
date (DATE/DATETIME) :2017-10-03, 2017-06-13 12:34:03.
（注意：他の日付形式については、インポートコマンドで変換するためにstrftimeやtime_format関数を使用できます）文字列クラス (CHAR/VARCHAR): "私は学生です"、"a"

NULL値: \ N

## 例

1. HDFSからデータのバッチをインポートし、タイムアウト時間とフィルタリング比率を指定します。Sparkの名前をmy_ spark resourcesとして使用します。

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

    ここで、hdfs_hostはnamenodeのホストであり、hdfs_portはfs.defaultfsポート（デフォルトは9000）です。

2. HDFSから "負の" データのバッチをインポートし、セパレータをカンマに指定し、ディレクトリ内のすべてのファイルを指定するためにワイルドカード*を使用し、sparkリソースの一時的なパラメータを指定します。

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

3. HDFSからデータのバッチをインポートし、パーティションを指定し、インポートされたファイルの列に変換を行います。以下のようにします:

    ```plain text
    テーブル構造は次のとおりです:
    k1 varchar(20)
    k2 int
    
    データファイルにはデータ行が1行だけ含まれていると仮定します:
    
    Adele,1,1
    
    データファイル内の各列は、インポートステートメントで指定された各列に対応します:
    k1,tmp_k2,tmp_k3
    
    変換は以下のようになります:
    
    1. k1: 変換なし
    2. k2: tmp_ k2とtmp_k3の合計
    
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

4. ファイルパスからパーティションフィールドを抽出
```sql
ファイルパス内のパーティションされたフィールドが必要に応じて、テーブルで定義されたフィールドタイプに従って解決され、Sparkのパーティション検出の機能と類似した動作になります。

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

`hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing` のディレクトリには、以下のファイルが含まれています:

`[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

ファイルパス内のcityおよびutc_dateフィールドが抽出されます。

5. インポートするデータをフィルタリングします。k1の値が10より大きい列のみがインポートされます。

```sql
LOAD LABEL example_db.label10
(
DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
INTO TABLE `my_table`
WHERE k1 > 10
)
WITH RESOURCE 'my_spark';
```

6. Hive外部テーブルからインポートし、ソーステーブルのuuid列をグローバル辞書を利用してビットマップタイプに変換します。

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