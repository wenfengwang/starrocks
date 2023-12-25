---
displayed_sidebar: Chinese
---

# AUTO_INCREMENT

StarRocksはバージョン3.0から`AUTO_INCREMENT`列属性をサポートしており、データ管理を簡素化できます。この記事では、`AUTO_INCREMENT`列属性の使用シナリオ、使用方法、および特性について説明します。

## 機能紹介

新しいレコードを挿入する際、StarRocksはそのレコードの自動増分列に対して、テーブル内でグローバルに一意な整数値を自動的に割り当て、自動増分IDとして使用し、その後の値は自動的に増加します。自動増分列はデータ管理を簡素化するだけでなく、いくつかのクエリシナリオを高速化することもできます。以下は自動増分列の使用シナリオの例です：

- 主キー：自動増分列は主キーの生成に使用でき、各レコードに一意の識別子を保証し、データのクエリと管理を容易にします。
- 関連テーブル：複数のテーブル間で関連付けを行う際、自動増分列をJoin Keyとして使用することで、UUIDなどの文字列型の列を使用するよりもクエリ速度を向上させることができます。
- 高基数列の正確な重複排除カウント：自動増分列のID値を辞書の一意な値の列として使用することで、文字列を直接使用して正確な重複排除カウントを行うよりも、クエリ速度が数倍から数十倍向上します。

