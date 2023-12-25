---
displayed_sidebar: English
---

# AUTO_INCREMENT

バージョン3.0以降、StarRocksは `AUTO_INCREMENT` カラム属性をサポートしており、データ管理を簡素化することができます。このトピックでは、`AUTO_INCREMENT` カラム属性のアプリケーションシナリオ、使用法、および特徴について紹介します。

## 紹介

新しいデータ行がテーブルにロードされ、`AUTO_INCREMENT` カラムに値が指定されていない場合、StarRocksはその行の `AUTO_INCREMENT` カラムに整数値を自動的に割り当て、テーブル全体で一意のIDとして機能させます。`AUTO_INCREMENT` カラムの後続の値は、行のIDから始まる特定のステップで自動的に増加します。`AUTO_INCREMENT` カラムはデータ管理を簡略化し、一部のクエリを高速化するために使用できます。以下は `AUTO_INCREMENT` カラムのアプリケーションシナリオです：

- 主キーとして機能：`AUTO_INCREMENT` カラムは主キーとして使用でき、各行に一意のIDを割り当て、データのクエリと管理を容易にします。
- テーブル結合：複数のテーブルを結合する際に、`AUTO_INCREMENT` カラムを結合キーとして使用すると、例えばUUIDのようなSTRING型のカラムを使用する場合に比べてクエリが迅速になります。
- 高カーディナリティカラムの異なる値の数をカウント：`AUTO_INCREMENT` カラムはディクショナリ内の一意の値カラムを表すために使用できます。直接STRING値をカウントするよりも、`AUTO_INCREMENT` カラムの異なる整数値をカウントする方が、クエリ速度が数倍から数十倍向上することがあります。

