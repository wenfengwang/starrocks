---
displayed_sidebar: "Japanese"
---

# データ圧縮

StarRocksはテーブルとインデックスのストレージのためにデータ圧縮をサポートしています。データ圧縮はストレージスペースを節約するだけでなく、StarRocksはリクエストごとにディスクからのページ読み込みが少なくなるため、I/O集中タスクのパフォーマンスも向上します。ただし、データの圧縮と展開には追加のCPUリソースが必要です。

## データ圧縮アルゴリズムの選択

StarRocksは、LZ4、Zstandard（またはzstd）、zlib、およびSnappyの4つのデータ圧縮アルゴリズムをサポートしています。これらのデータ圧縮アルゴリズムは、圧縮率と圧縮/展開パフォーマンスが異なります。一般的に、これらのアルゴリズムの圧縮率は以下のようにランク付けされます：zlib > Zstandard > LZ4 > Snappy。その中で、zlibは比較的高い圧縮率を示しています。高度に圧縮されたデータの結果、zlib圧縮アルゴリズムを使用したテーブルのロードおよびクエリパフォーマンスも影響を受けることがあります。特に、LZ4とZstandardは、圧縮率と展開パフォーマンスがバランスよく保たれています。ストレージスペースを節約するための特定の要求がない場合、これらの圧縮アルゴリズムから選択することができます。ストレージスペースをあまり取らずにパフォーマンスを向上させたい場合は、LZ4またはZstandardをお勧めします。

> **注記**
>
> 異なるデータ型は圧縮率に影響を与える可能性があります。

## テーブルのデータ圧縮アルゴリズムを指定する

テーブル作成時にのみテーブルのデータ圧縮アルゴリズムを指定することができ、後で変更することはできません。

次の例は、Zstandardアルゴリズムで`data_compression`テーブルを作成しています。詳しい手順については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

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

テーブルの圧縮アルゴリズムは[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を使用して表示することができます。