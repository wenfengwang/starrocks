---
displayed_sidebar: "Japanese"
---

# 生成された列

v3.1から、StarRocksは生成された列をサポートしています。生成された列を使用すると、複雑な式を持つクエリを高速化することができます。この機能は、式の結果を事前計算および保存し、[クエリの書き換え](#query-rewrites)をサポートしており、同じ複雑な式を持つクエリを大幅に高速化します。

テーブルの作成時に、1つ以上の生成された列を定義して式の結果を保存することができます。これにより、定義した生成された列を含む式を実行するクエリでは、CBOはクエリを生成された列からデータを直接読むように書き換えます。また、生成された列内のデータに直接クエリを実行することもできます。

また、計算式には時間がかかるため、生成された列がロードパフォーマンスに与える影響を**評価することが推奨されます**。さらに、テーブル作成後に**生成された列を作成することを推奨します**[テーブル作成時に生成された列を作成](#create-generated-columns-at-table-creation-recommended)。テーブル作成後に生成された列を追加または変更することは時間がかかりコストがかかるためです。

## 基本的な操作

### 生成された列の作成

#### 構文

```SQL
<col_name> <data_type> [NULL] AS <expr> [COMMENT 'string']
```

#### テーブル作成時に生成された列を作成（推奨される方法）

"test_tbl1"という名前のテーブルを作成し、そのテーブルには、"newcol1"および"newcol2"という列があります。これらの列は、それぞれ指定された式を使用して計算されており、通常の列"data_array"および"data_json"の値を参照しています。

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

- 生成された列は通常の列の後に定義する必要があります。
- 式で集約関数を使用することはできません。
- 生成された列の式では他の生成された列または[自動増分列](./auto_increment.md)を参照することはできませんが、式では複数の通常の列を参照することができます。
- 生成された列のデータ型は、生成された列の式によって生成された結果のデータ型と一致する必要があります。
- 生成された列は、集約テーブル上に作成することはできません。
- 現在のところ、StarRocksのshared-dataモードは生成された列をサポートしていません。

#### テーブル作成後に生成された列を追加する

> **NOTICE**
>
> この操作は時間がかかりリソースが必要です。したがって、テーブル作成時に生成された列を追加することを推奨します。ALTER TABLEを使用して生成された列を追加する必要がある場合は、事前にコストと時間を評価することをお勧めします。

1. "test_tbl2"という名前のテーブルを作成し、そこに3つの通常の列"id"、"data_array"、"data_json"を作成します。テーブルにデータ行を挿入します。

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

2. ALTER TABLE ... ADD COLUMN ...を実行して、"data_array"および"data_json"の値に基づいて式を評価して作成された"newcol1"および"newcol2"という生成された列を追加します。

    ```SQL
    ALTER TABLE test_tbl2
    ADD COLUMN newcol1 DOUBLE AS array_avg(data_array);

    ALTER TABLE test_tbl2
    ADD COLUMN newcol2 String AS json_string(json_query(data_json, "$.a"));
    ```

    **NOTICE**:

    - 集約テーブルに生成された列を追加することはサポートされていません。
    - 通常の列は生成された列の前に定義する必要があります。ALTER TABLE ... ADD COLUMN ...ステートメントを使用して新しい通常の列を追加する際に、新しい通常の列の位置を指定せずに使用すると、システムは自動的に生成された列の前に配置します。さらに、AFTERを明示的に使用して通常の列を生成された列の後に配置することはできません。

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

    結果では、生成された列"newcol1"および"newcol2"がテーブルに追加され、StarRocksが式に基づいて値を自動的に計算することが示されています。

### 生成された列のデータをロードする

データのロード中、StarRocksは式に基づいて生成された列の値を自動的に計算します。生成された列の値を指定することはできません。次の例では、[INSERT INTO](../../loading/InsertInto.md)ステートメントを使用してデータをロードします。

1. INSERT INTOを使用して"test_tbl1"テーブルにレコードを挿入します。なお、"VALUES ()"節内で生成された列の値を指定することはできません。

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

    結果では、StarRocksが式に基づいて生成された列"newcol1"および"newcol2"の値を自動的に計算することが示されています。

    **NOTICE**:

    データのロード中に生成された列の値を指定すると、次のエラーが返されることになります:

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
> この操作は時間がかかりリソースが必要です。ALTER TABLEを使用して生成された列を変更する必要がある場合は、事前にコストと時間を評価することをお勧めします。

生成された列のデータ型および式を変更することができます。

1. "test_tbl3"というテーブルを作成し、そのテーブルには、"newcol1"および"newcol2"という列があります。これらの列は、それぞれ指定された式を使用して計算されており、通常の列"data_array"および"data_json"の値を参照しています。テーブルにデータ行を挿入します。

    ```SQL
    -- テーブルを作成します。
    MySQL [example_db]> CREATE TABLE test_tbl3
    (
        id INT NOT NULL,
        data_array ARRAY<int> NOT NULL,
        data_json JSON NOT NULL,
        -- 生成された列のデータ型と式は次のように指定されています:
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
```markdown
    |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
    +------+------------+------------------+---------+---------+
    1 行をセット (0.01 秒)

2. 生成された列 `newcol1` と `newcol2` を変更しました:

    - 生成された列 `newcol1` のデータ型を `ARRAY<INT>` に変更し、その式を `data_array` に変更します。

        ```SQL
        ALTER TABLE test_tbl3 
        MODIFY COLUMN newcol1 ARRAY<INT> AS data_array;
        ```

    - 生成された列 `newcol2` の式を変更し、正規列 `data_json` からフィールド `b` の値を抽出します。

        ```SQL
        ALTER TABLE test_tbl3
        MODIFY COLUMN newcol2 String AS json_string(json_query(data_json, "$.b"));
        ```

3. 変更後のスキーマとテーブルのデータを表示します。

    - 変更後のスキーマを表示します。

        ```SQL
        MySQL [example_db]> show create table test_tbl3\G
        **** 1 行 ****
            テーブル: test_tbl3
        Create Table: CREATE TABLE test_tbl3 (
        id int(11) NOT NULL COMMENT "",
        data_array array<int(11)> NOT NULL COMMENT "",
        data_json json NOT NULL COMMENT "",
        -- 変更後、生成された列のデータ型および式は以下の通りです:
        newcol1 array<int(11)> NULL AS example_db.test_tbl3.data_array COMMENT "",
        newcol2 varchar(65533) NULL AS json_string(json_query(example_db.test_tbl3.data_json, '$.b')) COMMENT ""
        ) ENGINE=OLAP 
        PRIMARY KEY(id)
        DISTRIBUTED BY HASH(id)
        PROPERTIES (...);
        1 行をセット (0.00 秒)
        ```

    - 変更後のテーブルデータをクエリします。結果では、StarRocks が変更された式に基づいて生成列 `newcol1` と `newcol2` の値を再計算します。

        ```SQL
        MySQL [example_db]> select * from test_tbl3;
        +------+------------+------------------+---------+---------+
        | id   | data_array | data_json        | newcol1 | newcol2 |
        +------+------------+------------------+---------+---------+
        |    1 | [1,2]      | {"a": 1, "b": 2} | [1,2]   | 2       |
        +------+------------+------------------+---------+---------+
        1 行をセット (0.01 秒)

### 生成された列を削除する

テーブル `test_tbl3` から列 `newcol1` を削除します。

```SQL
ALTER TABLE test_tbl3 DROP COLUMN newcol1;
```

> **注意**:
>
> 生成列が式で正規列を参照している場合、正規列を直接削除または変更することはできません。代わりに、生成された列をまず削除し、その後正規列を削除または変更する必要があります。

### クエリの書き換え

クエリ内の式が生成された列の式と一致する場合、オプティマイザーは自動的にクエリを書き換えて生成列の値を直接読み取るようにします。

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

2. `SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;` ステートメントを使用してテーブル `test_tbl4` のデータをクエリした場合、クエリには正規列 `data_array` と `data_json` のみが関与します。しかし、クエリの式は生成された列 `newcol1` と `newcol2` の式と一致します。この場合、実行計画によると CBO は自動的にクエリを書き換えて生成列 `newcol1` と `newcol2` の値を読み取るようにします。

    ```SQL
    MySQL [example_db]> EXPLAIN SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;
    +---------------------------------------+
    | Explain String                        |
    +---------------------------------------+
    | PLAN FRAGMENT 0                       |
    |  OUTPUT EXPRS:4: newcol1 | 5: newcol2 | -- クエリは、生成列 newcol1 および newcol2 からのデータを読み取るように書き換えられています。
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
    15 行をセット (0.00 秒)
    ```

### 部分更新と生成列

主キー テーブルの部分更新を行うためには、生成列で参照されるすべての正規列を `columns` パラメータで指定する必要があります。以下の例では、部分更新を行うために Stream Load を使用します。

1. 列 `newcol1` と `newcol2` が生成列であり、それぞれの値が指定された式を使用して正規列 `data_array` と `data_json` の値を参照しているテーブル `test_tbl5` を作成し、データ行をテーブルに挿入します。

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
    1 行をセット (0.01 秒)
    ```

2. `my_data1.csv` を更新のための CSV ファイルとして準備します。

    ```SQL
    1|[3,4]|{"a": 3, "b": 4}
    2|[3,4]|{"a": 3, "b": 4} 
    ```

3. `my_data1.csv` ファイルを使用して [Stream Load](../../loading/StreamLoad.md) を使用して `test_tbl5` テーブルの一部の列を更新します。`partial_update:true` を設定し、`columns` パラメータで生成列で参照されるすべての正規列を指定する必要があります。

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
    2 行をセット (0.01 秒)

部分更新を行う際に、生成列で参照される正規列を指定せずに部分更新を行うと Stream Load はエラーを返します。

1. `my_data2.csv` という CSV ファイルを準備します。

      ```csv
      1|[3,4]
      2|[3,4]
      ```
```
```
2. [Stream Load](../../loading/StreamLoad.md)を使用して部分カラムの更新を行う場合、`my_data2.csv`ファイルを使用し、`my_data2.csv`に`data_json`カラムの値が提供されておらず、Stream Loadジョブの`columns`パラメータに`data_json`カラムが含まれていない場合、`data_json`カラムがnull値を許可していても、生成された`newcol2`カラムによって`data_json`カラムが参照されるため、Stream Loadはエラーを返します。
```