---
displayed_sidebar: English
---

# カラムから行への変換にLateral Joinを使用する

カラムから行への変換は、ETL処理で一般的な操作です。Lateralは、行を内部サブクエリやテーブル関数と関連付けることができる特別なJoinキーワードです。Lateralをunnest()と組み合わせて使用することで、1つの行を複数の行に展開できます。詳細は[unnest](../sql-reference/sql-functions/array-functions/unnest.md)を参照してください。

## 制限事項

* 現在、Lateral Joinはunnest()と組み合わせてカラムから行への変換を実現するためにのみ使用されます。他のテーブル関数やUDTFは将来的にサポートされる予定です。
* 現在、Lateral Joinはサブクエリをサポートしていません。

## Lateral Joinの使用

構文：

~~~SQL
from table_reference join [lateral] table_reference;
~~~

例：

~~~SQL
SELECT student, score
FROM tests
CROSS JOIN LATERAL UNNEST(scores) AS t (score);

SELECT student, score
FROM tests, UNNEST(scores) AS t (score);
~~~

ここでの2番目の構文は、最初のものの省略形で、UNNESTキーワードを使用することでLateralキーワードを省略できます。UNNESTキーワードは、配列を複数の行に変換するテーブル関数です。Lateral Joinと組み合わせることで、一般的な行の展開ロジックを実装できます。

> **注記**
>
> 複数のカラムに対してunnestを実行する場合は、それぞれのカラムにエイリアスを指定する必要があります。例えば、`select v1, t1.unnest as v2, t2.unnest as v3 from lateral_test, unnest(v2) t1, unnest(v3) t2;`のようにします。

StarRocksは、BITMAP、STRING、ARRAY、およびColumn間の型変換をサポートしています。
![Lateral Joinにおける型変換の一部](../assets/lateral_join_type_conversion.png)

## 使用例

unnest()と組み合わせることで、以下のカラムから行への変換機能を実現できます：

### 文字列を複数の行に展開する

1. テーブルを作成し、そのテーブルにデータを挿入します。

    ~~~SQL
    CREATE TABLE lateral_test2 (
        `v1` bigint(20) NULL COMMENT "",
        `v2` string NULL COMMENT ""
    )
    DUPLICATE KEY(v1)
    DISTRIBUTED BY HASH(`v1`)
    PROPERTIES (
        "replication_num" = "3",
        "storage_format" = "DEFAULT"
    );

    INSERT INTO lateral_test2 VALUES (1, "1,2,3"), (2, "1,3");
    ~~~

2. 展開前のデータをクエリします。

    ~~~Plain Text
    select * from lateral_test2;

    +------+-------+
    | v1   | v2    |
    +------+-------+
    |    1 | 1,2,3 |
    |    2 | 1,3   |
    +------+-------+
    ~~~

3. `v2`を複数の行に展開します。

    ~~~Plain Text
    -- 単一カラムに対してunnestを実行します。

    select v1,unnest from lateral_test2, unnest(split(v2, ","));

    +------+--------+
    | v1   | unnest |
    +------+--------+
    |    1 | 1      |
    |    1 | 2      |
    |    1 | 3      |
    |    2 | 1      |
    |    2 | 3      |
    +------+--------+

    -- 複数カラムに対してunnestを実行します。各操作にエイリアスを指定する必要があります。

    select v1, t1.unnest as v2, t2.unnest as v3 from lateral_test2, unnest(split(v2, ",")) t1, unnest(split(v3, ",")) t2;

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

### 配列を複数の行に展開する

 **v2.5から、unnest()は異なる型と長さの複数の配列を取り扱うことができます。** 詳細は[unnest()](../sql-reference/sql-functions/array-functions/unnest.md)を参照してください。

1. テーブルを作成し、そのテーブルにデータを挿入します。

    ~~~SQL
    CREATE TABLE lateral_test (
        `v1` bigint(20) NULL COMMENT "",
        `v2` ARRAY NULL COMMENT ""
    ) 
    DUPLICATE KEY(v1)
    DISTRIBUTED BY HASH(`v1`)
    PROPERTIES (
        "replication_num" = "3",
        "storage_format" = "DEFAULT"
    );

    INSERT INTO lateral_test VALUES (1, [1,2]), (2, [1, null, 3]), (3, null);
    ~~~

2. 展開前のデータをクエリします。

    ~~~Plain Text
    select * from lateral_test;

    +------+------------+
    | v1   | v2         |
    +------+------------+
    |    1 | [1,2]      |
    |    2 | [1,null,3] |
    |    3 | NULL       |
    +------+------------+
    ~~~

3. `v2`を複数の行に展開します。

    ~~~Plain Text
    select v1,v2,unnest from lateral_test, unnest(v2);

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

### Bitmapデータを複数の行に展開する

1. テーブルを作成し、そのテーブルにデータを挿入します。

    ~~~SQL
    CREATE TABLE lateral_test3 (
    `v1` bigint(20) NULL COMMENT "",
    `v2` Bitmap BITMAP_UNION COMMENT ""
    )
    AGGREGATE KEY(v1)
    DISTRIBUTED BY HASH(`v1`);

    INSERT INTO lateral_test3 VALUES (1, bitmap_from_string('1, 2')), (2, to_bitmap(3));
    ~~~

2. 展開前のデータをクエリします。

    ~~~Plain Text
    select v1, bitmap_to_string(v2) from lateral_test3;

    +------+------------------------+
    | v1   | bitmap_to_string(`v2`) |
    +------+------------------------+
    |    1 | 1,2                    |
    |    2 | 3                      |
    +------+------------------------+

3. 新しい行を挿入します。

    ~~~Plain Text
    insert into lateral_test3 values (1, to_bitmap(3));

    select v1, bitmap_to_string(v2) from lateral_test3;

    +------+------------------------+
    | v1   | bitmap_to_string(`v2`) |
    +------+------------------------+
    |    1 | 1,2,3                  |
    |    2 | 3                      |
    +------+------------------------+
    ~~~

4. `v2`を複数の行に展開します。

    ~~~Plain Text
    select v1,unnest from lateral_test3, unnest(bitmap_to_array(v2));

    +------+--------+
    | v1   | unnest |
    +------+--------+
    |    1 |      1 |
    |    1 |      2 |
    |    1 |      3 |
    |    2 |      3 |
    +------+--------+
    ~~~
