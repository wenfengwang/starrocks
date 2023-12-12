---
displayed_sidebar: "Japanese"
---

# AUTO_INCREMENT（オートインクリメント）

StarRocksはバージョン3.0以降で`AUTO_INCREMENT`カラム属性をサポートしており、これによりデータ管理を簡素化することができます。本トピックでは、`AUTO_INCREMENT`カラム属性の適用シナリオ、使用方法、機能について紹介します。

## 導入

新しいデータ行がテーブルにロードされ、`AUTO_INCREMENT`カラムの値が指定されていない場合、StarRocksはその行の`AUTO_INCREMENT`カラムに対して一意のIDとして整数値を自動的に割り当てます。`AUTO_INCREMENT`カラムの後続の値は、その行のIDから特定のステップで自動的に増加します。`AUTO_INCREMENT`カラムは、データ管理を簡素化し、特定のクエリを高速化するために使用できます。以下は`AUTO_INCREMENT`カラムの適用シナリオです：

- 主キーとしての利用：`AUTO_INCREMENT`カラムは主キーとして使用され、各行が一意のIDを持ち、データのクエリや管理が容易になります。
- テーブルの結合：複数のテーブルを結合する際に、`AUTO_INCREMENT`カラムはJoin Keyとして使用され、例えばSTRING型のカラム（UUIDなど）を使用する場合と比べてクエリを迅速化することができます。
- 高基数カラムにおける一意の値の数を数える：`AUTO_INCREMENT`カラムは、辞書内の一意の値カラムを表すために使用できます。直接文字列の一意の値をカウントするよりも、`AUTO_INCREMENT`カラムの整数値をカウントすることでクエリ速度を数倍、あるいは数十倍向上させることがあります。

