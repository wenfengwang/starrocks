---
displayed_sidebar: "Chinese"
---

# 数据流加载

## 1. 数据流加载是否支持在读取数据时识别 CSV 格式文件中前几行中保存的列名，或者跳过前几行？

数据流加载不支持在读取 CSV 格式文件时识别前几行中保存的列名。数据流加载将前几行视为像其他行一样的正常数据。

在 v2.5 及更早版本中，数据流加载不支持在读取 CSV 文件时跳过前几行。如果要加载的 CSV 文件的前几行保存列名，可执行以下操作之一：

- 修改用于导出数据的工具的设置，然后将数据重新导出为不保存列名在前几行的 CSV 文件。
- 使用命令，如 `sed -i '1d' filename`，删除 CSV 文件的前几行。
- 在加载命令或语句中，使用 `-H "where: <column_name> != '<column_name>'"` 过滤掉 CSV 文件的前几行。`<column_name>` 是前几行中保存的任何列名。请注意，StarRocks 首先转换，然后过滤源数据。因此，如果前几行中的列名未能转换为其匹配的目标数据类型，将返回它们的 `NULL` 值。这意味着目标 StarRocks 表不能包含被设置为 `NOT NULL` 的列。
- 在加载命令或语句中，添加 `-H "max_filter_ratio:0.01"` 来设置最大容错率，即 1% 或更低，但可以容忍一些错误行，从而允许 StarRocks 忽略前几行的数据转换失败。在这种情况下，即使返回 `ErrorURL` 指示错误行，数据流加载作业仍然可以成功。不要将 `max_filter_ratio` 设置为一个大值。如果将 `max_filter_ratio` 设置为一个大值，可能会忽略一些重要的数据质量问题。

从 v3.0 开始，数据流加载支持 `skip_header` 参数，用于指示是否跳过 CSV 文件的前几行。有关详细信息，请参见[CSV 参数](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)。

## 2. 要加载到分区列的数据不是标准的 DATE 或 INT 类型。例如，数据的格式类似于 202106.00。如果我使用数据流加载加载这些数据，应该如何转换数据？

StarRocks 支持在加载时转换数据。有关详细信息，请参见[加载时的数据转换](../../loading/Etl_in_loading.md)。

假设你要加载一个名为 `TEST` 的 CSV 格式文件，文件由四列 `NO`、`DATE`、`VERSION` 和 `PRICE` 组成，其中来自 `DATE` 列的数据是非标准格式，例如 202106.00。如果你希望在 StarRocks 中使用 `DATE` 作为分区列，首先需要创建一个 StarRocks 表，例如，由以下四列组成: `NO`、`VERSION`、`PRICE` 和 `DATE`。然后，你需要将 StarRocks 表中 `DATE` 列的数据类型指定为 DATE、DATETIME 或 INT。最后，在创建数据流加载作业时，需要在加载命令或语句中指定以下设置，将源 `DATE` 列的数据类型转换为目标列的数据类型：

```Plain
-H "columns: NO,DATE_1, VERSION, PRICE, DATE=LEFT(DATE_1,6)"
```

在上面的示例中，`DATE_1` 可以被视为是一个临时命名的列，映射到目标 `DATE` 列，最终加载到目标 `DATE` 列的结果由 `left()` 函数计算。请注意，你必须先列出源列的临时名称，然后使用函数转换数据。支持的函数是标量函数，包括非聚合函数和窗口函数。

## 3. 如果我的数据流加载作业报告了 "body exceed max size: 10737418240, limit: 10737418240" 错误，我该怎么办？

源数据文件的大小超过了数据流加载支持的最大文件大小 10 GB。你可以执行以下操作之一：

- 使用 `seq -w 0 n` 将源数据文件分割成较小的文件。
- 使用 `curl -XPOST http:///be_host:http_port/api/update_config?streaming_load_max_mb=<file_size>` 来调整[BE 配置项](../../administration/Configuration.md#configure-be-dynamic-parameters) `streaming_load_max_mb` 的值，以增加最大文件大小限制。