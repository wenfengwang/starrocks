---
displayed_sidebar: "Japanese"
---

# 生成列

v3.1以降、StarRocksは生成列をサポートしています。生成列は、複雑な式を持つクエリの実行を高速化するために使用できます。この機能は、式の結果と[クエリの書き換え](#クエリの書き換え)の事前計算と結果の格納をサポートしており、同じ複雑な式を持つクエリの実行を大幅に高速化します。

テーブル作成時に、式の結果を格納するための1つ以上の生成列を定義できます。したがって、定義した生成列に結果が格納されている式を含むクエリを実行する場合、CBOはクエリを書き換えて生成列からデータを直接読み取るようにします。また、生成列内のデータに直接クエリを実行することもできます。

また、**式の計算には時間がかかるため、生成列の読み込みパフォーマンスへの影響を評価することをお勧めします**。さらに、テーブル作成後に生成列を追加または変更するよりも、**[テーブル作成時に生成列を作成することをお勧めします](#テーブル作成時に生成列を作成することをお勧めします)**。なぜなら、テーブル作成後に生成列を追加または変更するのは時間がかかり、コストがかかるからです。

## 基本操作

### 生成列の作成

#### 構文

```SQL
<col_name> <data_type> [NULL] AS <expr> [COMMENT 'string']
```

#### テーブル作成時に生成列を作成する（推奨）

`test_tbl1`という名前のテーブルを作成し、5つの列を持つテーブルを作成します。列`newcol1`と`newcol2`は生成列であり、それぞれ指定された式を使用して通常の列`data_array`と`data_json`の値を参照して計算されます。

```SQL
CREATE TABLE test_tbl1
(
    id INT NOT NULL,
    data_array ARRAY<int> NOT NULL,
    data_json JSON NOT NULL,
    newcol1 DOUBLE AS array_avg(data_array),
    newcol2 String AS json_string(json_query(data_json, "$.a"))
)
PRIMARY KEY (id)
DISTRIBUTED BY HASH(id);
```

**注意**：

- 生成列は通常の列の後に定義する必要があります。
- 集計関数は生成列の式では使用できません。
- 生成列の式は他の生成列または[自動インクリメント列](./auto_increment.md)を参照することはできませんが、式は複数の通常の列を参照することができます。
- 生成列のデータ型は、生成列の式によって生成される結果のデータ型と一致する必要があります。
- 生成列は集計テーブルに作成することはできません。
- 現在、StarRocksの共有データモードは生成列をサポートしていません。

#### テーブル作成後に生成列を追加する

> **注意**
>
> この操作は時間とリソースを消費します。したがって、テーブル作成時に生成列を追加することをお勧めします。テーブル作成後にALTER TABLEを使用して生成列を追加する場合は、事前にコストと時間を評価することをお勧めします。

1. `id`、`data_array`、`data_json`の3つの通常の列を持つ`test_tbl2`という名前のテーブルを作成します。テーブルにデータ行を挿入します。

    ```SQL
    -- テーブルを作成します。
    CREATE TABLE test_tbl2
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- データ行を挿入します。
    INSERT INTO test_tbl2 VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- テーブルをクエリします。
    MySQL [example_db]> select * from test_tbl2;
    +------+------------+------------------+
    | id   | data_array | data_json        |
    +------+------------+------------------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |
    +------+------------+------------------+
    1 row in set (0.04 sec)
    ```

2. ALTER TABLE ... ADD COLUMN ... を実行して、生成列`newcol1`と`newcol2`を追加します。これらの生成列は、通常の列`data_array`と`data_json`の値に基づいて式を評価して作成されます。

    ```SQL
    ALTER TABLE test_tbl2
    ADD COLUMN newcol1 DOUBLE AS array_avg(data_array);

    ALTER TABLE test_tbl2
    ADD COLUMN newcol2 String AS json_string(json_query(data_json, "$.a"));
    ```

    **注意**：

    - 集計テーブルに生成列を追加することはサポートされていません。
    - 生成列を追加する前に通常の列を定義する必要があります。ALTER TABLE ... ADD COLUMN ... ステートメントを使用して新しい通常の列を追加する場合、システムは自動的に生成列の前に配置します。さらに、AFTERを使用して生成列の後に明示的に通常の列を配置することはできません。

3. テーブルデータをクエリします。

    ```SQL
    MySQL [example_db]> SELECT * FROM test_tbl2;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 row in set (0.04 sec)
    ```

    結果から、生成列`newcol1`と`newcol2`がテーブルに追加され、StarRocksが式に基づいて自動的に値を計算していることがわかります。

### 生成列へのデータのロード

データのロード中、StarRocksは式に基づいて生成列の値を自動的に計算します。生成列の値を指定することはできません。次の例では、[INSERT INTO](../../loading/InsertInto.md)ステートメントを使用してデータをロードします。

1. `test_tbl1`テーブルにレコードを挿入するために、INSERT INTOステートメントを使用します。ただし、`VALUES ()`句内で生成列の値を指定することはできません。

    ```SQL
    INSERT INTO test_tbl1 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
    ```

2. テーブルデータをクエリします。

    ```SQL
    MySQL [example_db]> SELECT * FROM test_tbl1;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 row in set (0.01 sec)
    ```

    結果から、StarRocksが式に基づいて生成列`newcol1`と`newcol2`の値を自動的に計算していることがわかります。

    **注意**：

    データのロード中に生成列の値を指定すると、次のエラーが返されます。

    ```SQL
    MySQL [example_db]> INSERT INTO test_tbl1 (id, data_array, data_json, newcol1, newcol2) 
    VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): Getting analyzing error. Detail message: materialized column 'newcol1' can not be specified.

    MySQL [example_db]> INSERT INTO test_tbl1 VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): Getting analyzing error. Detail message: Column count doesn't match value count.
    ```

### 生成列の変更

> **注意**
>
> この操作は時間とリソースを消費します。ALTER TABLEを使用して生成列を変更する場合は、事前にコストと時間を評価することをお勧めします。

生成列のデータ型と式を変更することができます。

1. `newcol1`と`newcol2`の生成列を持つ5つの列を持つ`test_tbl3`という名前のテーブルを作成します。テーブルにデータ行を挿入します。

    ```SQL
    -- テーブルを作成します。
    MySQL [example_db]> CREATE TABLE test_tbl3
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL,
        -- 生成列のデータ型と式は次のように指定されます：
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 String AS json_string(json_query(data_json, "$.a"))
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- データ行を挿入します。
    INSERT INTO test_tbl3 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- テーブルをクエリします。
    MySQL [example_db]> select * from test_tbl3;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 row in set (0.01 sec)
    ```

2. 生成列`newcol1`と`newcol2`を変更します：

    - 生成列`newcol1`のデータ型を`ARRAY<INT>`に変更し、式を`data_array`に変更します。

        ```SQL
        ALTER TABLE test_tbl3 
        MODIFY COLUMN newcol1 ARRAY<INT> AS data_array;
        ```

    - 生成列`newcol2`の式を変更し、通常の列`data_json`からフィールド`b`の値を抽出します。

        ```SQL
        ALTER TABLE test_tbl3
        MODIFY COLUMN newcol2 String AS json_string(json_query(data_json, "$.b"));
        ```

3. 変更されたスキーマとテーブルのデータを表示します。

    - 変更されたスキーマを表示します。

        ```SQL
        MySQL [example_db]> show create table test_tbl3\G
        **** 1. row ****
            Table: test_tbl3
        Create Table: CREATE TABLE test_tbl3 (
        id int(11) NOT NULL COMMENT "",
        data_array array<int(11)> NOT NULL COMMENT "",
        data_json json NOT NULL COMMENT "",
        -- 変更後、生成列のデータ型と式は次のようになります：
        newcol1 array<int(11)> NULL AS example_db.test_tbl3.data_array COMMENT "",
        newcol2 varchar(65533) NULL AS json_string(json_query(example_db.test_tbl3.data_json, '$.b')) COMMENT ""
        ) ENGINE=OLAP 
        PRIMARY KEY(id)
        DISTRIBUTED BY HASH(id)
        PROPERTIES (...);
        1 row in set (0.00 sec)
        ```

    - 変更後のテーブルデータをクエリします。結果から、StarRocksが変更された式に基づいて生成列`newcol1`と`newcol2`の値を再計算していることがわかります。

        ```SQL
        MySQL [example_db]> select * from test_tbl3;
        +------+------------+------------------+---------+---------+
        | id   | data_array | data_json        | newcol1 | newcol2 |
        +------+------------+------------------+---------+---------+
        |    1 | [1,2]      | {"a": 1, "b": 2} | [1,2]   | 2       |
        +------+------------+------------------+---------+---------+
        1 row in set (0.01 sec)
        ```

### 生成列の削除

テーブル`test_tbl3`から列`newcol1`を削除します。

```SQL
ALTER TABLE test_tbl3 DROP COLUMN newcol1;
```

> **注意**：
>
> 生成列が式で通常の列を参照している場合、通常の列を直接削除または変更することはできません。代わりに、まず生成列を削除し、その後通常の列を削除または変更する必要があります。

### クエリの書き換え

クエリの式が生成列の式と一致する場合、オプティマイザはクエリを自動的に生成列の値を直接読み取るように書き換えます。

1. 次のスキーマを持つ`test_tbl4`という名前のテーブルを作成します。

    ```SQL
    CREATE TABLE test_tbl4
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL,
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 String AS json_string(json_query(data_json, "$.a"))
    )
    PRIMARY KEY (id) DISTRIBUTED BY HASH(id);
    ```

2. `SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;`ステートメントを使用して`test_tbl4`テーブルのデータをクエリする場合、クエリは通常の列`data_array`と`data_json`のみを含みます。しかし、クエリの式は生成列`newcol1`と`newcol2`の式と一致します。この場合、実行計画ではCBOがクエリを生成列`newcol1`と`newcol2`の値を読み取るように自動的に書き換えることが示されます。

    ```SQL
    MySQL [example_db]> EXPLAIN SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;
    +---------------------------------------+
    | Explain String                        |
    +---------------------------------------+
    | PLAN FRAGMENT 0                       |
    |  OUTPUT EXPRS:4: newcol1 | 5: newcol2 | -- クエリは生成列newcol1とnewcol2のデータを読み取るように書き換えられます。
    |   PARTITION: RANDOM                   |
    |                                       |
    |   RESULT SINK                         |
    |                                       |
    |   0:OlapScanNode                      |
    |      TABLE: test_tbl4                 |
    |      PREAGGREGATION: ON               |
    |      partitions=0/1                   |
    |      rollup: test_tbl4                |
    |      tabletRatio=0/0                  |
    |      tabletList=                      |
    |      cardinality=1                    |
    |      avgRowSize=2.0                   |
    +---------------------------------------+
    15 rows in set (0.00 sec)
    ```

### 部分更新と生成列

主キーテーブルの部分更新を実行するには、生成列によって参照されるすべての通常の列を`columns`パラメータで指定する必要があります。次の例では、Stream Loadを使用して部分更新を実行します。

1. `newcol1`と`newcol2`の生成列を持つ5つの列を持つ`test_tbl5`という名前のテーブルを作成します。テーブルにデータ行を挿入します。

    ```SQL
    -- テーブルを作成します。
    CREATE TABLE test_tbl5
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NULL,
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 String AS json_string(json_query(data_json, "$.a"))
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- データ行を挿入します。
    INSERT INTO test_tbl5 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- テーブルをクエリします。
    MySQL [example_db]> select * from test_tbl5;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 row in set (0.01 sec)
    ```

2. `my_data1.csv`ファイルを準備して、`test_tbl5`テーブルの一部の列を更新します。

    ```csv
    1|[3,4]|{"a": 3, "b": 4}
    2|[3,4]|{"a": 3, "b": 4} 
    ```

3. `partial_update:true`を設定し、`columns`パラメータで生成列によって参照されるすべての通常の列を指定して、[Stream Load](../../loading/StreamLoad.md)を使用して`test_tbl5`テーブルの一部の列を更新します。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:1" \
        -H "column_separator:|" \
        -H "partial_update:true" \
        -H "columns:id,data_array,data_json" \ 
        -T my_data1.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
    ```

4. テーブルデータをクエリします。

    ```SQL
    [example_db]> select * from test_tbl5;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
    |    2 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
    +------+------------+------------------+---------+---------+
    2 rows in set (0.01 sec)
    ```

生成列によって参照される通常の列を指定せずに部分更新を実行すると、Stream Loadによってエラーが返されます。

1. `my_data2.csv`ファイルを準備します。

    ```csv
    1|[3,4]
    2|[3,4]
    ```

2. `my_data2.csv`ファイルを使用して[Stream Load](../../loading/StreamLoad.md)を実行して部分列の更新を行う場合、`my_data2.csv`内で`data_json`列の値を指定せず、Stream Loadの`columns`パラメータに`data_json`列が含まれていない場合、`data_json`列がnull値を許可していても、Stream Loadによってエラーが返されます。これは、生成列`newcol2`が通常の列`data_json`を参照しているためです。
