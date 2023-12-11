---
displayed_sidebar: "Japanese"
---

# 生成された列

v3.1以降、StarRocksは生成された列をサポートしています。生成された列は、複雑な式を持つクエリを加速するために使用できます。この機能は、式の結果を事前に計算して保存し、[クエリの書き換え](#query-rewrites)をサポートし、同じ複雑な式を持つクエリを大幅に高速化します。

テーブル作成時に、1つ以上の生成された列を定義して、式の結果を保存することができます。したがって、定義した生成された列を含む式を含むクエリを実行すると、CBOはクエリを書き換えて生成された列からデータを直接読み取るようにします。また、生成された列のデータを直接クエリできます。

また、**生成された列がロード性能に与える影響を評価することをお勧めします。なぜなら、式の計算には時間がかかるからです**。さらに、テーブル作成後に生成された列を追加または変更するよりも、[テーブル作成時に生成された列を作成すること](#create-generated-columns-at-table-creation-recommended)をお勧めします。生成された列を追加または変更するのには時間とコストがかかるからです。

## 基本操作

### 生成された列の作成

#### 構文

```SQL
<col_name> <data_type> [NULL] AS <expr> [COMMENT 'string']
```

#### テーブル作成時に生成された列を作成する（推奨）

`test_tbl1`という名前のテーブルを作成し、その中に5つの列を作成します。そのうちの`newcol1`と`newcol2`の列は、それぞれ`data_array`と`data_json`という通常の列の値を使用して計算された値を持つ生成された列です。

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

**NOTICE**:

- 生成された列は、通常の列の後に定義する必要があります。
- 式の中で集約関数を使用できません。
- 生成された列の式は他の生成された列または[自動増分列](./auto_increment.md)を参照できませんが、式は複数の通常の列を参照できます。
- 生成された列のデータ型は、生成された列の式によって生成された結果のデータ型と一致する必要があります。
- 生成された列は、集約テーブルに作成できません。
- 現在、StarRocksの共有データモードでは生成された列はサポートされていません。

#### テーブル作成後に生成された列を追加する

> **NOTICE**
>
> この操作は時間とリソースを消費します。したがって、生成された列を追加するには、テーブル作成時に生成された列を追加することをお勧めします。ALTER TABLEを使用して生成された列を追加することを避けられない場合は、事前にそのコストと時間を評価することをお勧めします。

1. `test_tbl2`という名前のテーブルを作成し、3つの通常の列 `id`、`data_array`、`data_json` を持つテーブルを作成します。テーブルにデータ行を挿入します。

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

    -- テーブルを照会します。
    MySQL [example_db]> select * from test_tbl2;
    +------+------------+------------------+
    | id   | data_array | data_json        |
    +------+------------+------------------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |
    +------+------------+------------------+
    1 行がセットされました (0.04 秒)
    ```

2. ALTER TABLE ... ADD COLUMN ... を使用して、`newcol1`と`newcol2`という名前の生成された列を追加します。これらは`data_array`と`data_json`の通常の列の値に基づいて式を評価して作成されます。

    ```SQL
    ALTER TABLE test_tbl2
    ADD COLUMN newcol1 DOUBLE AS array_avg(data_array);

    ALTER TABLE test_tbl2
    ADD COLUMN newcol2 String AS json_string(json_query(data_json, "$.a"));
    ```

    **NOTICE**:

    - 集約テーブルに生成された列を追加することはサポートされていません。
    - 通常の列は、生成された列の前に定義する必要があります。ALTER TABLE ... ADD COLUMN ... ステートメントを使用して、新しい通常の列を指定位置を指定せずに追加した場合、システムはそれを生成された列の前に自動的に配置します。さらに、AFTER を使用して通常の列を明示的に生成された列の後に配置することはできません。

3. テーブルデータを照会します。

    ```SQL
    MySQL [example_db]> SELECT * FROM test_tbl2;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 行がセットされました (0.04 秒)
    ```

    結果は、生成された列 `newcol1` と `newcol2` がテーブルに追加され、StarRocksが式に基づいて自動的に値を計算していることを示しています。

### 生成された列へのデータのロード

データのロード中、StarRocksは式に基づいて生成された列の値を自動的に計算します。生成された列の値を指定することはできません。次の例では、[INSERT INTO](../../loading/InsertInto.md) ステートメントを使用してデータをロードしています。

1. INSERT INTO を使用して `test_tbl1` テーブルにレコードを挿入します。`VALUES ()`句内で生成された列の値を指定できないことに注意してください。

    ```SQL
    INSERT INTO test_tbl1 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
    ```

2. テーブルデータを照会します。

    ```SQL
    MySQL [example_db]> SELECT * FROM test_tbl1;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 行がセットされました (0.01 秒)
    ```

    結果は、StarRocksが式に基づいて生成された列 `newcol1` と `newcol2` の値を自動的に計算していることを示しています。

    **NOTICE**:

    以下のエラーが返される点に注意してください。データをロード中に生成された列の値を指定すると、次のエラーが返されます。

    ```SQL
    MySQL [example_db]> INSERT INTO test_tbl1 (id, data_array, data_json, newcol1, newcol2) 
    VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): Getting analyzing error. Detail message: materialized column 'newcol1' can not be specified.

    MySQL [example_db]> INSERT INTO test_tbl1 VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): Getting analyzing error. Detail message: Column count doesn't match value count.
    ```

### 生成された列の変更

> **NOTICE**
>
> この操作は時間とリソースを消費します。ALTER TABLE を使用して生成された列を変更する場合は、コストと時間を事前に評価することをお勧めします。

生成された列のデータ型や式を変更できます。

1. `test_tbl3`という名前のテーブルを作成し、その中に5つの列を作成します。そのうちの`newcol1`と`newcol2`は、それぞれ`data_array`と`data_json`の通常の列の値を使用して計算された値を持つ生成された列です。テーブルにデータ行を挿入します。

    ```SQL
    -- テーブルを作成します。
    MySQL [example_db]> CREATE TABLE test_tbl3
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL,
        -- 生成された列のデータ型と式を次のように指定します。
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 String AS json_string(json_query(data_json, "$.a"))
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- データ行を挿入します。
    INSERT INTO test_tbl3 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- テーブルを照会します。
    MySQL [example_db]> select * from test_tbl3;
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1行 in set (0.01 sec)
    ```

2. 生成列 `newcol1` と `newcol2` を変更しました。

    - 生成列 `newcol1` のデータ型を `ARRAY<INT>` に変更し、式を `data_array` に変更します。

        ```SQL
        ALTER TABLE test_tbl3 
        MODIFY COLUMN newcol1 ARRAY<INT> AS data_array;
        ```

    - 生成列 `newcol2` の式を変更して、通常の列 `data_json` のフィールド `b` の値を抽出します。

        ```SQL
        ALTER TABLE test_tbl3
        MODIFY COLUMN newcol2 String AS json_string(json_query(data_json, "$.b"));
        ```

3. 変更後のスキーマとテーブル内のデータを表示します。

    - 変更後のスキーマを表示します。

        ```SQL
        MySQL [example_db]> show create table test_tbl3\G
        **** 1. 行 ****
            テーブル: test_tbl3
        テーブルの作成: CREATE TABLE test_tbl3 (
        id int(11) NOT NULL COMMENT "",
        data_array array<int(11)> NOT NULL COMMENT "",
        data_json json NOT NULL COMMENT "",
        -- 変更後、生成列のデータ型と式は以下の通りです:
        newcol1 array<int(11)> NULL AS example_db.test_tbl3.data_array COMMENT "",
        newcol2 varchar(65533) NULL AS json_string(json_query(example_db.test_tbl3.data_json, '$.b')) COMMENT ""
        ) ENGINE=OLAP 
        PRIMARY KEY(id)
        DISTRIBUTED BY HASH(id)
        PROPERTIES (...);
        1 行 in set (0.00 sec)
        ```

    - 変更後のテーブルデータをクエリします。結果には、StarRocks が変更された式に基づいて生成列 `newcol1` と `newcol2` の値を再計算することが示されます。

        ```SQL
        MySQL [example_db]> select * from test_tbl3;
        +------+------------+------------------+---------+---------+
        | id   | data_array | data_json        | newcol1 | newcol2 |
        +------+------------+------------------+---------+---------+
        |    1 | [1,2]      | {"a": 1, "b": 2} | [1,2]   | 2       |
        +------+------------+------------------+---------+---------+
        1 行 in set (0.01 sec)
        ```

### 生成列の削除

テーブル `test_tbl3` から列 `newcol1` を削除します。

```SQL
ALTER TABLE test_tbl3 DROP COLUMN newcol1;
```

> **注意**:
>
> 生成列が式で通常の列を参照している場合、その通常の列を直接削除または変更することはできません。代わりに、まず生成列を削除してから通常の列を削除または変更する必要があります。

### クエリの書き換え

クエリ内の式が生成列の式と一致する場合、オプティマイザは自動的にクエリを書き換えて生成列の値を直接読み取るようにします。

1. 次のスキーマでテーブル `test_tbl4` を作成したとします:

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

2. テーブル `test_tbl4` のデータを次のような `SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;` のステートメントでクエリした場合、クエリには通常の列 `data_array` と `data_json` のみが関与します。しかし、クエリ内の式は生成列 `newcol1` と `newcol2` の式と一致します。この場合、実行計画では CBO がクエリを自動的に生成列 `newcol1` と `newcol2` の値を読み取るように書き換えたことを示します。

    ```SQL
    MySQL [example_db]> EXPLAIN SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;
    +---------------------------------------+
    | Explain String                        |
    +---------------------------------------+
    | PLAN FRAGMENT 0                       |
    |  OUTPUT EXPRS:4: newcol1 | 5: newcol2 | -- クエリが生成列 newcol1 と newcol2 からデータを読み取るように書き換えられています。
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
    15 行 in set (0.00 sec)
    ```

### 部分的な更新と生成列

主キーのテーブルで部分的な更新を実行するには、`columns` パラメータで生成列の参照されるすべての通常の列を指定する必要があります。次の例では、Stream Load を使用して部分的な更新を行います。

1. 列 `newcol1` と `newcol2` を生成列とし、その値はそれぞれ指定された式を使用して計算され、通常の列 `data_array` と `data_json` を参照しています。テーブル `test_tbl5` を作成し、データ行をテーブルに挿入します。

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

    -- データ行をテーブルに挿入します。
    INSERT INTO test_tbl5 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- テーブルをクエリします。
    MySQL [example_db]> select * from test_tbl5;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 行 in set (0.01 sec)
    ```

