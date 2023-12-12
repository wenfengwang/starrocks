---
displayed_sidebar: "Japanese"
---

# SPARK LOAD

## 説明

Spark loadは、外部のsparkリソースを介してインポートされたデータを前処理し、大量のStarRocksデータのインポートパフォーマンスを向上させ、StarRocksクラスタの計算リソースを節約します。主に初期移行および大量のデータをStarRocksにインポートするシナリオで使用されます。

Spark loadは非同期のインポート方法です。ユーザーはMySQLプロトコルを介してSparkタイプのインポートタスクを作成し、`SHOW LOAD`を通じてインポート結果を表示する必要があります。

> **注意**
>
> - スターロックステーブルにデータをインポートすることができるのは、それらのスターロックステーブルにINSERT権限を持つユーザーのみです。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)で指示された手順に従って、スターロックスクラスタに接続するユーザーにINSERT権限を付与する必要があります。
> - Spark Loadを使用してStarRocksテーブルにデータをロードする場合、StarRocksテーブルのバケット列はDATE、DATETIME、またはDECIMALタイプであってはなりません。

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

1. load_label

現在インポートされたバッチのラベル。データベース内で一意です。

構文:

```sql
[database_name.]your_label
```

2. data_desc

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

hive外部テーブル名。
インポートされたstarrocksテーブルの列がhive外部テーブルに存在することが必要です。
各ロードタスクは1つのHive外部テーブルからのみロードをサポートします。
同時にfile_pathモードとは使用できません。

PARTITION:

このパラメータが指定されている場合、指定されたパーティションのみがインポートされ、インポートされたパーティションの外のデータはフィルタされます。
指定されていない場合、テーブルのすべてのパーティションがデフォルトでインポートされます。

NEGATIVE:

このパラメータが指定されている場合、これは以前にインポートされたデータと同じバッチの"ネガティブ"データのロードと同等です。値の列が存在し、値の列の集計タイプがSUMの場合にのみ適用されます。

column_separator:

インポートファイル内の列区切り記号を指定します。デフォルトは \ t です。
不可視文字の場合は \ \ xを接頭辞に付けて16進数で区切り記号を表す必要があります。
例えば、hiveファイル \ x01 の区切り記号は "\ \ x01" と指定します。

file_type:

インポートファイルのタイプを指定するために使用されます。現在、サポートされているファイルタイプはcsv、orc、およびparquetです。

column_list:

インポートファイル内の列とテーブル内の列との対応関係を指定するために使用されます。
インポートファイル内の列をスキップする場合は、テーブルに存在しない列名として列を指定します。

構文:
(col_name1, col_name2, ...)

SET:

このパラメータを指定する場合、ソースファイルの列を関数に従って変換し、変換された結果をテーブルにインポートできます。構文はcolumn_name = expressionです。
Spark SQLビルトイン関数のみがサポートされます。 詳細は https://spark.apache.org/docs/2.4.6/api/sql/index.html を参照してください。
理解を助けるためにいくつかの例を示します。
例1: テーブルには3つの列 "c1、c2、c3" があり、ソースファイルの最初の2つの列が (c1、c2) に対応し、最後の2つの列の合計がC3に対応している場合、列 (c1、c2、tmp_c3、tmp_c4) を指定した(set (c3 = tmp_c3 + tmp_c4)) が必要です。
例2: テーブルには3つの列 "年、月、日" があり、ソースファイルには "2018-06-01 01:02:03" 形式の時間列しかない場合。
その場合、(tmp_time)に対して (year = year(tmp_time), month = month(tmp_time), day = day(tmp_time)) を指定してインポートを完了させることができます。

WHERE:

変換されたデータをフィルタリングし、WHERE条件を満たすデータのみがインポートできます。 WHEREステートメントではテーブル内の列名のみを参照できます
```

3. resource_name

使用されるsparkリソースの名前は、`SHOW RESOURCES`コマンドを介して確認できます。

4. resource_properties

SparkやHDFSの設定を変更するなどの一時的な必要性がある場合、ここでパラメータを設定できます。これにより、この特定のsparkロードジョブにのみ効果があり、StarRocksクラスタの既存の設定に影響を与えません。

5. opt_properties

いくつかの特別なパラメータを指定するために使用されます。

構文:

```sql
[PROPERTIES ("key"="value", ...)]
```

以下のパラメータを指定できます:
timeout:         インポート操作のタイムアウトを指定します。デフォルトのタイムアウトは4時間です。秒単位で指定します。
max_filter_ratio:フィルタリングできる最大許容データの割合を指定します（非標準データなどによる）。デフォルトはゼロ許容です。
strict mode:     データを厳密に制限するかどうかを指定します。デフォルトはfalseです。
timezone:         時間帯が影響を受けるいくつかの関数のタイムゾーンを指定します（strftime / alignment_timestamp/from_unixtimeなど）。詳細については、[time zone] ドキュメントを参照してください。指定されていない場合、 "Asia / Shanghai" タイムゾーンが使用されます。

6. インポートデータのフォーマット例

int (TINYINT/SMALLINT/INT/BIGINT/LARGEINT): 1, 1000, 1234
float (FLOAT/DOUBLE/DECIMAL): 1.1, 0.23, .356
date (DATE/DATETIME) :2017-10-03, 2017-06-13 12:34:03.
（注: 他の日付形式では、Importコマンドでstrftimeまたはtime_format関数を使用して変換できます）
文字列クラス（CHAR/VARCHAR）: "I am a student", "a"

NULL値: \ N

## 例

1. HDFSからバッチのデータをインポートし、タイムアウト時間とフィルタリング比率を指定します。sparkのリソース名を `my_spark` として使用します。

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

    ここで、hdfs_hostはnamenodeのホストであり、hdfs_portはfs.defaultfsポート（デフォルト9000）です。

2. HDFSから "ネガティブ" データのバッチをインポートし、セパレータをカンマに指定し、ワイルドカードを使用してディレクトリ内のすべてのファイルを指定し、sparkリソースの一時的なパラメータを指定します。

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

3. HDFSからバッチのデータをインポートし、パーティションを指定し、インポートされたファイルの列に変換を行います。以下のようになります:

    ```plain text
    テーブル構造:
    k1 varchar(20)
    k2 int
    
    データファイルには1行のデータしかないとします:
    
    Adele,1,1
    
    データファイルの各列が、インポートステートメントで指定された各列に対応しているとします:
    k1,tmp_k2,tmp_k3
    
    変換は以下のようになります:
    
    1. k1: 変換なし
    2. k2: tmp_k2とtmp_k3の合計

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
    必要であれば、ファイルパス内の分割されたフィールドは、SparkのPartition Discoveryの機能に類似して、テーブルで定義されたフィールドタイプに従って解決されます。

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

    `hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing`ディレクトリには、以下のファイルが含まれています:

    `[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

    ファイルパス内のcityとutc_dateフィールドが抽出されます。

5. インポートするデータをフィルタリングします。k1の値が10より大きい列のみがインポートできます。

    ```sql
    LOAD LABEL example_db.label10
    (
    DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
    INTO TABLE `my_table`
    WHERE k1 > 10
    )
    WITH RESOURCE 'my_spark';
    ```

6. hive外部テーブルからのインポートし、ソーステーブル内のuuid列をグローバル辞書を介してbitmapタイプに変換します。

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
```