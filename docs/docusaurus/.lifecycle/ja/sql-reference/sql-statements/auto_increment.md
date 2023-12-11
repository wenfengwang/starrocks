---
displayed_sidebar: "Japanese"
---

# AUTO_INCREMENT

スターロックスはバージョン3.0から`AUTO_INCREMENT`カラム属性をサポートしており、これによりデータ管理を簡素化することができます。このトピックでは、`AUTO_INCREMENT`カラム属性の適用シナリオ、使用法、機能について説明します。

## はじめに

新しいデータ行がテーブルにロードされ、`AUTO_INCREMENT`カラムの値が指定されていない場合、スターロックスはその行の`AUTO_INCREMENT`カラムに対してテーブル全体で一意なIDとして整数値を自動的に割り当てます。`AUTO_INCREMENT`カラムの後続の値は、行のIDから特定のステップで自動的に増加します。`AUTO_INCREMENT`カラムは、データ管理を簡素化し、いくつかのクエリを高速化するために使用することができます。以下は、`AUTO_INCREMENT`カラムの適用シナリオです。

- 主キーとしての利用：`AUTO_INCREMENT`カラムは、各行が一意のIDを持ち、データの照会と管理を容易にするために主キーとして使用できます。
- テーブルの結合：複数のテーブルを結合する際、`AUTO_INCREMENT`カラムを結合キーとして使用することで、STRING型の列（例：UUID）を使用するよりもクエリを迅速化することができます。
- 高基数列の一意な値の数を数える：`AUTO_INCREMENT`カラムは、辞書内の一意な値列を表現するために使用できます。直接的に一意なSTRING値を数えるよりも、`AUTO_INCREMENT`カラムの整数値を数える方が、クエリのスピードを数倍または数十倍向上させることがあります。

