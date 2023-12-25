---
displayed_sidebar: English
---

# SPARK LOAD

## 説明

Spark Loadは、外部Sparkリソースを通じてインポートされたデータを前処理し、大量のStarRocksデータのインポートパフォーマンスを向上させ、StarRocksクラスターの計算リソースを節約します。主に初期移行やStarRocksへの大量データインポートのシナリオで使用されます。

Spark Loadは非同期インポート方法です。ユーザーはMySQLプロトコルを通じてSparkタイプのインポートタスクを作成し、`SHOW LOAD`を通じてインポート結果を確認する必要があります。

> **注意**
>
> - StarRocksテーブルにデータをロードできるのは、そのStarRocksテーブルにINSERT権限を持つユーザーのみです。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)に記載されている手順に従って、StarRocksクラスタに接続するために使用するユーザーにINSERT権限を付与してください。
> - Spark Loadを使用してStarRocksテーブルにデータをロードする場合、StarRocksテーブルのバケット列はDATE、DATETIME、またはDECIMAL型であってはなりません。

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

構文：

```sql
[database_name.]your_label
```

2.data_desc

インポートされるデータバッチを記述するために使用されます。

構文：

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

注記

```plain text
file_path:

ファイルパスは1つのファイルに指定することも、*ワイルドカードを使用してディレクトリ内のすべてのファイルを指定することもできます。ワイルドカードはファイルに一致する必要があり、ディレクトリには一致しません。

hive_external_tbl:

Hive外部テーブル名。
インポートされるStarRocksテーブルのカラムはHive外部テーブルに存在する必要があります。
各ロードタスクは、1つのHive外部テーブルからのロードのみをサポートします。
file_pathモードと同時に使用することはできません。

PARTITION:

このパラメータを指定すると、指定されたパーティションのみがインポートされ、インポートされたパーティション外のデータはフィルタリングされます。
指定されていない場合、デフォルトでテーブルのすべてのパーティションがインポートされます。

NEGATIVE:

このパラメータを指定すると、「負の」データのバッチをロードすることに相当します。以前にインポートされた同じバッチのデータを相殺するために使用されます。
このパラメータは、値カラムが存在し、その値カラムの集約タイプがSUMのみの場合に適用されます。

column_separator:

インポートファイル内のカラムセパレータを指定します。デフォルトは\tです。
不可視文字の場合は、\xとしてプレフィックスを付け、16進数でセパレータを表現する必要があります。
例えば、Hiveファイルのセパレータ\x01は"\\x01"として指定されます。

file_type:

インポートされるファイルのタイプを指定するために使用されます。現在サポートされているファイルタイプはcsv、orc、parquetです。

column_list:

インポートファイルのカラムとテーブルのカラムとの対応を指定するために使用されます。
インポートファイルのカラムをスキップする必要がある場合は、テーブルに存在しないカラム名として指定します。

構文：
(col_name1, col_name2, ...)

SET:

このパラメータを指定すると、ソースファイルのカラムを関数に従って変換し、変換された結果をテーブルにインポートすることができます。構文はcolumn_name = expressionです。
Spark SQLのビルトイン関数のみがサポートされています。詳細はhttps://spark.apache.org/docs/2.4.6/api/sql/index.htmlを参照してください。
理解を助けるためのいくつかの例を示します。
例1：テーブルには「c1、c2、c3」という3つのカラムがあり、ソースファイルの最初の2つのカラムは(c1、c2)に対応し、最後の2つのカラムの合計はc3に対応します。その場合、columns (c1、c2、tmp_c3、tmp_c4) set (c3 = tmp_c3 + tmp_c4)を指定する必要があります。
例2：テーブルには「year、month、day」という3つのカラムがあり、ソースファイルには"2018-06-01 01:02:03"の形式で1つの時間カラムのみがあります。
その場合、columns (tmp_time) set (year = year(tmp_time), month = month(tmp_time), day = day(tmp_time))を指定してインポートを完了することができます。

WHERE:

変換されたデータをフィルタリングし、where条件を満たすデータのみがインポートされます。WHERE文ではテーブルのカラム名のみを参照できます。
```

