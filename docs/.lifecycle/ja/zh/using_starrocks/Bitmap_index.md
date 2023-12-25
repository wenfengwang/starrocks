---
displayed_sidebar: Chinese
---

# Bitmap インデックス

本文では、Bitmap（ビットマップ）インデックスの作成と管理方法、およびBitmapインデックスの使用例について説明します。

Bitmapインデックスは、bitmapを使用する特殊なデータベースインデックスです。bitmapとは、ビット配列のことで、1ビットの値は0または1の2つの値を取ります。各ビットはデータテーブルの1行に対応し、その行の値に基づいてビットの値が0か1かを決定します。

Bitmapインデックスは、特定の列のクエリ効率を向上させることができます。クエリ条件がプレフィックスインデックス列にヒットする場合、StarRocksは[プレフィックスインデックス](../table_design/Sort_key.md)を使用してクエリ効率を向上させ、クエリ結果を迅速に返すことができます。しかし、プレフィックスインデックスの長さには限界があり、プレフィックスインデックス列以外の列のクエリ効率を向上させたい場合は、その列にBitmapインデックスを作成することができます。

## 优势

- 列の基数が低く、値が大量に重複している場合（例：ENUM型の列）は、Bitmapインデックスを使用することでクエリの応答時間を短縮できます。列の基数が高い場合は、[Bloom filter インデックス](../using_starrocks/Bloomfilter_index.md)の使用を推奨します。
- Bitmapインデックスが占めるストレージスペースは通常、インデックスデータのごく一部に過ぎず、他のインデックス技術と比較して、よりストレージスペースを節約できます。
- 複数の列にBitmapインデックスを作成することをサポートし、複数列のクエリ効率を向上させることができます。詳細は[複数列クエリ](#複数列クエリ)を参照してください。

## 使用説明

- Bitmapインデックスは、等価条件（`=`）のクエリや[NOT] IN範囲クエリに使用できる列に適しています。
- 主キーモデルと明細モデルのすべての列でBitmapインデックスを作成できます。集約モデルと更新モデルでは、次元列（つまりKey列）のみがBitmapインデックスの作成をサポートしています。
- FLOAT、DOUBLE、BOOLEAN、DECIMAL型の列にはBitmapインデックスを作成できません。
- クエリがBitmapインデックスを利用したかどうかを知りたい場合は、クエリのProfileの`BitmapIndexFilterRows`フィールドを確認できます。Profileの確認方法については、[クエリ分析](../administration/Query_planning.md#プロファイルの確認)を参照してください。

## インデックスの作成

- テーブル作成時にBitmapインデックスを作成します。

    ```SQL
    CREATE TABLE d0.table_hash
    (
        k1 TINYINT,
        k2 DECIMAL(10, 2) DEFAULT "10.5",
        v1 CHAR(10) REPLACE,
        v2 INT SUM,
        INDEX index_name (column_name) [USING BITMAP] [COMMENT '']
    )
    ENGINE = olap
    AGGREGATE KEY(k1, k2)
    DISTRIBUTED BY HASH(k1)
    PROPERTIES ("storage_type" = "column");
    ```

    インデックスに関するパラメータの説明は以下の通りです：

    | **パラメータ** | **必須** | **説明**                                                     |
    | -------------- | -------- | ------------------------------------------------------------ |
    | index_name     | はい     | Bitmapインデックスの名前。英字(a-zまたはA-Z)、数字(0-9)、アンダースコア(_)で構成され、英字で始まる必要があります。全長は64文字を超えることはできません。同一テーブル内で同名のインデックスを作成することはできません。 |
    | column_name    | はい     | Bitmapインデックスを作成する列名。複数の列名を指定し、テーブル作成時に複数の列に対してBitmapインデックスを同時に作成することができます。 |
    | COMMENT        | いいえ   | インデックスのコメント。                                     |

    複数の`INDEX index_name (column_name) [USING BITMAP] [COMMENT '']`コマンドを指定して、複数の列にBitmapインデックスを同時に作成できます。コマンド間はカンマ（,）で区切ります。
    テーブル作成の他のパラメータについては、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

- テーブル作成後にCREATE INDEXを使用してBitmapインデックスを作成します。詳細なパラメータ説明と例については、[CREATE INDEX](../sql-reference/sql-statements/data-definition/CREATE_INDEX.md)を参照してください。

    ```SQL
    CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT ''];
    ```

## インデックスの作成進行状況

Bitmapインデックスの作成は**非同期**プロセスであり、インデックス作成コマンドを実行した後、[SHOW ALTER TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_ALTER.md)コマンドでインデックスの作成進行状況を確認できます。戻り値の`State`フィールドが`FINISHED`と表示された場合、作成が成功したことを意味します。

```SQL
SHOW ALTER TABLE COLUMN [FROM db_name];
```

> 説明：現在、各テーブルは同時に1つのSchema Changeタスクのみを許可しています。一つのBitmapインデックスが非同期に作成されている間、新しいBitmapインデックスの作成はできません。

## インデックスの確認

指定されたテーブルのすべてのBitmapインデックスを確認します。詳細なパラメータと戻り値の説明については、[SHOW INDEX](../sql-reference/sql-statements/Administration/SHOW_INDEX.md)を参照してください。

```SQL
SHOW { INDEX[ES] | KEY[S] } FROM [db_name.]table_name [FROM db_name];
```

> **説明**
>
> Bitmapインデックスの作成は非同期プロセスであり、上記のステートメントを使用しても、作成が完了したインデックスのみを確認できます。

## インデックスの削除

指定されたテーブルのBitmapインデックスを削除します。詳細なパラメータ説明と例については、[DROP INDEX](../sql-reference/sql-statements/data-definition/DROP_INDEX.md)を参照してください。

```SQL
DROP INDEX index_name ON [db_name.]table_name;
```

## 使用例

たとえば、`employee`という表があり、ある会社の従業員情報を含んでいます。以下は、`employee`表の一部のデータを示しています。

| **ID** | **Gender** | **Position** | **Income_level** |
| ------ | ---------- | ------------ | ---------------- |
| 01     | female     | Developer    | level_1          |
| 02     | female     | Analyst      | level_2          |
| 03     | female     | Salesman     | level_1          |
| 04     | male       | Accountant   | level_3          |

### **単一列クエリ**

たとえば、`Gender`列のクエリ効率を向上させるために、その列にBitmapインデックスを作成することができます。

```SQL
CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
```

上記のステートメントを実行すると、Bitmapインデックスの生成プロセスは以下の通りです：

![図](../assets/3.6.1-2.png)

1. 辞書の構築：StarRocksは`Gender`列の値に基づいて辞書を構築し、`female`と`male`をそれぞれINT型のコード値`0`と`1`にマッピングします。
2. Bitmapの生成：StarRocksは辞書のコード値に基づいてBitmapを生成します。`female`が最初の3行に出現するため、`female`のBitmapは`1110`です。`male`が4行目に出現するため、`male`のBitmapは`0001`です。

同社の男性従業員を検索する場合は、以下のステートメントを実行します。

```SQL
SELECT * FROM employee WHERE Gender = male;
```

ステートメントを実行すると、StarRocksはまず辞書を検索して`male`のコード値が`1`であることを見つけ、次にBitmapを検索して`male`のBitmapが`0001`であることを見つけます。つまり、4行目のデータのみがクエリ条件に一致します。そのため、StarRocksは最初の3行をスキップし、4行目のデータのみを読み取ります。

### **複数列クエリ**

たとえば、`Gender`と`Income_level`の列のクエリ効率を向上させるために、これらの列にそれぞれBitmapインデックスを作成することができます。

- `Gender`

    ```SQL
    CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
    ```

- `Income_level`

    ```SQL
    CREATE INDEX index2 ON employee (Income_level) USING BITMAP COMMENT 'index2';
    ```

上記の2つのステートメントを実行すると、Bitmapインデックスの生成プロセスは以下の通りです：

![図](../assets/3.6.1-3.png)

StarRocksは`Gender`と`Income_level`の列にそれぞれ辞書を構築し、次に辞書に基づいてBitmapを生成します。

- `Gender`列：`female`のBitmapは`1110`、`male`のBitmapは`0001`です。
- `Income_level`列：`level_1`のBitmapは`1010`、`level_2`のBitmapは`0100`、`level_3`のBitmapは`0001`です。

`level_1`の収入レベルを持つ女性従業員を検索する場合は、以下のステートメントを実行します。

```SQL
 SELECT * FROM employee 
 WHERE Gender = female AND Income_level = level_1;
```

ステートメントを実行すると、StarRocksは`Gender`と`Income_level`の列の辞書を同時に検索し、以下の情報を得ます：

- `female`のコード値は`0`で、対応するBitmapは`1110`です。
- `level_1`のコード値は`0`で、対応するBitmapは`1010`です。

クエリステートメントにおける`Gender = female`と`Income_level = Level_1`の2つの条件は`AND`関係であるため、StarRocksは2つのBitmapに対してビット演算`1110 & 1010`を行い、最終結果`1010`を得ます。最終結果に基づき、StarRocksは1行目と3行目のデータのみを読み取り、すべてのデータを読み取ることはありません。
