---
displayed_sidebar: English
---

# データ圧縮

StarRocks は、テーブルとインデックスのストレージにおけるデータ圧縮をサポートしています。データ圧縮は、ストレージ容量を節約するだけでなく、StarRocks がリクエストごとにディスクから読み込むページ数を減らすことで、I/O 負荷の高いタスクのパフォーマンスを向上させます。ただし、データの圧縮と解凍には追加の CPU リソースが必要であることに注意してください。

## データ圧縮アルゴリズムの選択

StarRocks は、LZ4、Zstandard（または zstd）、zlib、Snappy の4つのデータ圧縮アルゴリズムをサポートしています。これらのデータ圧縮アルゴリズムは、圧縮率と圧縮/解凍のパフォーマンスにおいて異なります。一般的に、これらのアルゴリズムの圧縮率は次のようにランク付けされます：zlib > Zstandard > LZ4 > Snappy。中でも、zlib は比較的高い圧縮率を実現しています。高度に圧縮されたデータの結果として、zlib 圧縮アルゴリズムを使用したテーブルのロードおよびクエリパフォーマンスも影響を受けます。特に LZ4 と Zstandard は、圧縮率と解凍パフォーマンスのバランスが良いです。これらの圧縮アルゴリズムから選択して、ストレージ容量の削減やパフォーマンスの向上などのビジネスニーズに応じたものを選べます。特にストレージスペースの削減に特化した要求がない場合は、LZ4 または Zstandard を推奨します。

> **注記**
>
> 異なるデータ型は圧縮率に影響を与える可能性があります。

## テーブルのデータ圧縮アルゴリズムを指定する

テーブルのデータ圧縮アルゴリズムは、テーブルを作成する際にのみ指定でき、その後変更することはできません。

以下の例では、Zstandard アルゴリズムを使用して `data_compression` テーブルを作成します。詳細な手順については、[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。

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
> データ圧縮アルゴリズムが指定されていない場合、StarRocks はデフォルトで LZ4 を使用します。

テーブルの圧縮アルゴリズムは、[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) を使用して確認できます。
