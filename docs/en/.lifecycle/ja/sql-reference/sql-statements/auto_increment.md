---
displayed_sidebar: "Japanese"
---

# AUTO_INCREMENT（自動増分）

StarRocksはバージョン3.0以降、`AUTO_INCREMENT`列属性をサポートしており、データ管理を簡素化することができます。このトピックでは、`AUTO_INCREMENT`列属性の適用シナリオ、使用方法、および機能について説明します。

## 概要

テーブルに新しいデータ行をロードし、`AUTO_INCREMENT`列の値が指定されていない場合、StarRocksはその行の`AUTO_INCREMENT`列に整数値を自動的に割り当て、テーブル全体で一意のIDとして使用します。`AUTO_INCREMENT`列の後続の値は、行のIDを起点として特定のステップで自動的に増加します。`AUTO_INCREMENT`列は、データ管理を簡素化し、一部のクエリの処理を高速化するために使用できます。以下は`AUTO_INCREMENT`列の適用シナリオのいくつかです。

- プライマリキーとして使用：`AUTO_INCREMENT`列は、各行に一意のIDがあることを保証し、データのクエリと管理を容易にするためにプライマリキーとして使用できます。
- テーブルの結合：複数のテーブルを結合する場合、`AUTO_INCREMENT`列は結合キーとして使用できます。これにより、データ型がSTRING（例：UUID）である列を使用する場合に比べてクエリの処理が高速化されます。
- 高基数列の一意な値の数をカウント：`AUTO_INCREMENT`列は、辞書内の一意な値列を表すために使用できます。直接STRINGの一意な値をカウントするよりも、`AUTO_INCREMENT`列の整数値の一意な値をカウントすることで、クエリの処理速度を数倍または数十倍向上させることがあります。