3.resource_name

使用されるSparkリソースの名前は、`SHOW RESOURCES`コマンドを通じて確認できます。

4.resource_properties

一時的なニーズがある場合、例えばSparkやHDFSの設定を変更する場合、ここでパラメータを設定できますが、これはこの特定のSparkロードジョブにのみ効果があり、StarRocksクラスタの既存の設定には影響しません。

5.opt_properties

特別なパラメータを指定するために使用されます。

構文：

```sql
[PROPERTIES ("key"="value", ...)]
```

以下のパラメータを指定できます：
timeout: インポート操作のタイムアウトを指定します。デフォルトのタイムアウトは4時間です。秒単位で指定します。
max_filter_ratio: フィルタリングできるデータの最大許容比率を指定します（非標準データなどの理由による）。デフォルトはゼロトレランスです。
strict_mode: データを厳密に制限するかどうかを指定します。デフォルトはfalseです。
timezone: strftime/alignment_timestamp/from_unixtimeなど、タイムゾーンの影響を受ける関数のタイムゾーンを指定します。詳細は[タイムゾーン]ドキュメントを参照してください。指定されていない場合は、「Asia/Shanghai」タイムゾーンが使用されます。

6.インポートデータフォーマットの例

int (TINYINT/SMALLINT/INT/BIGINT/LARGEINT): 1, 1000, 1234
float (FLOAT/DOUBLE/DECIMAL): 1.1, 0.23, .356
date (DATE/DATETIME): 2017-10-03, 2017-06-13 12:34:03.
（注：他の日付形式については、インポートコマンドでstrftimeまたはtime_format関数を使用して変換できます）文字列型 (CHAR/VARCHAR): "I am a student", "a"

NULL値: \N

## 例

1. HDFSからデータバッチをインポートし、タイムアウト時間とフィルタリング比率を指定します。Sparkリソースには'my_spark'という名前を使用します。

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

    ここでhdfs_hostはnamenodeのホスト、hdfs_portはfs.defaultfsポート（デフォルトは9000）です。

2. HDFSから「負の」データバッチをインポートし、区切り文字をコンマに指定し、ワイルドカード*を使用してディレクトリ内のすべてのファイルを指定し、Sparkリソースの一時的なパラメータを指定します。

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

3. HDFSからデータバッチをインポートし、パーティションを指定し、インポートファイルの列に対して変換を行います。以下のようになります：

    ```plain text
    テーブル構造は以下の通りです：
    k1 varchar(20)
    k2 int
    
    データファイルには1行のデータがあると仮定します：
    
    Adele,1,1
    
    データファイルの各列は、インポートステートメントで指定された各列に対応します：
    k1,tmp_k2,tmp_k3
    
    変換は以下の通りです：
    
    1. k1：変換なし
    2. k2：tmp_k2とtmp_k3の合計
    
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

4. ファイルパス内のパーティションフィールドを抽出します

    必要に応じて、ファイルパス内のパーティションフィールドは、テーブルで定義されたフィールドタイプに従って解決されます。これはSparkのPartition Discovery機能に似ています。

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

    `hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing`のディレクトリには以下のファイルが含まれています：

    `[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

    ファイルパスのcityフィールドとutc_dateフィールドが抽出されます。

5. インポートするデータをフィルタリングします。k1の値が10より大きい列のみをインポートできます。

    ```sql
    LOAD LABEL example_db.label10
    (
    DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
    INTO TABLE `my_table`
    WHERE k1 > 10
    )
    WITH RESOURCE 'my_spark';
    ```

6. Hive外部テーブルからインポートし、ソーステーブルのuuid列をグローバルディクショナリを使用してビットマップ型に変換します。

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