`CREATE TABLE`ステートメントで`AUTO_INCREMENT`カラムを指定する必要があります。`AUTO_INCREMENT`カラムのデータ型はBIGINTである必要があります。`AUTO_INCREMENT`カラムの値は[暗黙的に割り当てまたは明示的に指定](#assign-values-for-auto_increment-column)することができます。値は1から始まり、新しい行ごとに1ずつ増加します。

## 基本操作

### テーブル作成時に`AUTO_INCREMENT`カラムを指定

2つの列、`id`と`number`を持つ`test_tbl1`というテーブルを作成し、`number`列を`AUTO_INCREMENT`カラムとして指定します。

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

### `AUTO_INCREMENT`カラムに値を割り当てる

#### 暗黙的に値を割り当てる

StarRocksテーブルにデータをロードする際、`AUTO_INCREMENT`カラムの値を指定する必要はありません。StarRocksはそのカラムに一意の整数値を自動的に割り当て、それをテーブルに挿入します。

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
5 行が選択されました (0.02 秒)
```

StarRocksテーブルにデータをロードする際に、`AUTO_INCREMENT`カラムの値を`DEFAULT`として指定することもできます。この場合も、StarRocksはそのカラムに一意の整数値を自動的に割り当て、それをテーブルに挿入します。

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
6 行が選択されました (0.02 秒)
```

実際の使用では、次に示す結果がテーブル内のデータを表示すると返されることがあります。これは、StarRocksが`AUTO_INCREMENT`カラムの値が厳密に単調ではないことを保証できないためです。ただし、StarRocksは値がおおむね時間順に増加することを保証できます。詳細は[単調性](#monotonicity)を参照してください。

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
6 行が選択されました (0.01 秒)
```

#### 明示的に値を指定する

`AUTO_INCREMENT`カラムの値を明示的に指定し、それをテーブルに挿入することもできます。

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
7 行が選択されました (0.01 秒)
```

また、明示的な値の指定は、StarRocksによって新たに挿入されたデータ行の後続の値に影響を与えません。

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
8 行が選択されました (0.01 秒)
```

**注意**

`AUTO_INCREMENT`カラムに対して暗黙的に割り当てられた値と明示的に指定された値を同時に使用しないことをお勧めします。なぜなら、指定された値がStarRocksによって生成された値と同じである可能性があり、[全体的な一意性が破れる](#uniqueness)可能性があるためです。

## 基本的な機能

### 一意性

一般的に、StarRocksは`AUTO_INCREMENT`カラムの値がテーブル全体で一意であることを保証します。`AUTO_INCREMENT`カラムの値に対して暗黙的に割り当てることと明示的に指定することを同時に行わないことをお勧めします。そうすることで、自動増加されたIDのグローバルな一意性が破れる可能性があります。以下は簡単な例です。`id`と`number`の2つの列を持つ`test_tbl2`というテーブルを作成し、`number`列を`AUTO_INCREMENT`カラムとして指定します。

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

`test_tbl2`テーブルで`AUTO_INCREMENT`カラム`number`の値に対して暗黙的に割り当てたり、明示的に指定することを行います。

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
3 行が選択されました (0.08 秒)
```

### 単調性

`AUTO_INCREMENT`されたIDの割り当てのパフォーマンスを向上させるために、BEは一部の`AUTO_INCREMENT`されたIDをローカルにキャッシュします。この状況下では、StarRocksは`AUTO_INCREMENT`カラムの値が厳密に単調ではないことを保証できません。おおむね時系列順に増加することだけを保証できます。

> **注意**
>
> BEによってキャッシュされる`AUTO_INCREMENT`されたIDの数は、FEの動的パラメータ`auto_increment_cache_size`で決まり、デフォルト値は`100,000`です。`ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");`を使用してこの値を変更することができます。

例えば、StarRocksクラスタにFEノード1台とBEノード2台がある場合、次のようにして`test_tbl3`というテーブルを作成し、データを5行挿入します：

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

テーブル `test_tbl3` の自動増分IDは、2台のBEノードがそれぞれ [1, 100000] と [100001, 200000] の自動増分IDをキャッシュしているため、単調に増加しない可能性があります。複数のINSERTステートメントを使用してデータをロードする際、異なるBEノードにデータが送信され、それぞれが独自に自動増分IDを割り当てます。そのため、自動増分IDの厳密な単調性を保証することはできません。

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

## 部分更新と `AUTO_INCREMENT` カラム

このセクションでは、`AUTO_INCREMENT` カラムを含むテーブルの特定の列のみを更新する方法について説明します。

> **注記**
>
> 現在、プライマリキー テーブルのみが部分更新をサポートしています。

### `AUTO_INCREMENT` カラムがプライマリキーである場合

部分更新時には、プライマリキーを指定する必要があります。したがって、`AUTO_INCREMENT` カラムがプライマリキーまたはプライマリキーの一部である場合、部分更新のユーザー操作は、`AUTO_INCREMENT` カラムが定義されていない場合とまったく同じです。

1. データベース `example_db` にテーブル `test_tbl4` を作成し、1つのデータ行を挿入します。

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

2. 表 `test_tbl4` を更新するための CSV ファイル **my_data4.csv** を準備します。CSV ファイルには `AUTO_INCREMENT` カラムの値が含まれず、`job1` カラムの値が含まれます。最初の行のプライマリキーは既にテーブル `test_tbl4` に存在しますが、2番目の行のプライマリキーはテーブルに存在しません。

    ```Plaintext
    0,0,99
    1,1,99
    ```

3. [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) ジョブを実行し、CSV ファイルを使用してテーブル `test_tbl4` を更新します。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:1" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns:id,name,job2" \
        -T my_data4.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl4/_stream_load
    ```

4. 更新されたテーブルをクエリします。最初のデータ行は既にテーブル `test_tbl4` に存在しますし、`job1` カラムの値は変更されません。2番目のデータ行は新しく挿入され、`job1` カラムのデフォルト値が指定されていないため、部分更新フレームワークがこの列の値を直接 `0` に設定します。

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

### `AUTO_INCREMENT` カラムがプライマリキーでない場合

`AUTO_INCREMENT` カラムがプライマリキーまたはプライマリキーの一部でない場合、および自動増分IDが[Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) ジョブで提供されない場合、次の状況が発生します。

- 表に行がすでに存在する場合、StarRocks は自動増分IDを更新しません。
- 表に新しくロードされた行の場合、StarRocks は新しい自動増分IDを生成します。

この機能は、一意の STRING 値を迅速に計算するための辞書テーブルを作成するために使用できます。

1. データベース `example_db` にテーブル `test_tbl5` を作成し、列 `job1` を `AUTO_INCREMENT` カラムとして指定し、1つのデータ行をテーブル `test_tbl5` に挿入します。

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
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |   -1 |   -1 |
    +------+------+------+------+
    1 row in set (0.01 sec)
    ```

2. 表 `test_tbl5` を更新するための CSV ファイル **my_data5.csv** を準備します。CSV ファイルは `AUTO_INCREMENT` カラム `job1` の値を含まず、最初の行のプライマリキーは既にテーブル `test_tbl5` に存在しますが、2番目と3番目の行のプライマリキーは存在しません。

    ```Plaintext
    0,0,99
    1,1,99
    2,2,99
    ```

3. [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) ジョブを実行し、CSV ファイルからテーブル `test_tbl5` にデータをロードします。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:2" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns: id,name,job2" \
        -T my_data5.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
    ```

4. 更新されたテーブルをクエリします。最初のデータ行は既にテーブル `test_tbl5` に存在するため、`AUTO_INCREMENT` カラム `job1` は元の値を保持します。2番目と3番目のデータ行は新しく挿入されるため、StarRocks は `AUTO_INCREMENT` カラム `job1` のために新しい値を生成します。

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

## 制限

- `AUTO_INCREMENT` カラムを持つテーブルを作成する場合、「'replicated_storage' = 'true'」を設定して、すべてのレプリカが同じ自動増分IDを持つようにする必要があります。
- 各テーブルには、`AUTO_INCREMENT` カラムは1つだけ持つことができます。
- `AUTO_INCREMENT` カラムのデータ型は BIGINT でなければなりません。
- `AUTO_INCREMENT` カラムは `NOT NULL` であり、デフォルト値を持っていてはいけません。
- `AUTO_INCREMENT` カラムを持つプライマリキー テーブルからデータを削除することができます。ただし、`AUTO_INCREMENT` カラムがプライマリキーまたはプライマリキーの一部でない場合、次のシナリオでデータを削除する際の制限に注意する必要があります:
```
- DELETE 操作中には、部分的な更新のためのロードジョブも実行されます。このロードジョブには UPSERT 操作だけが含まれます。DELETE と UPSERT の両方が同じデータ行に当たり、DELETE 操作の後に UPSERT 操作が実行された場合、UPSERT 操作が効果を及ぼさない可能性があります。
- データ行に対する部分的な更新のためのロードジョブがあります。このロードジョブには、同じデータ行に対する複数の UPSERT および DELETE 操作が含まれています。ある UPSERT 操作が DELETE 操作の後に実行された場合、UPSERT 操作が効果を及ぼさない可能性があります。

- ALTER TABLE を使用して AUTO_INCREMENT 属性を追加することはサポートされていません。
- バージョン 3.1 以降、StarRocks の共有データモードは AUTO_INCREMENT 属性をサポートしています。
- StarRocks では AUTO_INCREMENT カラムの開始値とステップサイズを指定することはサポートされていません。

## キーワード

AUTO_INCREMENT, AUTO INCREMENT