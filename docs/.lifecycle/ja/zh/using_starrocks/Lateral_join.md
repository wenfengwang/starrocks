---
displayed_sidebar: Chinese
---

# Lateral Join を使用して列を行に変換する

この記事では、Lateral Join 機能の使用方法について説明します。

「行列変換」は、ETL 処理の過程でよく見られる操作です。Lateral Join 機能は、各行のデータを内部のサブクエリや Table Function と関連付けることができます。Lateral Join と Unnest 機能を組み合わせることで、一行を複数行に変換する機能を実現できます。Unnest は Table Function の一種で、配列型をテーブルの複数行に変換することができます。詳細は [unnest](../sql-reference/sql-functions/array-functions/unnest.md) を参照してください。

> 注意
>
> * 現在のバージョンでは、Lateral Join は Unnest 関数と組み合わせて使用し、列を行に変換する機能にのみ使用されます。将来的には他の Table Function や UDTF と組み合わせて使用することがサポートされる予定です。
> * 現在のバージョンでは、Lateral Join はサブクエリをサポートしていません。

## Lateral Join の使用

Lateral Join 機能の構文は以下の通りです：

~~~SQL
from table_reference join [lateral] table_reference;
~~~

Lateral Join 機能と Unnest 機能を組み合わせて、一般的な行展開ロジックを実現できます。

~~~SQL
-- 完全な SQL ステートメント。
SELECT student, score, t.unnest
FROM tests
CROSS JOIN LATERAL UNNEST(scores) AS t;

-- 簡略化された SQL ステートメント。Unnest キーワードを使用して Lateral Join キーワードを省略できます。
SELECT student, score, t.unnest
FROM tests, UNNEST(scores) AS t;
~~~

> 注意
>
> 複数列の Unnest 操作ではエイリアスを指定する必要があります。例：`select v1, t1.unnest as v2, t2.unnest as v3 from lateral_test, unnest(v2) t1, unnest(v3) t2;`。

StarRocks がサポートする BITMAP、STRING、ARRAY、Column 間の型変換関係は以下の通りです。

![Lateral Join でのデータ型間の変換](../assets/lateral_join_type_conversion.png)

### STRING 型データの展開

Lateral Join 機能と Unnest 機能を組み合わせて、STRING 型データを複数行のデータに展開できます。

例：

1. テスト用のテーブルを作成し、テストデータを挿入します。

    ~~~SQL
    CREATE TABLE lateral_test2 (
        `v1` bigint(20) NULL COMMENT "",
        `v2` string NULL COMMENT "",
        `v3` string NULL COMMENT ""
    )
    duplicate key(v1)
    DISTRIBUTED BY HASH(`v1`)
    PROPERTIES (
        "replication_num" = "3",
        "storage_format" = "DEFAULT"
    );

    insert into lateral_test2 values (1, "1,2,3","1,2"), (2, "1,3","1,3");
    ~~~

2. 展開前のデータを確認します。

    ~~~Plain Text
    mysql> select * from lateral_test2;

    +------+-------+------+
    | v1   | v2    | v3   |
    +------+-------+------+
    |    1 | 1,2,3 | 1,2  |
    |    2 | 1,3   | 1,3  |
    +------+-------+------+
    ~~~

3. STRING データを展開します。

    ~~~Plain Text
    -- 単一行データに対する Unnest 操作。
    mysql> select v1, unnest from lateral_test2, unnest(split(v2, ","));

    +------+--------+
    | v1   | unnest |
    +------+--------+
    |    1 | 1      |
    |    1 | 2      |
    |    1 | 3      |
    |    2 | 1      |
    |    2 | 3      |
    +------+--------+

    -- 複数行データに対する Unnest 操作ではエイリアスを指定する必要があります。
    mysql> select v1, t1.unnest as v2, t2.unnest as v3 from lateral_test2, unnest(split(v2, ",")) t1, unnest(split(v3, ",")) t2;

    +------+------+------+
    | v1   | v2   | v3   |
    +------+------+------+
    |    1 | 1    | 1    |
    |    1 | 1    | 2    |
    |    1 | 2    | 1    |
    |    1 | 2    | 2    |
    |    1 | 3    | 1    |
    |    1 | 3    | 2    |
    |    2 | 1    | 1    |
    |    2 | 1    | 3    |
    |    2 | 3    | 1    |
    |    2 | 3    | 3    |
    +------+------+------+
    ~~~

