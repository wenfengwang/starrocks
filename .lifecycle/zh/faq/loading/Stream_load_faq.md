---
displayed_sidebar: English
---

# 流加载

## 1. Stream Load 是否支持识别 CSV 格式文件前几行的列名，或者在读取数据时跳过这几行？

Stream Load 不支持识别 CSV 格式文件前几行中的列名。Stream Load 将这几行作为普通数据处理，与其他数据行无异。

在 v2.5 及更早版本中，Stream Load 不支持在读取数据时跳过 CSV 文件的前几行。如果您要加载的 CSV 文件的前几行包含列名，请采取以下措施之一：

- 修改您用来导出数据的工具的设置，然后重新导出一个前几行不包含列名的 CSV 文件。
- 使用命令如 sed -i '1d' filename 删除 CSV 文件的前几行。
- 在加载命令或语句中，使用 -H "where: <column_name> != '<column_name>'" 来过滤掉 CSV 文件的前几行。这里的 <column_name> 指的是前几行中出现的任何一个列名。请注意，StarRocks 会先转换数据，然后进行过滤。因此，如果前几行中的列名无法成功转换成相应的目标数据类型，这些列名会被转换成 NULL 值。这意味着目标 StarRocks 表不能包含设置为 NOT NULL 的列。
- 在加载命令或语句中，添加 -H "max_filter_ratio:0.01" 来设置最大错误容忍率为 1% 或更低，但仍能容忍少量错误行，这样 StarRocks 就可以忽略前几行数据转换失败的问题。在这种情况下，即使返回 ErrorURL 指出存在错误行，Stream Load 任务也能成功完成。不要设置过高的 max_filter_ratio 值，否则可能会忽视重要的数据质量问题。

从 v3.0 起，Stream Load 支持 `skip_header` 参数，用于指定是否跳过 CSV 文件的前几行。更多信息请参见 [CSV 参数](../../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md#csv-parameters)。

## 2. 要加载到分区列的数据不符合标准的 DATE 或 INT 类型。比如数据格式是 202106.00，我该如何使用 Stream Load 转换数据？

StarRocks支持在加载数据时进行转换。更多信息请参见[加载时数据转换](../../loading/Etl_in_loading.md)。

假设您要加载的 CSV 格式文件名为 TEST，包含四列：NO、DATE、VERSION 和 PRICE，DATE 列的数据格式为非标准格式，如 202106.00。如果您想在 StarRocks 中使用 DATE 作为分区列，您需要首先创建一个 StarRocks 表，例如包含以下四列：NO、VERSION、PRICE 和 DATE。然后，您需要为 StarRocks 表的 DATE 列指定数据类型，可以是 DATE、DATETIME 或 INT。最后，在创建 Stream Load 任务时，您需要在加载命令或声明中指定以下设置，以将源 DATE 列的数据类型转换为目标列的数据类型：

```Plain
-H "columns: NO,DATE_1, VERSION, PRICE, DATE=LEFT(DATE_1,6)"
```

在上述示例中，DATE_1 可以视为映射到目标 DATE 列的临时命名列，而加载到目标 DATE 列的最终结果是通过 left() 函数计算得出的。请注意，您必须首先列出源列的临时名称，然后使用函数转换数据。支持的函数包括标量函数、非聚合函数和窗口函数。

## 3. 如果我的 Stream Load 任务报告了“体积超出最大大小：10737418240，限制：10737418240”的错误，我该怎么办？

源数据文件的大小超出了 Stream Load 支持的最大文件大小 10 GB。您可以采取以下措施之一：

- 使用 seq -w 0 n 命令将源数据文件分割成更小的文件。
- 使用 `curl -XPOST http:///be_host:http_port/api/update_config?streaming_load_max_mb=\u003cfile_size\u003e` 调整值[BE配置项](../../administration/BE_configuration.md#configure-be-dynamic-parameters) `streaming_load_max_mb` 以增加最大文件大小。