2. `my_data1.csv` という CSV ファイルを準備して、`test_tbl5` テーブルの一部の列を更新します。

    ```SQL
    1|[3,4]|{"a": 3, "b": 4}
    2|[3,4]|{"a": 3, "b": 4} 
    ```

3. `my_data1.csv` ファイルを使用して [Stream Load](../../loading/StreamLoad.md) を実行し、`test_tbl5` テーブルの一部の列を更新します。`partial_update:true` を設定し、`columns` パラメータで生成列の参照されるすべての通常の列を指定する必要があります。

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
    2 行 in set (0.01 sec)
    ```

生成列の参照されるすべての通常の列を指定せずに部分的な更新を行うと、Stream Load はエラーを返します。

1. `my_data2.csv` という CSV ファイルを準備します。

      ```csv
      1|[3,4]
      2|[3,4]
      ```
```
2. [ストリームロード](../../loading/StreamLoad.md)を使用して`my_data2.csv`ファイルで部分的なカラム更新が実行される際、`my_data2.csv`に`data_json`カラムの値が提供されず、`columns`パラメーターに`data_json`カラムが含まれていない場合、`data_json`カラムがnull値を許可していても、`newcol2`で生成されたカラムによって`data_json`カラムが参照されるため、Stream Loadはエラーを返します。
```