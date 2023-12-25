---
displayed_sidebar: Chinese
---

# 生成列

StarRocks はバージョン3.1から生成列（Generated Column）をサポートしています。この機能は、複雑な式を含むクエリを高速化するために、事前に計算された式の結果を保存することをサポートし、[クエリ改写](#クエリ改写)もサポートしているため、クエリのパフォーマンスを大幅に向上させます。

式の結果を保存するために、一つまたは複数の生成列を定義することができます。同じ式を含むクエリを実行するとき、オプティマイザーはクエリ改写を行い、式を生成列で置き換えます。また、生成列のデータを直接クエリすることもできます。

インポートフェーズで式を計算すると時間がかかるため、**生成列がインポートパフォーマンスに与える影響を事前に評価することをお勧めします**。さらに、テーブル作成後に生成列を追加または変更することは時間がかかり、コストがかかるため、**テーブル作成時に生成列を作成することをお勧めします**（[テーブル作成時に生成列を作成することを推奨](#テーブル作成時に生成列を作成することを推奨)）。

## 基本的な使い方

### 生成列の作成

#### 構文

```SQL
col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

#### テーブル作成時に生成列を作成する（推奨）

`test_tbl1` テーブルを作成し、5つの列を含む。`newcol1` と `newcol2` は生成列で、それぞれ `data_array` と `data_json` の通常の列を参照し、計算式を通じて生成された列です。

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

**注意事項：**

- 生成列は通常の列の後になければなりません。
- 生成列の式では集約関数を使用できません。
- 生成列の式では他の生成列や[オートインクリメント列](./auto_increment.md)を参照できませんが、複数の通常の列を参照することはできます。
- 生成列のデータ型は式の戻り値のデータ型と一致している必要があります。
- 集約テーブルでは生成列を作成することはできません。
- StarRocks のストレージと計算の分離モードでは、この機能は現在サポートされていません。

#### テーブル作成後に生成列を追加する

> **注意**
>
> この操作は時間がかかり、コストがかかるため、テーブル作成時に生成列を追加することをお勧めします。ALTER TABLE を使用して生成列を追加する必要がある場合は、コストと時間を事前に評価してください。

1. `test_tbl2` テーブルを作成し、`id`、`data_array`、`data_json` の3つの通常の列を含みます。テーブル作成後に1行のデータを挿入します。

      ```SQL
      -- テーブル作成
      CREATE TABLE test_tbl2
      (
          id INT NOT NULL,
          data_array ARRAY<int> NOT NULL,
          data_json JSON NOT NULL
       )
      PRIMARY KEY (id)
      DISTRIBUTED BY HASH(id);
      
      -- データ挿入
      INSERT INTO test_tbl2 VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
      
      -- データクエリ
      MySQL [example_db]> select * from test_tbl2;
      +------+------------+------------------+
      | id   | data_array | data_json        |
      +------+------------+------------------+
      |    1 | [1,2]      | {"a": 1, "b": 2} |
      +------+------------+------------------+
      1 row in set (0.04 sec)
      ```

2. ALTER TABLE ... ADD COLUMN ... を実行して、`newcol1` と `newcol2` の生成列を追加します。これらはそれぞれ `data_array` と `data_json` の通常の列を参照し、計算式を通じて生成された列です。

      ```SQL
      ALTER TABLE test_tbl2
      ADD COLUMN newcol1 DOUBLE AS array_avg(data_array);
      
      ALTER TABLE test_tbl2
      ADD COLUMN newcol2 STRING AS json_string(json_query(data_json, "$.a"));
      ```

    **注意事項：**

    - 集約テーブルに生成列を追加することはできません。
    - 通常の列は必ず生成列の前に配置されるため、ALTER TABLE ... ADD COLUMN ... を使用して通常の列を追加する際に、新しい通常の列の位置を指定しない場合、システムは自動的に生成列の前に追加します。また、AFTER を使用して生成列の後に追加することはできません。

3. テーブルのデータをクエリします。

      ```SQL
      MySQL [example_db]> SELECT * FROM test_tbl2;
      +------+------------+------------------+---------+---------+
      | id   | data_array | data_json        | newcol1 | newcol2 |
      +------+------------+------------------+---------+---------+
      |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
      +------+------------+------------------+---------+---------+
      1 row in set (0.04 sec)
      ```

    結果は、`newcol1` と `newcol2` の生成列がテーブルに追加され、StarRocks が計算式を通じて自動的に生成列の値を導き出したことを示しています。

### 生成列へのデータのインポート

データをインポートする際、StarRocks は計算式を通じて自動的に生成列の値を導き出します。**生成列の値を指定することはできません**。ここでは [INSERT INTO](../../loading/InsertInto.md) を使用してデータをインポートする例を説明します。

1. INSERT INTO を使用して `test_tbl1` テーブルに1行のデータを挿入します。`VALUES ()` の中で生成列の値を指定することはできません。

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
      1 row in set (0.01 sec)
      ```

    結果は、StarRocks が計算式を通じて自動的に `newcol1` と `newcol2` の生成列の値を導き出したことを示しています。

    **注意事項：**<br />データをインポートする際に生成列の値を指定すると、以下のようなエラーが返されます：

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
> この操作は時間がかかり、コストがかかるため、生成列を変更する必要がある場合は、コストと時間を事前に評価してください。

生成列のデータ型と式を変更することができます。

1. `test_tbl3` テーブルを作成し、5つの列を含む。`newcol1` と `newcol2` は生成列で、それぞれ `data_array` と `data_json` の通常の列を参照し、計算式を通じて生成された列です。テーブル作成後に1行のデータを挿入します。

      ```SQL
      -- テーブル作成
      MySQL [example_db]> CREATE TABLE test_tbl3
      (
          id INT NOT NULL,
          data_array ARRAY<int> NOT NULL,
          data_json JSON NOT NULL,
          -- 以下のように生成列のデータ型と式を指定
          newcol1 DOUBLE AS array_avg(data_array),
          newcol2 STRING AS json_string(json_query(data_json, "$.a"))
      )
      PRIMARY KEY (id)
      DISTRIBUTED BY HASH(id);
      
      -- データ挿入
      INSERT INTO test_tbl3 (id, data_array, data_json)
          VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
      
      -- データクエリ
      MySQL [example_db]> select * from test_tbl3;
      +------+------------+------------------+---------+---------+
      | id   | data_array | data_json        | newcol1 | newcol2 |
      +------+------------+------------------+---------+---------+
      |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
      +------+------------+------------------+---------+---------+
      1 row in set (0.01 sec)

      ```

2. 生成列 `newcol1` と `newcol2` の変更:
   - 生成列 `newcol1` のデータ型を `ARRAY<INT>` に変更し、式を `data_array` に変更します。

        ```SQL
        ALTER TABLE test_tbl3 
        MODIFY COLUMN newcol1 ARRAY<INT> AS data_array;
        ```

   - 生成列 `newcol2` の式を変更して、通常列 `data_json` のフィールド `b` の値を抽出します。

        ```SQL
        ALTER TABLE test_tbl3
        MODIFY COLUMN newcol2 STRING AS json_string(json_query(data_json, "$.b"));
        ```

3. 変更後のテーブル構造とデータを確認します。

    - 変更後のテーブル構造を確認

        ```SQL
        MySQL [example_db]>   SHOW CREATE TABLE test_tbl3\G
        **** 1. row ****
            Table: test_tbl3
        Create Table: CREATE TABLE test_tbl3 (
        id int(11) NOT NULL COMMENT "",
        data_array array<int(11)> NOT NULL COMMENT "",
        data_json json NOT NULL COMMENT "",
        -- 変更後の生成列のデータ型と式は以下の通りです
        newcol1 array<int(11)> NULL AS example_db.test_tbl3.data_array COMMENT "",
        newcol2 varchar(65533) NULL AS json_string(json_query(example_db.test_tbl3.data_json, '$.b')) COMMENT ""
        ) ENGINE=OLAP 
        PRIMARY KEY(id)
        DISTRIBUTED BY HASH(id)
        PROPERTIES (...);
        1 row in set (0.00 sec)
        ```

    - 変更後のテーブルデータを確認し、結果はStarRocksが変更後の式に基づいて生成列 `newcol1` と `newcol2` の値を再計算したことを示します。

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

テーブル `test_tbl3` から生成列 `newcol1` を削除します。

```SQL
ALTER TABLE test_tbl3 DROP COLUMN newcol1;
```

**注意事項：**<br />生成列の式が通常列を参照していて、その通常列を削除または変更する必要がある場合は、通常列を削除または変更する前に、生成列を先に削除する必要があります。

### クエリの書き換え

クエリ内の式が生成列の式と一致する場合、オプティマイザは自動的にクエリを書き換えて、生成列の値を直接読み取ります。

1. `test_tbl4` を作成したと仮定し、そのテーブル構造は以下の通りです:

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

2. `SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;` で `test_tbl4` のデータをクエリする場合、クエリ文が通常列 `data_array` と `data_json` のみに関連しているにもかかわらず、クエリ内の式が生成列 `newcol1` と `newcol2` の式と一致するため、実行計画ではオプティマイザが自動的にクエリを書き換えて、生成列 `newcol1` と `newcol2` の値を直接読み取ることが示されます。

      ```SQL
      MySQL [example_db]> EXPLAIN SELECT array_avg(data_array), json_string(json_query(data_json, "$.a")) FROM test_tbl4;
      +---------------------------------------+
      | Explain String                        |
      +---------------------------------------+
      | PLAN FRAGMENT 0                       |
      |  OUTPUT EXPRS:4: newcol1 | 5: newcol2 | -- クエリ書き換え後に生成列 newcol1 と newcol2 がヒット
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

### 部分列の更新と生成列

主キーテーブルに[部分列の更新](../../loading/Load_to_Primary_Key_tables.md#部分更新)を行う場合、`columns` に生成列が参照するすべての通常列を指定する必要があります。以下は Stream Load を使用した例です。

1. `test_tbl5` テーブルを作成し、5つの列を含むようにします。`newcol1` と `newcol2` は生成列で、通常列 `data_array` と `data_json` を参照し、計算式によって生成されます。テーブル作成後、1行のデータを挿入します。

      ```SQL
      -- テーブル作成
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
      
      -- データを1行挿入
      INSERT INTO test_tbl5 (id, data_array, data_json)
          VALUES (1, [1,2], parse_json('{"a" : 1, "b" : 2}'));
      
      -- テーブルのデータを確認
      MySQL [example_db]> select * from test_tbl5;
      +------+------------+------------------+---------+---------+
      | id   | data_array | data_json        | newcol1 | newcol2 |
      +------+------------+------------------+---------+---------+
      |    1 | [1,2]      | {"a": 1, "b": 2} |     1.5 | 1       |
      +------+------------+------------------+---------+---------+
      1 row in set (0.01 sec)
      ```

2. 部分列を更新するための CSV ファイル `my_data1.csv` を準備します。

      ```csv
      1|[3,4]|{"a": 3, "b": 4}
      2|[3,4]|{"a": 3, "b": 4} 
      ```

3. [Stream Load](../sql-statements/data-manipulation/STREAM_LOAD.md) を使用して CSV ファイル `my_data1.csv` でテーブル `test_tbl5` の部分列を更新します。`"partial_update:true"` を設定し、`columns` に生成列が参照するすべての通常列を指定する必要があります。

      ```Bash
      curl --location-trusted -u <username>:<password> -H "label:1" \
          -H "column_separator:|" \
          -H "partial_update:true" \
          -H "columns:id,data_array,data_json" \ 
          -T my_data1.csv -XPUT \
          http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
      ```

4. テーブルのデータを確認します。

      ```SQL
      MySQL [example_db]> select * from test_tbl5;
      +------+------------+------------------+---------+---------+
      | id   | data_array | data_json        | newcol1 | newcol2 |
      +------+------------+------------------+---------+---------+
      |    1 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
      |    2 | [3,4]      | {"a": 3, "b": 4} |     3.5 | 3       |
      +------+------------+------------------+---------+---------+
      2 rows in set (0.01 sec)
      ```

部分列の更新を行う際に、生成列が参照するすべての通常列を指定しなかった場合、Stream Load はエラーを返します。

1. CSV ファイル `my_data2.csv` を準備します。

      ```csv
      1|[3,4]
      2|[3,4]
      ```

2. [Stream Load](../sql-statements/data-manipulation/STREAM_LOAD.md) を使用して CSV ファイル `my_data2.csv` で部分列を更新します。`my_data2.csv` には `data_json` 列の値が提供されておらず、Stream Load のインポートジョブの `columns` パラメータに `data_json` 列が含まれていないため、`data_json` 列がNULLを許容していても、生成列 `newcol2` が `data_json` 列を参照しているため、Stream Load はエラーを返します。

      ```SQL
      $ curl --location-trusted -u root: -H "label:2" \
          -H "column_separator:|" \
          -H "partial_update:true" \
          -H "columns:id,data_array" \
          -T my_data2.csv -XPUT \
          http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
      {

          "TxnId": 3134,
          "Label": "2",
          "Status": "Fail",
          "Message": "カラム data_json は、マテリアライズドカラム newcol2 の式評価に使用する必要があります",
          ...
      }
      ```