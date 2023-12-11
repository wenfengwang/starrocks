---
displayed_sidebar: "Japanese"
---

# データ圧縮

StarRocksはテーブルとインデックスのストレージのためにデータ圧縮をサポートしています。データ圧縮はストレージスペースを節約するだけでなく、I/O集中タスクのパフォーマンスも向上させます。これは、StarRocksがリクエストごとにディスクからより少ないページを読み取ることができるためです。ただし、データを圧縮および展開するために余分なCPUリソースが必要です。

## データ圧縮アルゴリズムの選択

StarRocksでは、4つのデータ圧縮アルゴリズム（LZ4、Zstandard（またはzstd）、zlib、およびSnappy）をサポートしています。これらのデータ圧縮アルゴリズムは、圧縮率と圧縮/展開パフォーマンスが異なります。一般的に、これらのアルゴリズムの圧縮率は次のようにランク付けされます: zlib > Zstandard > LZ4 > Snappy。その中でも、zlibは比較的高い圧縮率を示しています。高度に圧縮されたデータの結果として、zlib圧縮アルゴリズムを使用したテーブルの読み込みおよびクエリのパフォーマンスにも影響があります。特に、LZ4およびZstandardは、圧縮率と展開パフォーマンスのバランスが取れています。これらの圧縮アルゴリズムからビジネスニーズに適したものを選択することができます。ストレージスペースの節約またはよりよいパフォーマンスに対する特定の要求がない場合は、LZ4またはZstandardをお勧めします。

> **注記**
>
> 異なるデータ型は圧縮率に影響する可能性があります。

## テーブルのデータ圧縮アルゴリズムの指定

テーブルを作成する際にのみ、テーブルのデータ圧縮アルゴリズムを指定することができます。後から変更することはできません。

以下の例では、Zstandardアルゴリズムを使用して`data_compression`という名前のテーブルを作成しています。詳しい手順については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)をご覧ください。

```SQL
CREATE TABLE `data_compression` (
  `id`      INT(11)     NOT NULL     COMMENT "",
  `name`    CHAR(200)   NULL         COMMENT ""
)
ENGINE=OLAP 
UNIQUE KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
"compression" = "ZSTD"
);
```

> **注記**
>
> データ圧縮アルゴリズムが指定されていない場合、StarRocksはデフォルトでLZ4を使用します。

テーブルの圧縮アルゴリズムは、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を使用して表示することができます。