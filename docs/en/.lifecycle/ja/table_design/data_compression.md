---
displayed_sidebar: "Japanese"
---

# データ圧縮

StarRocksは、テーブルとインデックスのストレージにデータ圧縮をサポートしています。データ圧縮は、ストレージスペースを節約するだけでなく、I/O集中タスクのパフォーマンスも向上させます。なぜなら、StarRocksは各リクエストごとにディスクから読み込むページ数を減らすことができるからです。ただし、データの圧縮と展開には追加のCPUリソースが必要です。

## データ圧縮アルゴリズムの選択

StarRocksは、4つのデータ圧縮アルゴリズム（LZ4、Zstandard（またはzstd）、zlib、Snappy）をサポートしています。これらのデータ圧縮アルゴリズムは、圧縮率と圧縮/展開のパフォーマンスが異なります。一般的に、これらのアルゴリズムの圧縮率は次のようにランク付けされます：zlib > Zstandard > LZ4 > Snappy。その中で、zlibは比較的高い圧縮率を示しています。高圧縮データの結果として、zlib圧縮アルゴリズムを使用したテーブルの読み込みとクエリのパフォーマンスにも影響があります。特に、LZ4とZstandardは圧縮率と展開パフォーマンスのバランスが良いです。ストレージスペースを節約するか、パフォーマンスを向上させるために、これらの圧縮アルゴリズムの中から選択することができます。ストレージスペースに特別な要求がない場合は、LZ4またはZstandardをおすすめします。

> **注意**
>
> 異なるデータ型は、圧縮率に影響を与える場合があります。

## テーブルのデータ圧縮アルゴリズムの指定

テーブルのデータ圧縮アルゴリズムは、テーブルを作成する際にのみ指定することができ、その後は変更できません。

以下の例では、Zstandardアルゴリズムを使用してテーブル「data_compression」を作成しています。詳しい手順については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

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

> **注意**
>
> データ圧縮アルゴリズムが指定されていない場合、StarRocksはデフォルトでLZ4を使用します。

テーブルの圧縮アルゴリズムは、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)を使用して表示することができます。