`AUTO_INCREMENT` カラムはCREATE TABLEステートメントで指定する必要があります。`AUTO_INCREMENT` カラムのデータ型はBIGINTでなければなりません。AUTO_INCREMENTカラムの値は、[暗黙的に割り当てられるか、明示的に指定される](#assign-values-for-auto_increment-column)ことができます。これは1から始まり、新しい行ごとに1ずつ増加します。

## 基本操作

### テーブル作成時に `AUTO_INCREMENT` カラムを指定

`id` と `number` の2つのカラムを持つ `test_tbl1` という名前のテーブルを作成し、`number` カラムを `AUTO_INCREMENT` カラムとして指定します。

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

### `AUTO_INCREMENT` カラムに値を割り当てる

#### 値を暗黙的に割り当てる

StarRocksテーブルにデータをロードする際、`AUTO_INCREMENT` カラムの値を指定する必要はありません。StarRocksは自動的にそのカラムに一意の整数値を割り当て、テーブルに挿入します。

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

StarRocksテーブルにデータをロードする際、`AUTO_INCREMENT` カラムの値を `DEFAULT` として指定することもできます。StarRocksは自動的にそのカラムに一意の整数値を割り当て、テーブルに挿入します。

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

実際の使用では、テーブル内のデータを表示すると次の結果が返される場合があります。これはStarRocksが `AUTO_INCREMENT` カラムの値が厳密に単調であることを保証できないためです。しかし、StarRocksは値が時系列順に大まかに増加することを保証できます。詳細については[単調性](#monotonicity)を参照してください。

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

#### 値を明示的に指定する

`AUTO_INCREMENT` カラムの値を明示的に指定し、テーブルに挿入することもできます。

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

さらに、値を明示的に指定しても、新しく挿入されたデータ行に対してStarRocksが生成する後続の値には影響しません。

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

`AUTO_INCREMENT` カラムに対して暗黙的に割り当てられた値と明示的に指定された値を同時に使用しないことを推奨します。指定された値がStarRocksによって生成された値と同じである可能性があり、[自動増分IDのグローバルな一意性](#uniqueness)を損なう可能性があります。

## 基本的な特徴

### 一意性

一般的に、StarRocksは `AUTO_INCREMENT` カラムの値がテーブル全体でグローバルに一意であることを保証します。`AUTO_INCREMENT` カラムの値を暗黙的に割り当て、同時に明示的に指定することは推奨されません。そうすると、自動増分IDのグローバルな一意性が損なわれる可能性があります。簡単な例を挙げると、`id` と `number` の2つのカラムを持つ `test_tbl2` という名前のテーブルを作成し、`number` カラムを `AUTO_INCREMENT` カラムとして指定します。

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

テーブル `test_tbl2` の `AUTO_INCREMENT` カラム `number` に対して値を暗黙的に割り当て、明示的に指定します。

```SQL
INSERT INTO test_tbl2 (id, number) VALUES (1, DEFAULT);
INSERT INTO test_tbl2 (id, number) VALUES (2, 2);
INSERT INTO test_tbl2 (id) VALUES (3);
```

テーブル `test_tbl2` をクエリします。

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

自動増分IDの割り当てパフォーマンスを向上させるために、BEはいくつかの自動増分IDをローカルにキャッシュします。この状況では、StarRocksは `AUTO_INCREMENT` カラムの値が厳密に単調であることを保証できません。時系列順に大まかに増加することのみが保証されます。

> **注記**
>
> BEによってキャッシュされる自動増分IDの数は、FEの動的パラメータ `auto_increment_cache_size` によって決定され、デフォルトは `100,000` です。この値は `ADMIN SET FRONTEND CONFIG ("auto_increment_cache_size" = "xxx");` を使用して変更できます。

例えば、StarRocksクラスタには1つのFEノードと2つのBEノードがあります。`test_tbl3` という名前のテーブルを作成し、次のように5行のデータを挿入します：

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

テーブル`test_tbl3`内の自動インクリメントされたIDは単調に増加しません。これは、2つのBEノードがそれぞれ[1, 100000]と[100001, 200000]の自動インクリメントされたIDをキャッシュしているためです。複数のINSERTステートメントを使用してデータをロードする際、データは異なるBEノードに送信され、それぞれが独立して自動インクリメントされたIDを割り当てます。したがって、自動インクリメントされたIDが厳密に単調であることは保証されません。

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

このセクションでは、`AUTO_INCREMENT`列を含むテーブルの特定の列のみを更新する方法について説明します。

> **注記**
>
> 現在、部分更新をサポートしているのはプライマリキーテーブルのみです。

### `AUTO_INCREMENT`列がプライマリキーの場合

部分更新時にはプライマリキーを指定する必要があります。したがって、`AUTO_INCREMENT`列がプライマリキーまたはプライマリキーの一部である場合、部分更新におけるユーザーの振る舞いは、`AUTO_INCREMENT`列が定義されていない場合と全く同じです。

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

2. テーブル`test_tbl4`を更新するためのCSVファイル**my_data4.csv**を準備します。このCSVファイルには`AUTO_INCREMENT`列の値が含まれており、`job1`列の値は含まれていません。最初の行のプライマリキーは既にテーブル`test_tbl4`に存在しますが、2行目のプライマリキーは存在しません。

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

4. 更新されたテーブルをクエリします。最初の行のデータは既にテーブル`test_tbl4`に存在し、`job1`列の値は変更されていません。2行目のデータは新たに挿入され、`job1`列のデフォルト値が指定されていないため、部分更新フレームワークはこの列の値を直接`0`に設定します。

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

`AUTO_INCREMENT`列がプライマリキーでもプライマリキーの一部でもなく、Stream Loadジョブで自動インクリメントされたIDが提供されない場合、以下の状況が発生します：

- 行が既にテーブルに存在する場合、StarRocksは自動インクリメントされたIDを更新しません。
- 行がテーブルに新たにロードされる場合、StarRocksは新しい自動インクリメントされたIDを生成します。

この機能は、STRING値の高速な重複排除を行う辞書テーブルを構築するために使用できます。

1. データベース`example_db`で、`job1`列を`AUTO_INCREMENT`列として指定し、テーブル`test_tbl5`を作成し、データ行を挿入します。

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

2. テーブル`test_tbl5`を更新するためのCSVファイル**my_data5.csv**を準備します。このCSVファイルには`AUTO_INCREMENT`列`job1`の値が含まれていません。最初の行のプライマリキーは既にテーブルに存在しますが、2行目と3行目のプライマリキーは存在しません。

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

4. 更新されたテーブルをクエリします。最初の行のデータは既にテーブル`test_tbl5`に存在するため、`AUTO_INCREMENT`列`job1`は元の値を保持します。2行目と3行目のデータは新たに挿入されるため、StarRocksは`AUTO_INCREMENT`列`job1`の新しい値を生成します。

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

- `AUTO_INCREMENT`列を含むテーブルを作成する場合、`'replicated_storage' = 'true'`を設定して、すべてのレプリカが同じ自動インクリメントIDを持つようにする必要があります。
- 各テーブルには1つの`AUTO_INCREMENT`列のみを含めることができます。
- `AUTO_INCREMENT`列のデータ型はBIGINTでなければなりません。
- `AUTO_INCREMENT`列は`NOT NULL`であり、デフォルト値を持ってはいけません。
- `AUTO_INCREMENT`列を持つプライマリキーテーブルからデータを削除することができます。ただし、`AUTO_INCREMENT`列がプライマリキーでもプライマリキーの一部でもない場合、以下のシナリオでデータを削除する際には以下の制限に注意する必要があります：

  - DELETE操作中に、UPSERT操作のみを含む部分更新のロードジョブが同時に行われている場合、UPSERT操作とDELETE操作が同じデータ行にヒットし、UPSERT操作がDELETE操作の後に実行されると、UPSERT操作が効果を発揮しない可能性があります。
  - 部分更新のロードジョブに、同じデータ行に対する複数のUPSERT操作とDELETE操作が含まれている場合、DELETE操作の後に実行されるUPSERT操作が効果を発揮しない可能性があります。

- ALTER TABLEを使用して`AUTO_INCREMENT`属性を追加することはサポートされていません。
- バージョン 3.1 以降、StarRocks の共有データモードは `AUTO_INCREMENT` 属性をサポートしています。
- StarRocks は `AUTO_INCREMENT` 列の開始値とステップサイズを指定することをサポートしていません。

## キーワード

AUTO_INCREMENT, AUTO_INCREMENT
