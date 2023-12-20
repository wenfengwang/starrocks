---
displayed_sidebar: English
---

# Stream Load

## 1. Stream Load 是否支持识别 CSV 格式文件前几行中的列名，或者在读取数据时跳过前几行？

Stream Load 不支持识别 CSV 格式文件前几行中的列名。Stream Load 将前几行视为与其他行一样的普通数据。

在 v2.5 及更早版本中，Stream Load 不支持在读取数据时跳过 CSV 文件的前几行。如果您要加载的 CSV 文件的前几行包含列名，请采取以下措施之一：

- 修改您用于导出数据的工具的设置，然后重新导出一个在前几行中不包含列名的 CSV 文件。
- 使用 `sed -i '1d' filename` 等命令删除 CSV 文件的前几行。
- 在加载命令或语句中，使用 `-H "where: <column_name> != '<column_name>'"` 来过滤掉 CSV 文件的前几行。`<column_name>` 是前几行中的任何一个列名。请注意，StarRocks 首先转换然后过滤源数据。因此，如果前几行中的列名无法转换成相应的目标数据类型，将返回 `NULL` 值。这意味着目标 StarRocks 表不能包含设置为 `NOT NULL` 的列。
- 在加载命令或语句中，添加 `-H "max_filter_ratio:0.01"` 来设置最大错误容忍率为 1% 或更低，但可以容忍少数错误行，从而允许 StarRocks 忽略前几行数据转换失败的情况。在这种情况下，即使返回 `ErrorURL` 指示错误行，Stream Load 作业也可以成功。不要将 `max_filter_ratio` 设置得太大，否则可能会忽略一些重要的数据质量问题。

从 v3.0 开始，Stream Load 支持 `skip_header` 参数，用于指定是否跳过 CSV 文件的前几行。更多信息，请参见 [CSV 参数](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)。

## 2. 要加载到分区列的数据不是标准的 DATE 或 INT 类型。例如，数据格式类似于 202106.00。使用 Stream Load 加载数据时，如何转换数据？

StarRocks 支持在加载时转换数据。更多信息，请参见 [加载时转换数据](../../loading/Etl_in_loading.md)。

假设您要加载一个名为 `TEST` 的 CSV 格式文件，该文件包含四列：`NO`、`DATE`、`VERSION` 和 `PRICE`，其中 `DATE` 列的数据为非标准格式，例如 202106.00。如果您想在 StarRocks 中使用 `DATE` 作为分区列，您需要首先创建一个 StarRocks 表，例如包含以下四列：`NO`、`VERSION`、`PRICE` 和 `DATE`。然后，您需要指定 StarRocks 表的 `DATE` 列的数据类型为 DATE、DATETIME 或 INT。最后，在创建 Stream Load 作业时，您需要在加载命令或语句中指定以下设置，以将源 `DATE` 列的数据类型转换为目标列的数据类型：

```Plain
-H "columns: NO,DATE_1, VERSION, PRICE, DATE=LEFT(DATE_1,6)"
```

在上述示例中，`DATE_1` 可以被视为映射到目标 `DATE` 列的临时命名列，而最终加载到目标 `DATE` 列的结果是通过 `LEFT()` 函数计算得出的。请注意，您必须首先列出源列的临时名称，然后使用函数进行数据转换。支持的函数是标量函数，包括非聚合函数和窗口函数。

## 3. 如果我的 Stream Load 作业报告 “body 超出最大大小：10737418240，限制：10737418240” 错误，我该怎么办？

源数据文件的大小超过了 10 GB，这是 Stream Load 支持的最大文件大小。采取以下措施之一：

- 使用 `seq -w 0 n` 将源数据文件分割成更小的文件。
- 使用 `curl -XPOST http://<be_host>:<http_port>/api/update_config?streaming_load_max_mb=<file_size>` 调整 [BE 配置项](../../administration/BE_configuration.md#configure-be-dynamic-parameters) `streaming_load_max_mb` 的值，以增加最大文件大小。