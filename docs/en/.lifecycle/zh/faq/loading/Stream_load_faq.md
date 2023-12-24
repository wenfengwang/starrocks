---
displayed_sidebar: English
---

# 流加载

## 1. Stream Load 是否支持识别 CSV 格式文件前几行中的列名，或者在读取数据时跳过前几行？

Stream Load 不支持识别 CSV 格式文件前几行中保存的列名。Stream Load 将前几行视为普通数据，就像其他行一样。

在 v2.5 及更早版本中，Stream Load 不支持在读取数据时跳过 CSV 文件的前几行。如果要加载的 CSV 文件的前几行保留列名，请执行以下操作之一：

- 修改用于导出数据的工具的设置。然后，重新将数据导出为前几行不包含列名的 CSV 文件。
- 使用诸如 `sed -i '1d' filename` 等命令删除 CSV 文件的前几行。
- 在 load 命令或语句中，使用 `-H "where: <column_name> != '<column_name>'"` 来筛选掉 CSV 文件的前几行。 `<column_name>` 是前几行中保存的任何列名。需要注意的是，StarRocks 会先对源数据进行转换，然后进行过滤。因此，如果前几行中的列名无法转换为其匹配的目标数据类型，将为它们返回 `NULL` 值。这意味着目标 StarRocks 表不能包含设置为 `NOT NULL` 的列。
- 在 load 命令或语句中，添加 `-H "max_filter_ratio:0.01"` 来设置最大容错范围为 1% 或更低，但可以容忍几个错误行，从而允许 StarRocks 忽略前几行的数据转换失败。在这种情况下，即使返回 `ErrorURL` 以指示错误行，流加载作业仍可成功。不要将 `max_filter_ratio` 设置为较大的值。如果将 `max_filter_ratio` 设置为较大的值，可能会错过一些重要的数据质量问题。

从 v3.0 开始，Stream Load 支持 `skip_header` 参数，该参数指定是否跳过 CSV 文件的前几行。更多信息，请参见[CSV 参数](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)。

## 2. 要加载到分区列中的数据不是标准的 DATE 或 INT 类型。例如，数据的格式类似于 202106.00。如果使用 Stream Load 加载数据，如何转换数据？

StarRocks 支持在加载时进行数据转换。有关详细信息，请参阅 [加载时转换数据](../../loading/Etl_in_loading.md)。

假设您要加载一个名为 `TEST` 的 CSV 格式文件，该文件由四列 `NO`、`DATE`、`VERSION` 和 `PRICE` 组成，其中 `DATE` 列的数据是非标准格式，例如 202106.00。如果要将 `DATE` 用作 StarRocks 中的分区列，需要先创建一个 StarRocks 表，例如，该表由以下四列组成：`NO`、`VERSION`、`PRICE` 和 `DATE`。然后，您需要将 StarRocks 表的 `DATE` 列数据类型指定为 DATE、DATETIME 或 INT。最后，在创建流加载作业时，需要在 load 命令或语句中指定以下设置，将数据从源列的数据类型 `DATE` 转换为目标列的数据类型：

```Plain
-H "columns: NO,DATE_1, VERSION, PRICE, DATE=LEFT(DATE_1,6)"
```

在前面的示例中，`DATE_1` 可以认为是映射目标列的临时命名列 `DATE`，加载到目标列中的最终结果由 `left()` 函数计算。请注意，必须首先列出源列的临时名称，然后使用函数转换数据。支持的函数是标量函数，包括非聚合函数和窗口函数。

## 3. 如果我的 Stream Load 作业报告“body exceeded max size： 10737418240， limit： 10737418240”错误，我该怎么办？

源数据文件大小超过 10 GB，这是 Stream Load 支持的最大文件大小。执行以下操作之一：

- 使用 `seq -w 0 n` 将源数据文件拆分为较小的文件。
- 使用 `curl -XPOST http:///be_host:http_port/api/update_config?streaming_load_max_mb=<file_size>` 来调整 [BE 配置项](../../administration/BE_configuration.md#configure-be-dynamic-parameters) `streaming_load_max_mb` 的值，以增加最大文件大小。
