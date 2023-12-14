---
displayed_sidebar: "Chinese"
---

# 数据压缩

StarRocks支持表和索引存储的数据压缩。数据压缩不仅有助于节省存储空间，还可以提高I/O密集型任务的性能，因为StarRocks可以在每个请求中从磁盘读取更少的页面。请注意，压缩和解压数据需要额外的CPU资源。

## 选择数据压缩算法

StarRocks支持四种数据压缩算法：LZ4、Zstandard（或zstd）、zlib和Snappy。这些数据压缩算法在压缩比和压缩/解压性能上各不相同。一般来说，这些算法的压缩比排序如下：zlib > Zstandard > LZ4 > Snappy。其中，zlib表现出相对较高的压缩比。由于数据压缩得非常高，对于使用zlib压缩算法的表的加载和查询性能也会受到影响。特别是，LZ4和Zstandard具有良好的压缩比和解压性能。您可以根据业务需求选择这些压缩算法，以节省存储空间或获得更好的性能。如果您对存储空间没有特殊需求，我们建议使用LZ4或Zstandard。

> **注意**
>
> 不同的数据类型会影响压缩比。

## 为表指定数据压缩算法

您只能在创建表时为表指定数据压缩算法，并且创建后无法修改。

以下示例创建了一个名为 `data_compression` 的表，使用了Zstandard算法。有关详细说明，请参见[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)。

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
> 如果未指定数据压缩算法，StarRocks将默认使用LZ4。

您可以使用[SHOW CREATE TABLE](../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_TABLE.md)查看表的压缩算法。