---
displayed_sidebar: "English"
---

# ビットマップインデックス

このトピックでは、ビットマップインデックスの作成と管理方法について説明します。

ビットマップインデックスは、ビットマップを使用する特別なデータベースインデックスであり、ビットマップはビットの配列です。ビットは常に0または1の2つの値のいずれかです。ビットマップの各ビットはテーブルの単一の行に対応しており、ビットの値は対応する行の値によって決まります。

ビットマップインデックスは、指定した列のクエリ性能を向上させるのに役立ちます。クエリがソートキー列にアクセスした場合、StarRocksは[プレフィックスインデックス](../table_design/Sort_key.md)を使用してクエリ結果を効率的に返します。ただし、データブロックのプレフィックスインデックスエントリは36バイトを超えることはできません。ソートキーとして使用されていない列のクエリパフォーマンスを改善する場合は、その列のためにビットマップインデックスを作成できます。

## 利点

ビットマップインデックスを使用すると以下の点で利点があります。

- 列の基数が低い場合、つまりENUM型の列など、応答時間を短縮できます。列の区別される値の数が比較的多い場合、問い合わせ速度を向上させるために、ブルームフィルタインデックスを使用することをお勧めします。詳細については、[ブルームフィルタインデックス](../using_starrocks/Bloomfilter_index.md)を参照してください。
- 他のインデックス技術と比較してストレージスペースを少なく使用できます。通常、ビットマップインデックスはテーブル内のインデックス化されたデータのサイズの一部しか占めません。
- 複数のビットマップインデックスを組み合わせて複数の列でクエリを実行できます。詳細については、[複数の列でクエリ](#query-multiple-columns)を参照してください。

## 使用上の注意事項

- イコール(`=`)や[NOT] IN演算子を使用してフィルタリングできる列に対してビットマップインデックスを作成できます。
- 重複キー表または一意キー表のすべての列にビットマップインデックスを作成できます。集約表または主キー表の場合、主キー列にのみビットマップインデックスを作成できます。
- FLOAT、DOUBLE、BOOLEAN、およびDECIMAL型の列はビットマップインデックスの作成をサポートしていません。
- クエリがビットマップインデックスを使用しているかどうかを確認するには、クエリプロファイルの`BitmapIndexFilterRows`フィールドを表示してください。

## ビットマップインデックスの作成

列のビットマップインデックスを作成する方法は2つあります。

- テーブルの作成時に列のビットマップインデックスを作成します。例：

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

    以下の表は、ビットマップインデックスに関連するパラメータを説明しています。

    | **パラメータ** | **必須** | **説明**                                       |
    | ------------- | -------- | --------------------------------------------- |
    | index_name    | はい     | ビットマップインデックスの名前。以下の命名規則に従います：<ul><li>名前には文字、数字(0-9)、アンダースコア(_)を含めることができます。ただし、文字で始める必要があります。</li><li>名前は64文字を超えることはできません。</li></ul>ビットマップインデックスの名前はテーブル内で一意である必要があります。 |
    | column_name   | はい     | ビットマップインデックスを作成する列の名前。複数の列名を指定して一度に複数の列のビットマップインデックスを作成することができます。複数の列をカンマ(,)で区切ります。  |
    | COMMENT       | いいえ   | ビットマップインデックスのコメント。                  |

    `INDEX index_name (column_name [, ...]) [USING BITMAP] [COMMENT '']`コマンドを複数指定して複数列のビットマップインデックスを一度に作成することができます。これらのコマンドはカンマ(,)で区切る必要があります。CREATE TABLE文の他のパラメータの説明については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

- CREATE INDEX文を使用してテーブルの列にビットマップインデックスを作成します。パラメータの説明と例については、[CREATE INDEX](../sql-reference/sql-statements/data-definition/CREATE_INDEX.md)を参照してください。

    ```SQL
    CREATE INDEX index_name ON table_name (column_name) [USING BITMAP] [COMMENT ''];
    ```

## ビットマップインデックスの表示

SHOW INDEX文を使用して、テーブルに作成されたすべてのビットマップインデックスを表示できます。パラメータの説明と例については、[SHOW INDEX](../sql-reference/sql-statements/Administration/SHOW_INDEX.md)を参照してください。

```SQL
SHOW { INDEX[ES] | KEY[S] } FROM [db_name.]table_name [FROM db_name];
```

> **注記**
>
> インデックスの作成は非同期プロセスです。したがって、作成プロセスが完了したインデックスのみ表示できます。

## ビットマップインデックスの削除

DROP INDEX文を使用して、テーブルからビットマップインデックスを削除できます。パラメータの説明と例については、[DROP INDEX](../sql-reference/sql-statements/data-definition/DROP_INDEX.md)を参照してください。

```SQL
DROP INDEX index_name ON [db_name.]table_name;
```

## 使用ケース

たとえば、以下の表`employee`は企業の従業員情報の一部を示しています。

| **ID** | **Gender** | **Position** | **Income_level** |
| ------ | ---------- | ------------ | ---------------- |
| 01     | 女性       | 開発者       | レベル1           |
| 02     | 女性       | アナリスト   | レベル2           |
| 03     | 女性       | セールスマン | レベル1           |
| 04     | 男性       | 会計士       | レベル3           |

### 単一列の問い合わせ

たとえば、`Gender`列のクエリパフォーマンスを改善したい場合は、以下のステートメントを使用して列のためにビットマップインデックスを作成できます。

```SQL
CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
```

前述のステートメントを実行すると、次の図に示すようにビットマップインデックスが生成されます。

![図](../assets/3.6.1-2.png)

1. ディクショナリを構築します: StarRocksは`Gender`列のためにディクショナリを構築し、`female`と`male`をINT型のコード化された値`0`と`1`にマッピングします。
2. ビットマップを生成します: StarRocksはコード化された値に基づいて`female`および`male`のビットマップを生成します。`female`のビットマップは`1110`であり、`female`は最初の3行に表示されます。`male`のビットマップは`0001`であり、`male`は4行目にのみ表示されます。

企業内の男性従業員を検索したい場合、次のようにクエリを送信できます。

```SQL
SELECT xxx FROM employee WHERE Gender = male;
```

クエリが送信された後、StarRocksはディクショナリを検索して`male`のコード化された値である`1`を取得し、次に`male`のビットマップである`0001`を取得します。これにより、クエリ条件に一致するのは4行目だけであることがわかります。そのため、StarRocksは最初の3行をスキップして4行目だけを読み取ります。

### 複数列の問い合わせ

たとえば、`Gender`および`Income_level`列のクエリパフォーマンスを改善したい場合は、この2つの列のためにビットマップインデックスを作成できます。

- `Gender`

    ```SQL
    CREATE INDEX index1 ON employee (Gender) USING BITMAP COMMENT 'index1';
    ```

- `Income_level`

    ```SQL
    CREATE INDEX index2 ON employee (Income_level) USING BITMAP COMMENT 'index2';
    ```

前述の2つのステートメントを実行すると、次の図に示すようにビットマップインデックスが生成されます。

![図](../assets/3.6.1-3.png)

StarRocksはそれぞれ`Gender`および`Income_level`列のためにディクショナリを構築し、次にこれら2つの列の異なる値のためにビットマップを生成します。

- `Gender`：`female`のビットマップは`1110`であり、`male`のビットマップは`0001`です。
- `Income_level`：`level_1`のビットマップは`1010`、`level_2`のビットマップは`0100`、`level_3`のビットマップは`0001`です。

`Gender`が`female`でかつ`Income_level`が`level_1`の女性従業員を見つけたい場合、次のようにクエリを送信できます。

```SQL
SELECT xxx FROM employee WHERE Gender = female AND Income_level = level_1;
```

クエリが送信された後、StarRocksは`Gender`および`Income_level`のディクショナリを同時に検索して次の情報を取得します。

- `female`のコード化された値は`0`であり、`female`のビットマップは`1110`です。
- `level_1`のコード化された値は`0`であり、`level_1`のビットマップは`1010`です。

StarRocksは、`AND`演算子に基づいて`1110 & 1010`のビットごとの論理演算を行い、結果`1010`を取得します。この結果により、StarRocksは最初の行と3行目だけを読み取ります。