### ARRAY 型データの展開

Lateral Join 機能と Unnest 機能を組み合わせて、ARRAY 型データを複数行のデータに展開できます。**バージョン 2.5 から、Unnest は複数の array を受け入れることができ、それぞれの array の要素の型や長さが異なっていても構いません。**

例：

1. テスト用のテーブルを作成し、テストデータを挿入します。

    ~~~SQL
    CREATE TABLE lateral_test (
        `v1` bigint(20) NULL COMMENT "",
        `v2` ARRAY<int> NULL COMMENT ""
    ) 
    duplicate key(v1)
    DISTRIBUTED BY HASH(`v1`)
    PROPERTIES (
        "replication_num" = "3",
        "storage_format" = "DEFAULT"
    );

    insert into lateral_test values (1, [1,2]), (2, [1, null, 3]), (3, null);
    ~~~

2. 展開前のデータを確認します。

    ~~~Plain Text
    mysql> select * from lateral_test;

    +------+------------+
    | v1   | v2         |
    +------+------------+
    |    1 | [1,2]      |
    |    2 | [1,null,3] |
    |    3 | NULL       |
    +------+------------+
    ~~~

3. ARRAY データを展開します。

    ~~~Plain Text
    mysql> select v1, v2, unnest from lateral_test, unnest(v2);

    +------+------------+--------+
    | v1   | v2         | unnest |
    +------+------------+--------+
    |    1 | [1,2]      |      1 |
    |    1 | [1,2]      |      2 |
    |    2 | [1,null,3] |      1 |
    |    2 | [1,null,3] |   NULL |
    |    2 | [1,null,3] |      3 |
    +------+------------+--------+
    ~~~

### Bitmap 型データの展開

Lateral Join 機能と Unnest 機能を組み合わせて、Bitmap 型データを展開できます。

例：

1. テスト用のテーブルを作成し、テストデータを挿入します。

    ~~~SQL
    CREATE TABLE lateral_test3 (
    `v1` bigint(20) NULL COMMENT "",
    `v2` Bitmap BITMAP_UNION COMMENT ""
    )
    Aggregate key(v1)
    DISTRIBUTED BY HASH(`v1`);

    insert into lateral_test3 values (1, bitmap_from_string('1, 2')), (2, to_bitmap(3));
    ~~~

2. 現在のデータで `v1` と `bitmap_to_string(v2)` を確認します。

    ~~~Plain Text
    mysql> select v1, bitmap_to_string(v2) from lateral_test3;

    +------+------------------------+
    | v1   | bitmap_to_string(`v2`) |
    +------+------------------------+
    |    1 | 1,2                    |
    |    2 | 3                      |
    +------+------------------------+
    ~~~

3. 新しい行のデータを挿入します。

    ~~~Plain Text
    mysql> insert into lateral_test3 values (1, to_bitmap(3));
    ~~~

4. 新しいデータで `v1` と `bitmap_to_string(v2)` を確認します。

    ~~~Plain Text
    mysql> select v1, bitmap_to_string(v2) from lateral_test3;

    +------+------------------------+
    | v1   | bitmap_to_string(`v2`) |
    +------+------------------------+
    |    1 | 1,2,3                  |
    |    2 | 3                      |
    +------+------------------------+
    ~~~

5. Bitmap 型データを展開します。

    ~~~Plain Text
    mysql> select v1, unnest from lateral_test3, unnest(bitmap_to_array(v2));

    +------+--------+
    | v1   | unnest |
    +------+--------+
    |    1 |      1 |
    |    1 |      2 |
    |    1 |      3 |
    |    2 |      3 |
    +------+--------+
    ~~~

## キーワード

explode, 爆裂関数
