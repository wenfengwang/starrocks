---
displayed_sidebar: English
---

# 数据压缩

StarRocks 支持表和索引存储的数据压缩。数据压缩不仅有助于节省存储空间，而且还可以提高 I/O 密集型任务的性能，因为 StarRocks 可以为每个请求从磁盘读取更少的页面。请注意，压缩和解压缩数据需要额外的 CPU 资源。

## 选择数据压缩算法

StarRocks 支持四种数据压缩算法：LZ4、Zstandard（或 zstd）、zlib 和 Snappy。这些数据压缩算法在压缩率和压缩/解压缩性能方面有所不同。一般来说，这些算法的压缩比排名如下：zlib > Zstandard > LZ4 > Snappy。其中 zlib 表现出了较高的压缩比。由于数据高度压缩，使用 zlib 压缩算法的表在加载和查询性能上也会受到影响。LZ4 和 Zstandard 尤其具有均衡的压缩比和解压缩性能。您可以根据业务需求在这些压缩算法中选择，以减少存储空间或提高性能。如果您对存储空间的需求不是特别高，我们建议使用 LZ4 或 Zstandard。

> **注意**
> 不同的数据类型可能会影响压缩比。

## 指定表的数据压缩算法

只有在创建表时才可以指定表的数据压缩算法，创建后无法更改。

以下示例创建了一个使用 Zstandard 算法的表 `data_compression`。有关详细说明，请参阅 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

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
> 如果没有指定数据压缩算法，StarRocks 默认使用 LZ4。

您可以使用 [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) 查看表的压缩算法。