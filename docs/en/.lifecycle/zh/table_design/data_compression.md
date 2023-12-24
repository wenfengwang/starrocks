---
displayed_sidebar: English
---

# 数据压缩

StarRocks 支持对表和索引进行数据压缩。数据压缩不仅有助于节省存储空间，还能提高 I/O 密集型任务的性能，因为 StarRocks 可以减少每个请求从磁盘读取的页面数量。需要注意的是，压缩和解压数据需要额外的 CPU 资源。

## 选择数据压缩算法

StarRocks 支持四种数据压缩算法：LZ4、Zstandard（或 zstd）、zlib 和 Snappy。这些数据压缩算法在压缩比和压缩/解压性能方面各有不同。通常来说，这些算法的压缩比排名如下：zlib > Zstandard > LZ4 > Snappy。其中，zlib 的压缩比相对较高。由于数据高度压缩，使用 zlib 压缩算法的表的加载和查询性能也会受到影响。特别是 LZ4 和 Zstandard，它们具有良好的压缩比和解压性能。您可以根据业务需求选择这些压缩算法，以节省存储空间或提高性能。如果对存储空间没有特定要求，我们建议使用 LZ4 或 Zstandard。

> **注意**
>
> 不同的数据类型会影响压缩比。

## 为表指定数据压缩算法

您只能在创建表时为表指定数据压缩算法，创建后无法更改。

下面的示例创建了一个名为 `data_compression` 的表，并使用了 Zstandard 算法。有关详细说明，请参阅 [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

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
> 如果未指定数据压缩算法，StarRocks 默认使用 LZ4。

您可以使用 [SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md) 查看表的压缩算法。
