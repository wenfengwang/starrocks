---
displayed_sidebar: English
---

# 生成列

v3.1以降、StarRocksは生成列をサポートしています。生成列は、複雑な式を含むクエリの高速化に使用できます。この機能は、式の結果を事前計算して保存し、[クエリの書き換え](#query-rewrites)をサポートしており、同じ複雑な式を含むクエリの大幅な高速化を実現します。

テーブル作成時に、1つ以上の生成列を定義して式の結果を格納することができます。そのため、定義した生成列に格納されている式を含むクエリを実行する際、CBOはクエリを書き換えて生成列から直接データを読み取るようにします。また、生成列に格納されているデータを直接クエリすることも可能です。

**式の計算には時間がかかるため、生成列がロードパフォーマンスに与える影響を評価することが推奨されます**。さらに、**テーブル作成後に生成列を追加または変更するのではなく、[テーブル作成時に生成列を作成する](#create-generated-columns-at-table-creation-recommended)ことを推奨します**。テーブル作成後に生成列を追加または変更することは、時間とコストがかかります。

## 基本操作

### 生成列の作成

#### 構文

```SQL
<col_name> <data_type> [NULL] AS <expr> [COMMENT 'string']
```

#### テーブル作成時に生成列を作成する（推奨）

`test_tbl1`という名前のテーブルを作成し、5つの列のうち`newcol1`と`newcol2`を生成列として定義します。これらの生成列は、指定された式を使用して計算され、通常の列`data_array`と`data_json`をそれぞれ参照します。

```SQL
CREATE TABLE test_tbl1
(
    id INT NOT NULL,
    data_array ARRAY<int> NOT NULL,
    data_json JSON NOT NULL,
    newcol1 DOUBLE AS array_avg(data_array),
    newcol2 STRING AS json_string(json_query(data_json, "$.a"))
)
PRIMARY KEY (id)
DISTRIBUTED BY HASH(id);
```

**注意**:

- 生成列は通常の列の後に定義する必要があります。
- 集約関数は生成列の式で使用できません。
- 生成列の式は他の生成列や[自動インクリメント列](./auto_increment.md)を参照することはできませんが、複数の通常の列を参照することは可能です。
- 生成列のデータ型は、生成列の式によって生成される結果のデータ型と一致している必要があります。
- 集約テーブルに生成列を作成することはできません。
- 現在、StarRocksの共有データモードでは生成列はサポートされていません。

#### テーブル作成後に生成列を追加する

> **注意**
>
> この操作は時間とリソースを大量に消費します。そのため、テーブル作成時に生成列を追加することを推奨します。ALTER TABLEを使用して生成列を追加する場合は、事前にコストと時間を評価することが推奨されます。

1. `id`、`data_array`、`data_json`の3つの通常の列を持つ`test_tbl2`という名前のテーブルを作成し、データ行をテーブルに挿入します。

    ```SQL
    -- テーブルを作成する。
    CREATE TABLE test_tbl2
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL
    )
    PRIMARY KEY (id)
    DISTRIBUTED BY HASH(id);

    -- データ行を挿入する。
    INSERT INTO test_tbl2 VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));

    -- テーブルをクエリする。
    MySQL [example_db]> SELECT * FROM test_tbl2;
    +------+------------+------------------+
    | id   | data_array | data_json        |
    +------+------------+------------------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |
    +------+------------+------------------+
    1行セット (0.04秒)
    ```

2. ALTER TABLE ... ADD COLUMN ... を実行して、通常の列`data_array`と`data_json`の値に基づいて式を評価することによって作成される生成列`newcol1`と`newcol2`を追加します。

    ```SQL
    ALTER TABLE test_tbl2
    ADD COLUMN newcol1 DOUBLE AS array_avg(data_array);

    ALTER TABLE test_tbl2
    ADD COLUMN newcol2 STRING AS json_string(json_query(data_json, "$.a"));
    ```

    **注意**:

    - 集約テーブルに生成列を追加することはサポートされていません。
    - 通常の列は生成列の前に定義する必要があります。ALTER TABLE ... ADD COLUMN ... ステートメントを使用して新しい通常の列を追加する際に、新しい通常の列の位置を指定しない場合、システムは自動的に生成列の前に配置します。また、AFTERを使用して通常の列を生成列の後に明示的に配置することはできません。

3. テーブルのデータをクエリします。

    ```SQL
    MySQL [example_db]> SELECT * FROM test_tbl2;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1行セット (0.04秒)
    ```

    結果は、生成列`newcol1`と`newcol2`がテーブルに追加され、StarRocksが式に基づいてその値を自動的に計算したことを示しています。

### 生成列へのデータのロード

データロード中、StarRocksは式に基づいて生成列の値を自動的に計算します。生成列の値を指定することはできません。次の例では、[INSERT INTO](../../loading/InsertInto.md)ステートメントを使用してデータをロードします。

1. INSERT INTOを使用して、`test_tbl1`テーブルにレコードを挿入します。`VALUES ()`句内で生成列の値を指定することはできません。

    ```SQL
    INSERT INTO test_tbl1 (id, data_array, data_json)
    VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
    ```

2. テーブルのデータをクエリします。

    ```SQL
    MySQL [example_db]> SELECT * FROM test_tbl1;
    +------+------------+------------------+---------+---------+
    | id   | data_array | data_json        | newcol1 | newcol2 |
    +------+------------+------------------+---------+---------+
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1行セット (0.01秒)
    ```

    結果は、StarRocksが生成列`newcol1`と`newcol2`の値を式に基づいて自動的に計算したことを示しています。

    **注意**:

    データロード中に生成列の値を指定すると、次のエラーが返されます。

    ```SQL
    MySQL [example_db]> INSERT INTO test_tbl1 (id, data_array, data_json, newcol1, newcol2) 
    VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): 分析エラーを取得しました。詳細メッセージ: 実体化された列 'newcol1' を指定することはできません。

    MySQL [example_db]> INSERT INTO test_tbl1 VALUES (2, [3,4], parse_json('{"a" : 3, "b" : 4}'), 3.5, "3");
    ERROR 1064 (HY000): 分析エラーを取得しました。詳細メッセージ: 列の数が値の数と一致しません。
    ```

### 生成列の変更

> **注意**
>
> この操作は時間とリソースを大量に消費します。ALTER TABLEを使用して生成列を変更することが避けられない場合は、事前にコストと時間を評価することが推奨されます。

生成列のデータ型と式を変更することができます。

1. `data_array`と`data_json`の通常の列を参照し、指定された式を使用して値が計算される生成列`newcol1`と`newcol2`を持つ`test_tbl3`という名前のテーブルを作成し、データ行をテーブルに挿入します。

    ```SQL
    -- テーブルを作成する。
    MySQL [example_db]> CREATE TABLE test_tbl3
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL,
        -- 生成列のデータ型と式は以下のように指定されます：
        newcol1 DOUBLE AS array_avg(data_array),
        newcol2 STRING AS json_string(json_query(data_json, "$.a"))
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
    1行セット (0.01秒)
    ```

2. 生成された列 `newcol1` と `newcol2` を変更しました：

    - 生成された列 `newcol1` のデータ型を `ARRAY<INT>` に変更し、その式を `data_array` に変更します。

        ```SQL
        ALTER TABLE test_tbl3 
        MODIFY COLUMN newcol1 ARRAY<INT> AS data_array;
        ```

    - 生成された列 `newcol2` の式を変更して、通常の列 `data_json` からフィールド `b` の値を抽出します。

        ```SQL
        ALTER TABLE test_tbl3
        MODIFY COLUMN newcol2 String AS json_string(json_query(data_json, "$.b"));
        ```

3. 変更されたスキーマとテーブル内のデータを表示します。

    - 変更されたスキーマを表示します。

        ```SQL
        MySQL [example_db]> show create table test_tbl3\G
        **** 1. row ****
            Table: test_tbl3
        Create Table: CREATE TABLE test_tbl3 (
        id int(11) NOT NULL COMMENT '',
        data_array array<int(11)> NOT NULL COMMENT '',
        data_json json NOT NULL COMMENT '',
        -- 変更後、生成された列のデータ型と式は以下の通りです：
        newcol1 array<int(11)> NULL AS example_db.test_tbl3.data_array COMMENT '',
        newcol2 varchar(65533) NULL AS json_string(json_query(example_db.test_tbl3.data_json, '$.b')) COMMENT ''
        ) ENGINE=OLAP 
        PRIMARY KEY(id)
        DISTRIBUTED BY HASH(id)
        PROPERTIES (...);
        1行セット (0.00秒)
        ```

    - 変更後のテーブルデータをクエリします。結果は、StarRocksが変更された式に基づいて生成された列 `newcol1` と `newcol2` の値を再計算することを示しています。

        ```SQL
        MySQL [example_db]> select * from test_tbl3;
        +------+------------+------------------+---------+---------+
        | id   | data_array | data_json        | newcol1 | newcol2 |
        +------+------------+------------------+---------+---------+
        |    1 | [1,2]      | {"a": 1, "b": 2} | [1,2]   | 2       |
        +------+------------+------------------+---------+---------+
        1行セット (0.01秒)
        ```

### 生成された列を削除する

テーブル `test_tbl3` から列 `newcol1` を削除します。

```SQL
ALTER TABLE test_tbl3 DROP COLUMN newcol1;
```

> **注意**：
>
> 生成された列が式内で通常の列を参照している場合、その通常の列を直接削除または変更することはできません。まず生成された列を削除し、その後で通常の列を削除または変更する必要があります。

### クエリの書き換え

クエリの式が生成された列の式と一致する場合、オプティマイザはクエリを自動的に書き換えて、生成された列の値を直接読み取るようにします。

1. 次のスキーマでテーブル `test_tbl4` を作成するとします：

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

2. ステートメント `SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;` を使用してテーブル `test_tbl4` のデータをクエリする場合、クエリは通常の列 `data_array` と `data_json` のみを含みます。しかし、クエリの式は生成された列 `newcol1` と `newcol2` の式と一致しています。この場合、実行計画は、CBOがクエリを自動的に書き換えて生成された列 `newcol1` と `newcol2` の値を読み取ることを示しています。

    ```SQL
    MySQL [example_db]> EXPLAIN SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;
    +---------------------------------------+
    | Explain String                        |
    +---------------------------------------+
    | PLAN FRAGMENT 0                       |
    |  OUTPUT EXPRS:4: newcol1 | 5: newcol2 | -- クエリは生成された列 newcol1 と newcol2 からデータを読み取るように書き換えられます。
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
    15行セット (0.00秒)
    ```

### 部分更新と生成された列

プライマリキーテーブルで部分更新を実行するには、`columns` パラメータで生成された列によって参照されるすべての通常の列を指定する必要があります。以下の例では、Stream Loadを使用して部分更新を実行します。

1. 5つの列を持つテーブル `test_tbl5` を作成し、そのうちの列 `newcol1` と `newcol2` は生成された列で、指定された式を使用して通常の列 `data_array` と `data_json` の値を参照して値が計算されます。データ行をテーブルに挿入します。

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
    1行セット (0.01秒)
    ```

2. テーブル `test_tbl5` の一部の列を更新するためのCSVファイル `my_data1.csv` を準備します。

    ```SQL
    1|[3,4]|{"a": 3, "b": 4}
    2|[3,4]|{"a": 3, "b": 4} 
    ```

3. CSVファイル `my_data1.csv` を使用して、Stream Loadでテーブル `test_tbl5` の一部の列を更新します。`partial_update:true` を設定し、`columns` パラメータで生成された列によって参照されるすべての通常の列を指定する必要があります。

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
    2行セット (0.01秒)
    ```

生成された列によって参照されるすべての通常の列を指定せずに部分更新を実行すると、Stream Loadによってエラーが返されます。

1. CSVファイル `my_data2.csv` を準備します。

      ```csv
      1|[3,4]
      2|[3,4]
      ```
2. [Stream Load](../../loading/StreamLoad.md)を使用して`my_data2.csv`ファイルで部分的な列の更新を行う場合、`my_data2.csv`内で`data_json`列の値が提供されておらず、Stream Loadジョブの`columns`パラメータに`data_json`列が含まれていないと、`data_json`列がNULL値を許容するとしても、生成された列`newcol2`が`data_json`列を参照しているため、Stream Loadはエラーを返します。