`AUTO_INCREMENT`属性を使用してCREATE TABLEステートメントで自動増分列を指定する必要があります。自動増分列のデータ型はBIGINTのみをサポートし、1から始まり、増分ステップは1です。また、StarRocksは[自動増分列の値の暗黙的な割り当てと明示的な自動増分IDの指定](#分配自增列的值)をサポートしています。

## 基本的な使い方

### テーブル作成時に自動増分列を指定

`test_tbl1`というテーブルを作成し、`id`と`number`の2つの列を含むようにします。以下に示すように、テーブル作成時に`number`列を自動増分列として指定します：

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

### 自動増分列の値の割り当て

**自動増分列の値の暗黙的な割り当て**

インポート時に自動増分列の値を指定する必要はありません。StarRocksは自動的にその自動増分列に一意の整数値を割り当て、テーブルに挿入します。

```SQL
INSERT INTO test_tbl1 (id) VALUES (1);
INSERT INTO test_tbl1 (id) VALUES (2);
INSERT INTO test_tbl1 (id) VALUES (3),(4),(5);
```

テーブルのデータを確認します。

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

また、自動増分列の値を`DEFAULT`として指定することもできます。StarRocksは自動的にその自動増分列に一意の整数値を割り当て、テーブルに挿入します。

```SQL
INSERT INTO test_tbl1 (id, number) VALUES (6, DEFAULT);
```

テーブルのデータを確認します。

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

実際の使用では、テーブルのデータを確認すると以下のような結果が返されることがあります。これは、StarRocksが自動増分列の値を時間順に厳密に増加させることができないためですが、自動増分列の値がおおよそ増加していることは保証できます。詳細については、[単調性の保証](#单调性保证)を参照してください。

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

**自動増分列の値の明示的な指定**

自動増分列の値を明示的に指定し、テーブルに挿入することもできます。

```SQL
INSERT INTO test_tbl1 (id, number) VALUES (7, 100);

-- テーブルのデータを確認
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

さらに、新しいデータを挿入しても、StarRocksが新しく生成する自動増分列の値には影響しません。

```SQL
INSERT INTO test_tbl1 (id) VALUES (8);

-- テーブルのデータを確認
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

**注意事項**

自動増分IDの暗黙的な割り当てと明示的な指定を同時に行うと、自動増分IDの[グローバル一意性](#唯一性保证)が損なわれる可能性があるため、混在させないことをお勧めします。

## 基本特性

### グローバル一意性の保証

通常、StarRocksはテーブル内で自動増分IDがグローバルに一意であることを保証します。

しかし、自動増分IDの暗黙的な割り当てと明示的な指定を混在させると、自動増分IDのグローバル一意性が損なわれる可能性があります。そのため、暗黙的な割り当てと明示的な指定を同時に行わないことをお勧めします。以下は簡単な例です：

`test_tbl2`というテーブルを作成し、`number`列を自動増分列としています。

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

`test_tbl2`テーブルに対して、自動増分IDを暗黙的に割り当てつつ、明示的に指定も行います。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

`test_tbl2`テーブルのデータをクエリします。

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

### 単調性の保証

自動増分IDの割り当てパフォーマンスを向上させるために、BEは一部の自動増分IDをローカルにキャッシュします。この状況では、StarRocksは自動増分IDが時間順に厳密に増加することを保証できませんが、自動増分IDがおおよそ増加していることは保証できます。
> **説明**
>
> BEがキャッシュする自動増分IDの数は、FEの動的パラメータ`auto_increment_cache_size`によって決定され、デフォルトは`100000`です。`ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");`を使用して変更することができます。
StarRocksクラスターに1つのFEノードと2つのBEノードがあると仮定します。`test_tbl3`というテーブルを作成し、以下のように5行のデータを挿入します：

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

`test_tbl3` テーブルの自動増分 ID は単調増加ではありません。これは、2つの BE ノードがそれぞれ [1, 100000] と [100001, 200000] の範囲の自動増分 ID をキャッシュしており、複数の INSERT 文を使用してデータをインポートする際に異なる BE に送信され、異なる BE によって自動増分 ID が割り当てられるため、自動増分 ID の厳密な単調性を保証することはできません。

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

## 部分列の更新と自動増分列

このセクションでは、自動増分列を持つテーブルで部分列の更新を行う方法について説明します。

> **説明**
>
> 現在、主キーモデルのテーブルのみが部分列の更新をサポートしています。

### 自動増分列が主キーの場合

自動増分列が主キーまたは主キーの一部である場合、部分列の更新では主キーを指定する必要があるため、自動増分列が定義されていない場合と同じユーザー操作になります。

1. `example_db` データベースに `test_tbl4` テーブルを作成し、データを1行挿入します。

    ```SQL
    -- テーブル作成
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

    -- データ準備
    mysql > INSERT INTO test_tbl4 (id, name, job1, job2) VALUES (0, 0, 1, 1);
    Query OK, 1 row affected (0.04 sec)
    {'label':'insert_6af28e77-7d2b-11ed-af6e-02424283676b', 'status':'VISIBLE', 'txnId':'152'}

    -- データ確認
    mysql > SELECT * FROM test_tbl4 ORDER BY id;
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |    1 |    1 |
    +------+------+------+------+
    1 row in set (0.01 sec)
    ```

2. `test_tbl4` テーブルを更新するための CSV ファイル **my_data4.csv** を準備します。CSV ファイルには自動増分列の ID 値が含まれており、`job1` の値は含まれていません。また、最初の行の主キーは `test_tbl4` テーブルに存在し、2行目の主キーは存在しません。

    ```Plaintext
    0,0,99
    1,1,99
    ```

3. [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を使用して CSV ファイルのデータを `test_tbl4` テーブルに更新します。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:1" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns:id,name,job2" \
        -T my_data4.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl4/_stream_load
    ```

4. 更新後のテーブルを確認します。最初の行のデータは元々 `test_tbl4` テーブルに存在しており、`job1` 列は元の値を保持しています。2行目のデータは新しく挿入されたデータであり、`job1` 列にデフォルト値が定義されていないため、部分列の更新フレームワークはこの列の値を `0` に設定します。

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

### 自動増分列が主キーでない場合

自動増分列が主キーまたは主キーの一部ではなく、Stream Load で自動増分 ID が提供されていない場合、以下の状況が発生します：

- テーブルにその行が既に存在する場合、StarRocks は自動増分 ID を更新しません。
- テーブルにその行が存在しない場合、StarRocks は新しい自動増分 ID を自動生成します。

この機能は、文字列の正確な重複カウントを高速化するための辞書テーブルの値を構築するために使用できます。

1. `example_db` データベースに `test_tbl5` テーブルを作成し、`job1` を自動増分列として指定し、データを1行挿入します。

    ```SQL
    -- テーブル作成
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

    -- データ準備
    mysql > INSERT INTO test_tbl5 VALUES (0, 0, -1, -1);
    Query OK, 1 row affected (0.04 sec)
    {'label':'insert_458d9487-80f6-11ed-ae56-aa528ccd0ebf', 'status':'VISIBLE', 'txnId':'94'}

    mysql > SELECT * FROM test_tbl5 ORDER BY id;
    +------+------+------+------+
    | id   | name | job1 | job2 |
    +------+------+------+------+
    |    0 |    0 |   -1 |   -1 |
    +------+------+------+------+
    1 row in set (0.01 sec)
    ```

2. `test_tbl5` テーブルを更新するための CSV ファイル **my_data5.csv** を準備します。CSV ファイルには自動増分列 `job1` の値が含まれておらず、最初の行の主キーはテーブルに存在し、2行目と3行目の主キーは存在しません。

    ```Plaintext
    0,0,99
    1,1,99
    2,2,99
    ```

3. [Stream Load](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) を使用して CSV ファイルのデータを `test_tbl5` テーブルにインポートします。

    ```Bash
    curl --location-trusted -u <username>:<password> -H "label:2" \
        -H "column_separator:," \
        -H "partial_update:true" \
        -H "columns: id,name,job2" \
        -T my_data5.csv -XPUT \
        http://<fe_host>:<fe_http_port>/api/example_db/test_tbl5/_stream_load
    ```

4. 更新後のテーブルを確認します。最初の行のデータは `test_tbl5` テーブルに既に存在しており、自動増分列 `job1` は元の ID 値を保持しています。2行目と3行目のデータは新しく挿入されたデータであり、自動増分列 `job1` の ID 値は StarRocks によって自動生成されます。

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

## 使用制限

- 自動増分列を持つテーブルを作成する際には、`'replicated_storage' = 'true'` を設定して、すべてのレプリカが同じ自動増分 ID を持つことを確実にする必要があります。
- 各テーブルには最大1つの自動増分列を設定できます。
- 自動増分列は BIGINT 型でなければなりません。
- 自動増分列は `NOT NULL` でなければならず、デフォルト値を指定することはできません。
- 主キーモデルの自動増分列を持つテーブルからデータを削除することができます。ただし、自動増分列が Primary Key でない場合、データを削除する際に以下の2つのシナリオにおける制限に注意する必要があります：
  - DELETE 操作が行われている間に、部分列の更新のインポートタスクが存在し、その中に UPSERT 操作のみが含まれている場合。UPSERT 操作と DELETE 操作が同じ行のデータにヒットし、かつ UPSERT 操作が DELETE 操作の後に実行された場合、その UPSERT 操作は無効になる可能性があります。
  - 部分的に列を更新するインポートタスクが存在し、同一行に対する複数の UPSERT、DELETE 操作が含まれています。DELETE 操作の後に UPSERT 操作が行われた場合、その UPSERT 操作は無効になる可能性があります。
- ALTER TABLE を使用して `AUTO_INCREMENT` 属性を追加することはサポートされていません。
- バージョン 3.1 以降、ストレージと計算の分離モードはこの機能をサポートしています。
- 自動増分列の開始値と増分値を設定することはサポートされていません。

## キーワード

AUTO_INCREMENT, AUTO_INCREMENT