`CREATE TABLE`ステートメントで`AUTO_INCREMENT`カラムを指定する必要があります。`AUTO_INCREMENT`カラムのデータ型はBIGINTでなければなりません。`AUTO_INCREMENT`カラムの値は[暗黙的に割り当てられるか、明示的に指定される](#assign-values-for-auto_increment-column)ことができます。始まりの値は1であり、新しい行ごとに1ずつ増加します。

## 基本的な操作

### テーブル作成時に`AUTO_INCREMENT`カラムを指定

2つの列、`id`と`number`を持つ`test_tbl1`という名前のテーブルを作成し、列`number`を`AUTO_INCREMENT`カラムとして指定します。

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

### `AUTO_INCREMENT`カラムへの値の割り当て

#### 暗黙的に値を割り当てる

スターロックスのテーブルにデータをロードする際、`AUTO_INCREMENT`カラムの値を指定する必要はありません。スターロックスはそのカラムに一意な整数値を自動的に割り当て、テーブルに挿入します。

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

スターロックスのテーブルにデータをロードする際、`AUTO_INCREMENT`カラムの値も`DEFAULT`として指定することができます。スターロックスはそのカラムに一意な整数値を自動的に割り当て、テーブルに挿入します。

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

実際の使用において、テーブル内のデータを表示すると、以下のような結果が返されることがあります。これは、スターロックスが`AUTO_INCREMENT`カラムの値が厳密に単調ではないことを保証できないためです。しかし、スターロックスは値がおおよそ年代順に増加することを保証できます。詳細については[Monotonicity](#monotonicity)を参照してください。

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

`AUTO_INCREMENT`カラムの値を明示的に指定し、テーブルに挿入することもできます。

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

さらに、明示的な値の指定は、新しく挿入されるデータ行に対してスターロックスが生成する後続の値に影響を与えません。

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

`AUTO_INCREMENT`カラムに対して暗黙的に割り当てられた値と明示的に指定された値を同時に使用しないことをお勧めします。指定された値がスターロックスが生成する値と同じである可能性があり、[auto-incremented IDsのグローバルユニーク性](#uniqueness)が破られる可能性があります。

## 基本的な機能

### ユニーク性

一般的に、スターロックスは`AUTO_INCREMENT`カラムの値がテーブル全体で一意であることを保証します。`AUTO_INCREMENT`カラムに対して値を暗黙的に割り当てることと明示的に指定することを同時に行わないようお勧めします。これを行うと、auto-incremented IDsのグローバルユニーク性が破られる可能性があります。以下は簡単な例です：`id`と`number`という2つの列を持つ`test_tbl2`という名前のテーブルを作成し、列`number`を`AUTO_INCREMENT`カラムとして指定します。

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

`test_tbl2`で`AUTO_INCREMENT`カラム`number`に対して暗黙的に値を割り当て、明示的に指定します。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

テーブル`test_tbl2`を照会します。

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

auto-incremented IDsの割り当てのパフォーマンスを向上させるために、BEはいくつかのauto-incremented IDsをローカルにキャッシュしています。この状況では、スターロックスは`AUTO_INCREMENT`カラムの値が厳密に単調であることを保証できません。おおよそ年代順に増加することだけを保証することができます。

> **注記**
>
> BEによってキャッシュされるauto-incremented IDsの数は、FEの動的パラメータ`auto_increment_cache_size`によって決定されます。デフォルトは`100,000`です。`ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");`を使用してこの値を変更できます。

例えば、スターロックスクラスターには1つのFEノードと2つのBEノードがあります。以下のようにして名前が`test_tbl3`のテーブルを作成し、データを5行挿入します。

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
```
      INSERT INTO test_tbl3 VALUES (4, DEFAULT);
      INSERT INTO test_tbl3 VALUES (5, DEFAULT);
    ```

    テーブル`test_tbl3`の自動インクリメントIDは、2つのBEノードがそれぞれ[1, 100000]および[100001, 200000]という自動インクリメントIDをキャッシュするため、単調増加しない場合があります。複数の`INSERT`ステートメントを使用してデータをロードすると、異なるBEノードにデータが送信され、それぞれが独自に自動インクリメントIDを割り当てるため、自動インクリメントIDが厳密に単調であることを保証できません。

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

    ## 部分更新および`AUTO_INCREMENT`列

    このセクションでは、`AUTO_INCREMENT`列を含むテーブルの指定された列のみを更新する方法について説明します。

    > **注意**
    >
    > 現在、プライマリキーテーブルのみが部分更新をサポートしています。

    ### `AUTO_INCREMENT`列がプライマリキーの場合

    部分更新時にプライマリキーを指定する必要があります。そのため、`AUTO_INCREMENT`列がプライマリキーまたはプライマリキーの一部である場合、部分更新のユーザー動作は、`AUTO_INCREMENT`列が定義されていない場合とまったく同じです。

    1. データベース`example_db`にテーブル`test_tbl4`を作成し、1つのデータ行を挿入します。

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

        -- データを用意します。
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

    2. テーブル`test_tbl4`を更新するためのCSVファイル**my_data4.csv**を準備します。CSVファイルには`AUTO_INCREMENT`列の値が含まれておらず、`job1`列の値が含まれています。最初の行のプライマリキーは既にテーブル`test_tbl4`に存在しますが、2番目の行のプライマリキーは存在しません。

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

    4. 更新されたテーブルをクエリします。データの最初の行は既にテーブル`test_tbl4`に存在し、列`job1`の値は変更されません。2番目の行のデータは新しく挿入されます。また、`job1`列のデフォルト値が指定されていないため、部分更新フレームワークはこの列の値を直接`0`に設定します。

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

    ### `AUTO_INCREMENT`列がプライマリキーでない場合

    `AUTO_INCREMENT`列がプライマリキーではないかプライマリキーの一部でない場合、かつStream Loadジョブで自動インクリメントIDが提供されていない場合、以下の状況が発生します。

    - 行が既にテーブルに存在する場合、StarRocksは自動インクリメントIDを更新しません。
    - 行がテーブルに新しくロードされる場合、StarRocksは新しい自動インクリメントIDを生成します。

    この機能は、DISTINCT STRING値を迅速に計算するための辞書テーブルを構築するために使用できます。

    1. データベース`example_db`で、テーブル`test_tbl5`を作成し、列`job1`を`AUTO_INCREMENT`列とし、テーブル`test_tbl5`にデータ行を挿入します。

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

        -- データを用意します。
        mysql > INSERT INTO test_tbl5 VALUES (0, 0, -1, -1);
        Query OK, 1 row affected (0.04 sec)
        {'label':'insert_458d9487-80f6-11ed-ae56-aa528ccd0ebf', 'status':'VISIBLE', 'txnId':'94'}

        -- テーブルをクエリします。
        mysql > SELECT * FROM test_tbl5 ORDER BY id;
        +------+------+------+------+
        | id   | name | job1 | job2 |
        +------+------+------+------+
        |    0 |    0 |   -1 |   -1 |
        +------+------+------+------+
        1 row in set (0.01 sec)
        ```

    2. テーブル`test_tbl5`を更新するためのCSVファイル**my_data5.csv**を準備します。CSVファイルには`AUTO_INCREMENT`列`job1`の値が含まれていません。最初の行のプライマリキーは既にテーブル`test_tbl5`に存在する一方、2番目と3番目の行のプライマリキーは存在しません。

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

    4. 更新されたテーブルをクエリします。データの最初の行は既にテーブル`test_tbl5`に存在するため、`AUTO_INCREMENT`列`job1`は元の値のままです。2番目と3番目のデータ行は新しく挿入されます。そのため、StarRocksは`AUTO_INCREMENT`列`job1`の新しい値を生成します。

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

    - `AUTO_INCREMENT`列を持つテーブルを作成する際、`'replicated_storage' = 'true'`を設定して、すべてのレプリカが同じ自動インクリメントIDを持つようにする必要があります。
    - 各テーブルには`AUTO_INCREMENT`列を1つだけ持つことができます。
    - `AUTO_INCREMENT`列のデータ型はBIGINTである必要があります。
    - `AUTO_INCREMENT`列は`NOT NULL`である必要があり、デフォルト値を持っていません。
    - プライマリキーを持つテーブルからデータを削除することができますが、`AUTO_INCREMENT`列がプライマリキーでない場合やプライマリキーの一部でない場合、次のシナリオでデータを削除する際に以下の制限事項に注意する必要があります。
```
- DELETEオペレーション中には、部分的な更新用のロードジョブも実行されます。このジョブにはUPSERTオペレーションのみが含まれます。UPSERTおよびDELETEオペレーションの両方が同じデータ行に影響を与えた場合、DELETEオペレーションの後にUPSERTオペレーションが実行されると、UPSERTオペレーションが適用されない可能性があります。

- 同じデータ行に複数のUPSERTおよびDELETEオペレーションを含む部分的な更新用のロードジョブがあります。特定のUPSERTオペレーションがDELETEオペレーションの後に実行された場合、UPSERTオペレーションが適用されない可能性があります。

- ALTER TABLEを使用してAUTO_INCREMENT属性を追加することはサポートされていません。

- バージョン3.1から、StarRocksの共有データモードはAUTO_INCREMENT属性をサポートしています。

- StarRocksはAUTO_INCREMENT列の開始値とステップサイズを指定することをサポートしていません。