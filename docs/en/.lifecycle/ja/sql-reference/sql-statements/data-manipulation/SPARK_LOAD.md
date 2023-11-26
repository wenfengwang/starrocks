---
displayed_sidebar: "Japanese"
---

# SPARK LOAD（スパークロード）

## 説明

Spark Loadは、外部のSparkリソースを介してインポートされたデータを前処理し、大量のStarRocksデータのインポートパフォーマンスを向上させ、StarRocksクラスタの計算リソースを節約します。主に初期移行や大量のデータをStarRocksにインポートするシナリオで使用されます。

Spark Loadは非同期のインポート方法です。ユーザーはMySQLプロトコルを介してSparkタイプのインポートタスクを作成し、`SHOW LOAD`を使用してインポート結果を表示する必要があります。

> **注意**
>
> - StarRocksテーブルにデータをロードするには、そのStarRocksテーブルにINSERT権限を持つユーザーとしてのみデータをロードできます。INSERT権限がない場合は、[GRANT](../account-management/GRANT.md)の指示に従って、StarRocksクラスタに接続するために使用するユーザーにINSERT権限を付与してください。
> - Spark Loadを使用してデータをStarRocksテーブルにロードする場合、StarRocksテーブルのバケット化列はDATE、DATETIME、またはDECIMAL型ではない必要があります。

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

現在のインポートバッチのラベル。データベース内で一意です。

構文:

```sql
[database_name.]your_label
```

2. data_desc

インポートされたデータのバッチを説明するために使用されます。

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

ファイルパスは、1つのファイルを指定するか、ワイルドカード*を使用してディレクトリ内のすべてのファイルを指定することができます。ワイルドカードはディレクトリではなくファイルに一致する必要があります。

hive_external_tbl:

Hive外部テーブル名。
インポートされるStarRocksテーブルの列は、Hive外部テーブルに存在する必要があります。
各ロードタスクは、1つのHive外部テーブルからのみのロードをサポートします。
file_pathモードと同時に使用することはできません。

PARTITION:

このパラメータが指定されている場合、指定されたパーティションのみがインポートされ、インポートされたパーティションの外のデータはフィルタリングされます。
指定されていない場合、デフォルトでテーブルのすべてのパーティションがインポートされます。

NEGATIVE:

このパラメータが指定されている場合、同じバッチの以前にインポートされたデータをオフセットするための「負の」データバッチのロードと等価です。
このパラメータは、値列が存在し、値列の集計タイプがSUMの場合にのみ適用されます。

column_separator:

インポートファイルの列セパレータを指定します。デフォルトは\ tです。
見えない文字の場合は、\ \ xを接頭辞に付けてセパレータを表すために16進数を使用する必要があります。
たとえば、hiveファイルのセパレータ\ x01は"\ \ x01"と指定されます。

file_type:

インポートファイルのタイプを指定するために使用されます。現在、サポートされているファイルタイプはcsv、orc、およびparquetです。

column_list:

インポートファイルの列とテーブルの列との対応関係を指定するために使用されます。
インポートファイルで列をスキップする必要がある場合は、テーブルに存在しない列名として列を指定します。

構文:
(col_name1, col_name2, ...)

SET:

このパラメータを指定すると、ソースファイルの列を関数に従って変換し、変換された結果をテーブルにインポートすることができます。構文はcolumn_name = expressionです。
Spark SQLのビルトイン関数のみサポートされています。詳細については、https://spark.apache.org/docs/2.4.6/api/sql/index.htmlを参照してください。
理解を助けるためにいくつかの例を示します。
例1：テーブルに「c1、c2、c3」という3つの列があり、ソースファイルの最初の2つの列が（c1、c2）に対応し、最後の2つの列の合計がC3に対応する場合、列（c1、c2、tmp_c3、tmp_c4）set（c3 = tmp_c3 + tmp_c4）を指定する必要があります。
例2：テーブルに「year、month、day」という3つの列があり、ソースファイルには「2018-06-01 01:02:03」という形式の1つの時間列しかありません。
その場合、列（tmp_time）set（year = year（tmp_time）、month = month（tmp_time）、day = day（tmp_time））を指定してインポートを完了できます。

WHERE:

変換されたデータをフィルタリングし、WHERE条件を満たすデータのみをインポートできます。WHEREステートメントではテーブルの列名のみを参照できます。
```

3. resource_name

使用するSparkリソースの名前は、`SHOW RESOURCES`コマンドで表示できます。

4. resource_properties

SparkやHDFSの設定を変更するなど、一時的な必要性がある場合は、ここでパラメータを設定できます。これは、この特定のSparkロードジョブにのみ効果があり、StarRocksクラスタの既存の設定に影響を与えません。

5. opt_properties

いくつかの特殊なパラメータを指定するために使用されます。

構文:

```sql
[PROPERTIES ("key"="value", ...)]
```

以下のパラメータを指定できます:
timeout:         インポート操作のタイムアウトを指定します。デフォルトのタイムアウトは4時間です。秒単位で指定します。
max_filter_ratio:フィルタリングできる最大のデータ比率を指定します（非標準のデータなどの理由で）。デフォルトはゼロ許容です。
strict mode:     データを厳密に制限するかどうかを指定します。デフォルトはfalseです。
timezone:         タイムゾーンに影響を受ける一部の関数（strftime / alignment_timestamp/from_unixtimeなど）のタイムゾーンを指定します。詳細については、[time zone]ドキュメントを参照してください。指定されていない場合、"Asia / Shanghai"タイムゾーンが使用されます。

6. インポートデータの形式の例

int（TINYINT/SMALLINT/INT/BIGINT/LARGEINT）: 1, 1000, 1234
float（FLOAT/DOUBLE/DECIMAL）: 1.1, 0.23, .356
date（DATE/DATETIME）: 2017-10-03, 2017-06-13 12:34:03.
（注：他の日付形式については、Importコマンドでstrftimeまたはtime_format関数を使用して変換できます）stringクラス（CHAR/VARCHAR）: "I am a student", "a"

NULL値: \ N

## 例

1. HDFSからデータのバッチをインポートし、タイムアウト時間とフィルタリング比率を指定します。スパークの名前をmy_sparkリソースとして使用します。

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

    ここで、hdfs_hostはNamenodeのホストであり、hdfs_portはfs.defaultfsポート（デフォルトは9000）です。

2. HDFSから「負の」データのバッチをインポートし、セパレータをカンマとして指定し、ワイルドカード*を使用してディレクトリ内のすべてのファイルを指定し、スパークリソースの一時パラメータを指定します。

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

3. HDFSからデータのバッチをインポートし、パーティションを指定し、インポートされたファイルの列にいくつかの変換を行います。以下のようになります。

    ```plain text
    テーブル構造は次のとおりです：
    k1 varchar(20)
    k2 int
    
    データファイルにはデータが1行しかないと仮定します：
    
    Adele,1,1
    
    データファイルの各列は、インポートステートメントで指定された各列に対応します：
    k1,tmp_k2,tmp_k3
    
    変換は次のようになります：
    
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

4. ファイルパスからパーティションフィールドを抽出します

    必要に応じて、ファイルパスのパーティションフィールドは、テーブルで定義されたフィールドタイプに従って解決されます。これは、SparkのPartition Discoveryの機能に似ています。

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

    `hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing`ディレクトリには、次のファイルが含まれています：

    `[hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv, hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv, ...]`

    ファイルパスのcityとutc_dateフィールドが抽出されます

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

6. Hive外部テーブルからインポートし、ソーステーブルのuuid列をグローバル辞書を介してビットマップ型に変換します。

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
