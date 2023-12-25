---
displayed_sidebar: English
---

# ビットマップインデックス

このトピックでは、ビットマップインデックスの作成と管理方法、および使用例について説明します。

ビットマップインデックスは、ビットの配列であるビットマップを使用する特殊なデータベースインデックスです。ビットは常に0または1のいずれかの値を取ります。ビットマップ内の各ビットは、テーブル内の単一行に対応しています。各ビットの値は、それに対応する行の値に依存します。

ビットマップインデックスは、特定の列でのクエリパフォーマンスを向上させるのに役立ちます。クエリがソートキーカラムにヒットする場合、StarRocksは[prefix index](../table_design/Sort_key.md)を使用してクエリ結果を効率的に返します。しかし、データブロックのプレフィックスインデックスエントリは36バイトを超えることはできません。ソートキーとして使用されていないカラムでクエリパフォーマンスを向上させたい場合、そのカラムにビットマップインデックスを作成することができます。

## 利点

ビットマップインデックスは以下の面で利点があります：

- カーディナリティが低い列（ENUM型の列など）の応答時間を短縮します。列の異なる値の数が比較的多い場合は、クエリ速度を向上させるためにブルームフィルタインデックスの使用を推奨します。詳細は[Bloom filter indexing](../using_starrocks/Bloomfilter_index.md)を参照してください。
- 他のインデックス技術と比較して、少ないストレージスペースを使用します。ビットマップインデックスは通常、テーブル内のインデックス付きデータのごく一部のサイズしか占めません。
- 複数のビットマップインデックスを組み合わせて、複数の列に対するクエリを実行することができます。詳細は[Query multiple columns](#query-multiple-columns)を参照してください。

## 使用上の注意

- `=`または[NOT] IN演算子を使用してフィルタリング可能な列に対してビットマップインデックスを作成できます。
- Duplicate KeyテーブルやUnique Keyテーブルの全ての列にビットマップインデックスを作成できます。AggregateテーブルやPrimary Keyテーブルでは、キーカラムにのみビットマップインデックスを作成できます。
- FLOAT、DOUBLE、BOOLEAN、DECIMAL型の列はビットマップインデックスの作成をサポートしていません。
- クエリがビットマップインデックスを使用しているかどうかは、クエリのプロファイルの`BitmapIndexFilterRows`フィールドを確認することで判断できます。

## ビットマップインデックスの作成

列にビットマップインデックスを作成する方法は2つあります。

- テーブル作成時に列のビットマップインデックスを作成します。例：

    ```SQL
    CREATE TABLE d0.table_hash
    (
        k1 TINYINT,
        k2 DECIMAL(10, 2) DEFAULT "10.5",
        v1 CHAR(10) REPLACE,
        v2 INT SUM,
        INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '']
    )
    ENGINE = olap
    AGGREGATE KEY(k1, k2)
    DISTRIBUTED BY HASH(k1)
    PROPERTIES ("storage_type" = "column");
    ```

    次の表は、ビットマップインデックスに関連するパラメーターを説明しています。

    | **パラメーター** | **必須** | **説明**                                              |
    | ------------- | ------------ | ------------------------------------------------------------ |
    | index_name    | はい          | ビットマップインデックスの名前。命名規則は以下の通りです：<ul><li>名前には文字、数字(0-9)、アンダースコア(_)が含まれていてもよい。ただし、最初の文字はアルファベットでなければなりません。</li><li>名前は64文字を超えることはできません。</li></ul>ビットマップインデックスの名前はテーブル内で一意でなければなりません。                              |
    | column_name   | はい          | ビットマップインデックスが作成される列の名前。複数の列名を指定して、一度に複数の列にビットマップインデックスを作成することができます。複数の列はコンマ(`,`)で区切ります。  |
    | COMMENT       | いいえ           | ビットマップインデックスのコメント。                             |


複数の `INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '']` コマンドを指定することで、一度に複数の列のビットマップインデックスを作成できます。これらのコマンドはコンマ(`,`)で区切る必要があります。CREATE TABLE ステートメントのその他のパラメーターの説明については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

- CREATE INDEX ステートメントを使用して、テーブルの列にビットマップインデックスを作成します。パラメーターの説明と例については、[CREATE INDEX](../sql-reference/sql-statements/data-definition/CREATE_INDEX.md)を参照してください。

    ```SQL
    CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT ''];
    ```

## ビットマップインデックスの表示

SHOW INDEX ステートメントを使用して、テーブルに作成されたすべてのビットマップインデックスを表示できます。パラメーターの説明と例については、[SHOW INDEX](../sql-reference/sql-statements/Administration/SHOW_INDEX.md)を参照してください。

```SQL
SHOW { INDEX[ES] | KEY[S] } FROM [db_name.]table_name [FROM db_name];
```

> **注記**
>
> インデックスの作成は非同期プロセスです。したがって、作成プロセスが完了したインデックスのみを表示できます。

## ビットマップインデックスの削除

DROP INDEX ステートメントを使用して、テーブルからビットマップインデックスを削除できます。パラメーターの説明と例については、[DROP INDEX](../sql-reference/sql-statements/data-definition/DROP_INDEX.md)を参照してください。

```SQL
DROP INDEX index_name ON [db_name.]table_name;
```

## 使用例

たとえば、次の `employee` テーブルは、ある会社の従業員情報の一部を示しています。

| **ID** | **Gender** | **Position** | **Income_level** |
| ------ | ---------- | ------------ | ---------------- |
| 01     | female     | Developer    | level_1          |
| 02     | female     | Analyst      | level_2          |
| 03     | female     | Salesman     | level_1          |
| 04     | male       | Accountant   | level_3          |

### 単一列のクエリ

たとえば、`Gender` 列のクエリパフォーマンスを向上させたい場合、次のステートメントを使用してその列にビットマップインデックスを作成できます。

```SQL
CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
```

上記のステートメントを実行すると、次の図に示すようにビットマップインデックスが生成されます。

![図](../assets/3.6.1-2.png)

1. 辞書の構築: StarRocksは `Gender` 列の辞書を構築し、`female` と `male` をINT型のコード値 `0` と `1` にマッピングします。
2. ビットマップの生成: StarRocksはコード値に基づいて `female` と `male` のビットマップを生成します。`female` のビットマップは `1110` で、最初の3行に `female` が表示されるためです。`male` のビットマップは `0001` で、`male` が4行目にのみ表示されるためです。

会社の男性従業員を調べたい場合は、次のようにクエリを送信できます。

```SQL
SELECT xxx FROM employee WHERE Gender = male;
```

クエリが送信されると、StarRocksは辞書を検索して `male` のコード値 `1` を取得し、次に `male` のビットマップ `0001` を取得します。これは、4行目のみがクエリ条件に一致することを意味します。その後、StarRocksは最初の3行をスキップして4行目のみを読み取ります。

### 複数列のクエリ

たとえば、`Gender` と `Income_level` 列のクエリパフォーマンスを向上させたい場合、次のステートメントを使用してこれら2つの列にビットマップインデックスを作成できます。

- `Gender`

    ```SQL
    CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
    ```

- `Income_level`

    ```SQL
    CREATE INDEX index2 ON employee (Income_level) USING BITMAP COMMENT 'index2';
    ```

上記の2つのステートメントを実行すると、次の図に示すようにビットマップインデックスが生成されます。

![図](../assets/3.6.1-3.png)

StarRocksはそれぞれ `Gender` 列と `Income_level` 列の辞書を構築し、これら2つの列の異なる値のビットマップを生成します。

- `Gender`: `female` のビットマップは `1110` で、`male` のビットマップは `0001` です。
- `Income_level`: `level_1` のビットマップは `1010`、`level_2` のビットマップは `0100`、`level_3` のビットマップは `0001` です。

`level_1` の給与を受け取る女性従業員を見つけたい場合は、次のようにクエリを送信できます。

```SQL
 SELECT xxx FROM employee 
 WHERE Gender = female AND Income_level = level_1;
```

クエリが送信されると、StarRocksは `Gender` と `Income_level` の辞書を同時に検索して、以下の情報を取得します：

- `female`のコード値は`0`で、`female`のビットマップは`1110`です。
- `level_1`のコード値は`0`で、`level_1`のビットマップは`1010`です。

StarRocksは`AND`演算子に基づいてビット単位の論理演算`1110 & 1010`を実行し、結果`1010`を取得します。この結果により、StarRocksは1行目と3行目のみを読み取ることになります。