`CREATE TABLE`ステートメントで`AUTO_INCREMENT`列を指定する必要があります。`AUTO_INCREMENT`列のデータ型はBIGINTである必要があります。`AUTO_INCREMENT`列の値は[暗黙的に割り当てるか、明示的に指定する](#assign-values-for-auto_increment-column)ことができます。値は1から始まり、新しい行ごとに1ずつ増加します。

## 基本操作

### テーブル作成時に`AUTO_INCREMENT`列を指定する

`id`と`number`の2つの列を持つ`test_tbl1`という名前のテーブルを作成し、列`number`を`AUTO_INCREMENT`列として指定します。

```SQL
CREATE TABLE test_tbl1
(
    id BIGINT NOT NULL, 
    number BIGINT NOT NULL AUTO_INCREMENT
) 
PRIMARY KEY (id) 
DISTRIBUTED BY HASH(id)
PROPERTIES("replicated_storage" = "true");
```

### `AUTO_INCREMENT`列に値を割り当てる

#### 暗黙的に値を割り当てる

StarRocksテーブルにデータをロードする際、`AUTO_INCREMENT`列の値を指定する必要はありません。StarRocksは自動的に一意の整数値をその列に割り当て、テーブルに挿入します。

```SQL
INSERT INTO test_tbl1 (id) VALUES (1);
INSERT INTO test_tbl1 (id) VALUES (2);
INSERT INTO test_tbl1 (id) VALUES (3),(4),(5);
```

テーブル内のデータを表示します。

```SQL
mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 |      2 |
|    3 |      3 |
|    4 |      4 |
|    5 |      5 |
+------+--------+
5 rows in set (0.02 sec)
```

StarRocksテーブルにデータをロードする際、`AUTO_INCREMENT`列の値を`DEFAULT`として指定することもできます。StarRocksは自動的に一意の整数値をその列に割り当て、テーブルに挿入します。

```SQL
INSERT INTO test_tbl1 (id, number) VALUES (6, DEFAULT);
```

テーブル内のデータを表示します。

```SQL
mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 |      2 |
|    3 |      3 |
|    4 |      4 |
|    5 |      5 |
|    6 |      6 |
+------+--------+
6 rows in set (0.02 sec)
```

実際の使用では、テーブル内のデータを表示する際に次の結果が返されることがあります。これは、StarRocksが`AUTO_INCREMENT`列の値が厳密に単調ではないことを保証できないためです。ただし、StarRocksは値がおおよそ時系列順に増加することを保証できます。詳細については、[単調性](#monotonicity)を参照してください。

```SQL
mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 | 200002 |
|    5 | 200003 |
|    6 | 200004 |
+------+--------+
6 rows in set (0.01 sec)
```

#### 明示的に値を指定する

`AUTO_INCREMENT`列の値を明示的に指定し、テーブルに挿入することもできます。

```SQL
INSERT INTO test_tbl1 (id, number) VALUES (7, 100);

-- テーブル内のデータを表示します。

mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 | 200002 |
|    5 | 200003 |
|    6 | 200004 |
|    7 |    100 |
+------+--------+
7 rows in set (0.01 sec)
```

さらに、値を明示的に指定しても、StarRocksが新しく挿入されるデータ行に対して生成する後続の値には影響しません。

```SQL
INSERT INTO test_tbl1 (id) VALUES (8);

-- テーブル内のデータを表示します。

mysql > SELECT * FROM test_tbl1 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 | 200002 |
|    5 | 200003 |
|    6 | 200004 |
|    7 |    100 |
|    8 |      2 |
+------+--------+
8 rows in set (0.01 sec)
```

**注意**

`AUTO_INCREMENT`列に対して暗黙的に割り当てられた値と明示的に指定された値を同時に使用しないことをお勧めします。なぜなら、指定された値がStarRocksによって生成された値と同じである可能性があり、[自動増分IDのグローバル一意性](#uniqueness)が破られるためです。

## 基本機能

### 一意性

一般的に、StarRocksは`AUTO_INCREMENT`列の値がテーブル全体で一意であることを保証します。`AUTO_INCREMENT`列に対して暗黙的に割り当てられた値と明示的に指定された値を同時に使用しないことをお勧めします。そうすると、自動増分IDのグローバル一意性が破られる可能性があります。以下は簡単な例です。`number`列を`AUTO_INCREMENT`列として指定し、`id`と`number`の2つの列を持つ`test_tbl2`という名前のテーブルを作成します。

```SQL
CREATE TABLE test_tbl2
(
    id BIGINT NOT NULL,
    number BIGINT NOT NULL AUTO_INCREMENT
 ) 
PRIMARY KEY (id) 
DISTRIBUTED BY HASH(id)
PROPERTIES("replicated_storage" = "true");
```

`test_tbl2`テーブルに`number`列の`AUTO_INCREMENT`列として暗黙的に割り当てられた値と明示的に指定された値を割り当てます。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

`test_tbl2`テーブルをクエリします。

```SQL
mysql > SELECT * FROM test_tbl2 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 |      2 |
|    3 | 100001 |
+------+--------+
3 rows in set (0.08 sec)
```

### 単調性

自動増分IDの割り当てパフォーマンスを向上させるため、BEはいくつかの自動増分IDをローカルにキャッシュしています。この状況では、StarRocksは`AUTO_INCREMENT`列の値が厳密に単調ではないことを保証できません。値がおおよそ時系列順に増加することだけが保証されます。

> **注意**
>
> BEがキャッシュする自動増分IDの数は、FEの動的パラメータ`auto_increment_cache_size`によって決まります。デフォルト値は`100,000`です。`ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");`を使用して値を変更できます。

例えば、StarRocksクラスタには1つのFEノードと2つのBEノードがあります。`test_tbl3`という名前のテーブルを作成し、次のように5つのデータ行を挿入します。

```SQL
CREATE TABLE test_tbl3
(
    id BIGINT NOT NULL,
    number BIGINT NOT NULL AUTO_INCREMENT
) 
PRIMARY KEY (id)
DISTRIBUTED BY HASH(id)
PROPERTIES("replicated_storage" = "true");

INSERT INTO test_tbl3 VALUES (1, DEFAULT);
INSERT INTO test_tbl3 VALUES (2, DEFAULT);
INSERT INTO test_tbl3 VALUES (3, DEFAULT);
INSERT INTO test_tbl3 VALUES (4, DEFAULT);
INSERT INTO test_tbl3 VALUES (5, DEFAULT);
```

`test_tbl3`テーブルの自動増分IDは単調に増加しないため、2つのBEノードはそれぞれ[1, 100000]と[100001, 200000]の自動増分IDをキャッシュしています。複数のINSERTステートメントでデータをロードする場合、データは異なるBEノードに送信され、それぞれのBEノードが独立して自動増分IDを割り当てます。そのため、自動増分IDが厳密に単調であることは保証されません。

```SQL
mysql > SELECT * FROM test_tbl3 ORDER BY id;
+------+--------+
| id   | number |
+------+--------+
|    1 |      1 |
|    2 | 100001 |
|    3 | 200001 |
|    4 |      2 |
|    5 | 100002 |
+------+--------+
5 rows in set (0.07 sec)
```

## 部分更新と`AUTO_INCREMENT`列

このセクションでは、`AUTO_INCREMENT`列を含むテーブルの一部の指定された列のみを更新する方法について説明します。

> **注意**
>
> 現在、部分更新はプライマリキーテーブルのみサポートされています。

### `AUTO_INCREMENT`列がプライマリキーの場合

部分更新時にはプライマリキーを指定する必要があります。したがって、`AUTO_INCREMENT`列がプライマリキーまたはプライマリキーの一部である場合、部分更新のユーザー動作は`AUTO_INCREMENT`列が定義されていない場合とまったく同じです。

1. データベース`example_db`に`test_tbl4`という名前のテーブルを作成し、1つのデータ行を挿入します。

    ```SQL
    -- テーブルを作成します。
    CREATE TABLE test_tbl4
    (
        id BIGINT AUTO_INCREMENT,
        name BIGINT NOT NULL,
        job1 BIGINT NOT NULL,
        job2 BIGINT NOT NULL
    ) 
    PRIMARY KEY (id, name)
    DISTRIBUTED BY HASH(id)
    PROPERTIES("replicated_storage" = "true");

    -- データを準備します。
    mysql > INSERT INTO test_tbl4 (id, name, job1, job2) VALUES (0, 0, 1, 1);
    Query OK, 1 row affected (0.04 sec)
    {'label':'insert_6af28e77-7d2b-11ed-af6e-02424283676b', 'status':'VISIBLE', 'txnId':'152'}

    -- テーブルをクエリします。
    mysql > SELECT * FROM test_tbl4 ORDER BY id;
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |    1 |    1 |
    +------+------+------+------+
    1 row in set (0.01 sec)
    ```

2. テーブル`test_tbl4`を更新するためのCSVファイル**my_data4.csv**を準備します。CSVファイルには`AUTO_INCREMENT`列の値が含まれており、列`job1`の値は含まれていません。最初の行のプライマリキーは既にテーブル`test_tbl4`に存在しますが、2番目の行のプライマリキーはテーブルに存在しません。

    ```Plaintext
    0,0,99
    1,1,99
    ```

3. [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)ジョブを実行し、CSVファイルを使用してテーブル`test_tbl4`を更新します。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:1" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns:id,name,job2" \
        -T my_data4.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl4/_stream_load
    ```

4. 更新されたテーブルをクエリします。データの最初の行は既にテーブル`test_tbl4`に存在するため、`AUTO_INCREMENT`列`job1`の値は変更されません。2番目の行は新しく挿入され、列`job1`のデフォルト値が指定されていないため、部分更新フレームワークはこの列の値を`0`に直接設定します。

    ```SQL
    mysql > SELECT * FROM test_tbl4 ORDER BY id;
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |    1 |   99 |
    |    1 |    1 |    0 |   99 |
    +------+------+------+------+
    2 rows in set (0.01 sec)
    ```

### `AUTO_INCREMENT`列がプライマリキーではない場合

`AUTO_INCREMENT`列がプライマリキーまたはプライマリキーの一部ではなく、自動増分IDがStream Loadジョブで提供されない場合、次の状況が発生します。

- テーブル内の行が既に存在する場合、StarRocksは自動増分IDを更新しません。
- テーブルに新しくロードされる行の場合、StarRocksは新しい自動増分IDを生成します。

この機能は、高速にDISTINCTなSTRING値を計算するための辞書テーブルを構築するために使用できます。

1. データベース`example_db`に`test_tbl5`という名前のテーブルを作成し、列`job1`を`AUTO_INCREMENT`列として指定し、1つのデータ行をテーブル`test_tbl5`に挿入します。

    ```SQL
    -- テーブルを作成します。
    CREATE TABLE test_tbl5
    (
        id BIGINT NOT NULL,
        name BIGINT NOT NULL,
        job1 BIGINT NOT NULL AUTO_INCREMENT,
        job2 BIGINT NOT NULL
    )
    PRIMARY KEY (id, name)
    DISTRIBUTED BY HASH(id)
    PROPERTIES("replicated_storage" = "true");

    -- データを準備します。
    mysql > INSERT INTO test_tbl5 VALUES (0, 0, -1, -1);
    Query OK, 1 row affected (0.04 sec)
    {'label':'insert_458d9487-80f6-11ed-ae56-aa528ccd0ebf', 'status':'VISIBLE', 'txnId':'94'}

    -- テーブルをクエリします。
    mysql > SELECT * FROM test_tbl5 ORDER BY id;
    +------+------+--------+------+
    | id   | name | job1   | job2 |
    +------+------+--------+------+
    |    0 |    0 |     -1 |   -1 |
    +------+------+--------+------+
    1 row in set (0.01 sec)
    ```

2. テーブル`test_tbl5`を更新するためのCSVファイル**my_data5.csv**を準備します。CSVファイルには`AUTO_INCREMENT`列`job1`の値は含まれておらず、列`job1`のデフォルト値が指定されていません。最初の行のプライマリキーは既にテーブル`test_tbl5`に存在しますが、2番目と3番目の行のプライマリキーはテーブルに存在しません。

    ```Plaintext
    0,0,99
    1,1,99
    2,2,99
    ```

3. [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)ジョブを実行し、CSVファイルからテーブル`test_tbl5`にデータをロードします。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:2" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns: id,name,job2" \
        -T my_data5.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
    ```

4. 更新されたテーブルをクエリします。データの最初の行は既にテーブル`test_tbl5`に存在するため、`AUTO_INCREMENT`列`job1`は元の値を保持します。2番目と3番目の行は新しく挿入され、`AUTO_INCREMENT`列`job1`のためにStarRocksが新しい値を生成します。

    ```SQL
    mysql > SELECT * FROM test_tbl5 ORDER BY id;
    +------+------+--------+------+
    | id   | name | job1   | job2 |
    +------+------+--------+------+
    |    0 |    0 |     -1 |   99 |
    |    1 |    1 |      1 |   99 |
    |    2 |    2 | 100001 |   99 |
    +------+------+--------+------+
    3 rows in set (0.01 sec)
    ```

## 制限事項

- `AUTO_INCREMENT`列を持つテーブルを作成する際には、`'replicated_storage' = 'true'`を設定する必要があります。これにより、すべてのレプリカで同じ自動増分IDが持たれるようになります。
- 各テーブルには、`AUTO_INCREMENT`列は1つだけ持つことができます。
- `AUTO_INCREMENT`列のデータ型はBIGINTである必要があります。
- `AUTO_INCREMENT`列は`NOT NULL`である必要があり、デフォルト値を持つことはできません。
- `AUTO_INCREMENT`属性を使用してALTER TABLEを行うことはサポートされていません。
- StarRocksの共有データモードは、バージョン3.1以降、`AUTO_INCREMENT`属性をサポートしています。
- StarRocksは、`AUTO_INCREMENT`列の開始値とステップサイズを指定することはサポートしていません。

## キーワード

AUTO_INCREMENT, AUTO INCREMENT
