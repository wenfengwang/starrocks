---
displayed_sidebar: Chinese
---

# データ圧縮

StarRocks は、テーブルとインデックスデータの圧縮（compression）をサポートしています。データ圧縮は、ストレージスペースを節約するだけでなく、I/O集中型タスクのパフォーマンスを向上させることができます。ただし、データの圧縮と解凍には追加の CPU リソースが必要です。

## データ圧縮アルゴリズムの選択

StarRocks は、LZ4、Zstandard（または zstd）、zlib、Snappy の4種類のデータ圧縮アルゴリズムをサポートしています。これらのデータ圧縮アルゴリズムは、圧縮率と圧縮/解凍のパフォーマンスに違いがあります。一般的に、これらのアルゴリズムの圧縮率は次のようにランク付けされます：zlib > Zstandard > LZ4 > Snappy。中でも、zlib は高い圧縮比を持っています。しかし、データが高度に圧縮されるため、zlib アルゴリズムを使用するテーブルは、インポートとクエリのパフォーマンスに影響を受ける可能性があります。一方、LZ4 と Zstandard アルゴリズムは、圧縮比と解凍のパフォーマンスがバランスが取れています。ビジネスシーンに応じてこれらの圧縮アルゴリズムから選択し、ストレージやパフォーマンスの要件を満たすことができます。ストレージスペースの使用に特別な要件がない場合は、LZ4 または Zstandard アルゴリズムの使用をお勧めします。

> **説明**
>
> 異なるデータタイプも圧縮率に影響を与える可能性があります。

## データ圧縮アルゴリズムの設定

データ圧縮アルゴリズムは、テーブルを作成する際にのみ設定でき、その後の変更はできません。

以下の例は、Zstandard アルゴリズムを使用して `data_compression` テーブルを作成する方法を示しています。詳細は [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。

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

> **説明**
>
> データ圧縮アルゴリズムを指定しない場合、StarRocks はデフォルトで LZ4 を使用します。

[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) コマンドを使用して、特定のテーブルで使用されている圧縮アルゴリズムを確認できます。
