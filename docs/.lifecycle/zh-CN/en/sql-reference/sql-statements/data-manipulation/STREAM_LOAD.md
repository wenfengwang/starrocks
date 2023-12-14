---
displayed_sidebar: "Chinese"
---

# 流式加载

## 描述

StarRocks提供了基于HTTP的流式加载方法，帮助您从本地文件系统或数据流源加载数据。在您提交加载作业后，StarRocks会同步运行作业，并在作业完成后返回作业结果。您可以根据作业结果确定作业是否成功。有关流式加载的应用场景、限制、原理和支持的数据文件格式的信息，请参见[通过流式加载从本地文件系统加载](../../../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)。

> **注意**
>
> - 使用流式加载将数据加载到StarRocks表后，还会更新在该表上创建的物化视图的数据。
> - 您只能以对拥有这些StarRocks表具有INSERT权限的用户身份将数据加载到StarRocks表中。如果您没有INSERT权限，请按照[GRANT](../account-management/GRANT.md)中提供的说明向连接到StarRocks集群的用户授予INSERT权限。

## 语法

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

本主题以curl为例，介绍使用流式加载加载数据的方法。除了curl之外，您也可以使用其他兼容HTTP的工具或语言来执行流式加载。与加载相关的参数包含在HTTP请求头字段中。在输入这些参数时，注意以下几点：

- 您可以使用分块传输编码，就像本主题中演示的那样。如果您不选择分块传输编码，您必须输入`Content-Length`头字段以指示要传输的内容的长度，从而确保数据的完整性。

  > **注意**
  >
  > 如果您使用curl执行流式加载，StarRocks会自动添加`Content-Length`头字段，您不需要手动输入它。

- 您必须添加`Expect`头字段并将其值指定为`100-continue`，如`"Expect:100-continue"`。这有助于防止不必要的数据传输，并在您的作业请求被拒绝时减少资源开销。

请注意，在StarRocks中，某些文字作为SQL语言的保留关键字。不要直接在SQL语句中使用这些关键字。如果要在SQL语句中使用此类关键字，请将其括在一对反引号（`）中。请参见[关键字](../../../sql-reference/sql-statements/keywords.md)。

## 参数

### username和password

指定您用于连接到StarRocks集群的帐户的用户名和密码。这是一个必须的参数。如果您使用的帐户没有设置密码，您只需输入`<username>:`。

### XPUT

指定HTTP请求方法。这是一个必须的参数。流式加载仅支持PUT方法。

### url

指定StarRocks表的URL。语法：

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

以下表格描述了URL中的参数。

| 参数           | 是否必须 | 描述                                                         |
| -------------- | -------- | ------------------------------------------------------------ |
| fe_host        | 是       | StarRocks集群中FE节点的IP地址。<br/>**注意**<br/>如果您将加载作业提交到特定的BE节点，则必须输入BE节点的IP地址。 |
| fe_http_port   | 是       | StarRocks集群中FE节点的HTTP端口号。默认端口号为`8030`。<br/>**注意**<br/>如果您将加载作业提交到特定的BE节点，则必须输入BE节点的HTTP端口号。默认端口号为`8030`。 |
| database_name  | 是       | StarRocks表所属数据库的名称。                                  |
| table_name     | 是       | StarRocks表的名称。                                           |

> **注意**
>
> 您可以使用[显示前端](../Administration/SHOW_FRONTENDS.md)查看FE节点的IP地址和HTTP端口。

### data_desc

描述您要加载的数据文件。`data_desc`描述符可以包括数据文件的名称、格式、列分隔符、行分隔符、目标分区以及对StarRocks表的列映射。语法：

```Bash
-T <file_path>
-H "format: CSV | JSON"
-H "column_separator: <column_separator>"
-H "row_delimiter: <row_delimiter>"
-H "columns: <column1_name>[, <column2_name>, ... ]"
-H "partitions: <partition1_name>[, <partition2_name>, ...]"
-H "temporary_partitions: <temporary_partition1_name>[, <temporary_partition2>, ...]"
-H "jsonpaths: [ \"<json_path1>\"[, \"<json_path2>\", ...] ]"
-H "strip_outer_array:  true | false"
-H "json_root: <json_path>"
```

`data_desc`描述符中的参数可分为三类：通用参数、CSV参数和JSON参数。

#### 通用参数

| 参数              | 是否必须 | 描述                                                         |
| ----------------- | -------- | ------------------------------------------------------------ |
| file_path         | 是       | 数据文件的保存路径。您可以选择包括文件名的扩展名。           |
| format            | 否       | 数据文件的格式。有效值：`CSV` 和 `JSON`。默认值：`CSV`。      |
| partitions        | 否       | 您要将数据文件加载到的分区。如果您不指定此参数，默认情况下，StarRocks将数据文件加载到StarRocks表的所有分区中。 |
| temporary_partitions| 否       | 要将数据文件加载到的[临时分区](../../../table_design/Temporary_partition.md)的名称。您可以指定多个临时分区，各临时分区必须使用逗号（,）分隔。|
| columns           | 否       | 数据文件和StarRocks表之间的列映射。<br/>如果数据文件中的字段可以按顺序映射到StarRocks表中的列，则无需指定此参数。而是可以使用此参数对数据进行转换。例如，如果您加载CSV数据文件，文件由两列组成，可以按顺序映射到StarRocks表的两列`id`和`city`，则您可以指定`"columns: city,tmp_id, id = tmp_id * 100"`。有关更多信息，请参见本主题中的"[列映射](#列映射)"部分。 |

#### CSV参数

| 参数                 | 是否必须 | 描述                                                         |
| --------------------- | -------- | ------------------------------------------------------------ |
| column_separator      | 否       | 用于在数据文件中分隔字段的字符。如果不指定此参数，默认为`\t`，表示制表符。<br/>请确保您使用此参数指定的列分隔符与数据文件中使用的列分隔符相同。<br/>**注意**<br/>对于CSV数据，您可以使用UTF-8字符串（例如逗号（,）、制表符或竖线（\|））作为长度不超过50个字节的文本分隔符。 |
| row_delimiter         | 否       | 用于在数据文件中分隔行的字符。如果不指定此参数，默认为`\n`。 |
| skip_header           | 否       | 指定在数据文件为CSV格式时，是否跳过数据文件的前几行。类型：INTEGER。默认值：`0`。<br />在某些CSV格式的数据文件中，开头的前几行用于定义诸如列名和列数据类型的元数据。通过设置`skip_header`参数，您可以启用StarRocks跳过数据加载期间的数据文件前几行。例如，如果将此参数设置为`1`，StarRocks将在数据加载期间跳过数据文件的第一行。<br />数据文件中开头的前几行必须使用您在加载命令中指定的行分隔符分隔。 |
| trim_space       | 否       | 指定在数据文件为CSV格式时，是否删除数据文件中列分隔符前后的空格。类型：BOOLEAN。默认值：`false`。<br />对于某些数据库，在将数据导出为CSV格式的数据文件时，会添加空格到列分隔符中。这些空格称为前置空格或尾随空格，取决于它们的位置。通过设置`trim_space`参数，您可以启用StarRocks在数据加载过程中删除此类不必要的空格。<br />请注意，StarRocks不会删除双引号“enclose”指定字符对包裹的字段内的空格（包括前置空格和尾随空格）。例如，以下字段值使用竖线（<code class="language-text">&#124;</code>）作为列分隔符，双引号（`"`）作为`enclose`指定字符：<br /><code class="language-text">&#124;"爱StarRocks"&#124;</code> <br /><code class="language-text">&#124;" 爱StarRocks "&#124;</code> <br /><code class="language-text">&#124; "爱StarRocks" &#124;</code> <br />如果将`trim_space`设置为`true`，StarRocks将处理前述字段值如下：<br /><code class="language-text">&#124;"爱StarRocks"&#124;</code> <br /><code class="language-text">&#124;" 爱StarRocks "&#124;</code> <br /><code class="language-text">&#124;"爱StarRocks"&#124| 
| enclose          | 否       | 指定用于根据[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)包裹数据文件中字段值的字符。类型：单字节字符。默认值：`NONE`。最常用的字符是单引号（`'`）和双引号（`"`）。<br />使用`enclose`指定字符包裹的所有特殊字符（包括行分隔符和列分隔符）被视为普通符号。StarRocks可以做比RFC4180更多，因为它允许您指定任何单字节字符作为`enclose`指定字符。<br />如果字段值包含一个`enclose`指定字符，则可以使用相同的字符来转义该`enclose`指定字符。例如，您将`enclose`设为`"`，字段值为`a "quoted" c`。在这种情况下，您可以将字段值输入为`"a ""quoted"" c"`到数据文件中。|
| escape           | 否       | 指定用于转义各种特殊字符（例如行分隔符、列分隔符、转义字符和`enclose`指定字符）的字符。这些特殊字符然后由StarRocks视为普通字符，并解析为它们所在的字段值的一部分。类型：单字节字符。默认值：`NONE`。最常用的字符是斜杠（`\`），在SQL语句中必须写为双斜杠（`\\`）。<br />**注意**<br />`escape`指定的字符同时应用于每一对`enclose`指定字符的内部和外部。<br />以下是两个示例：<ul><li>当您将`enclose`设置为`"`，`escape`设置为`\`时，StarRocks将`"say \"Hello world\""`解析为`say "Hello world"`。</li><li>假设列分隔符为逗号（`,`）。当您将`escape`设置为`\\`时，StarRocks将`a, b\\, c`解析为两个独立的字段值：`a`和`b, c`。</li></ul>|
| max_filter_ratio | No       | 加载作业的最大错误容忍度。错误容忍度是加载作业请求的所有数据记录中，由于数据质量不合格而被过滤掉的数据记录的最大百分比。有效值：`0` 到 `1`。默认值：`0`。<br/>我们建议您保留默认值 `0`。这样，如果检测到不合格的数据记录，加载作业将失败，从而确保数据的正确性。<br/>如果要忽略不合格的数据记录，可以将此参数设置为大于 `0` 的值。这样，即使数据文件包含不合格的数据记录，加载作业也可以成功。<br/>**注意**<br/>不合格的数据记录不包括通过 WHERE 子句过滤掉的数据记录。 |
| log_rejected_record_num | No           | 指定可以记录的不合格数据行的最大数目。此参数从 v3.1 开始受支持。有效值：`0`、`-1` 和任意非零正整数。默认值：`0`。<ul><li>值 `0` 指定被过滤掉的数据行不会被记录。</li><li>值 `-1` 指定所有被过滤掉的数据行都会被记录。</li><li>例如 `n` 这样的非零正整数指定每个 BE 最多可以记录 `n` 个被过滤掉的数据行。</ul> |
| timeout          | No       | 加载作业的超时时间。有效值：`1` 到 `259200`。单位：秒。默认值：`600`。<br/>**注意**除了 `timeout` 参数，还可以使用[FE 参数](../../../administration/Configuration.md) `stream_load_default_timeout_second` 来集中控制 StarRocks 集群中所有 Stream Load 作业的超时时间。如果指定了 `timeout` 参数，那么 `timeout` 参数指定的超时时间生效。如果未指定 `timeout` 参数，则 `stream_load_default_timeout_second` 参数指定的超时时间生效。 |
| strict_mode      | No       | 指定是否启用[严格模式](../../../loading/load_concept/strict_mode.md)。有效值：`true` 和 `false`。默认值：`false`。值 `true` 指定启用严格模式，值 `false` 指定禁用严格模式。 |
| timezone         | No       | 加载作业使用的时区。默认值：`Asia/Shanghai`。该参数的值会影响像 strftime、alignment_timestamp 和 from_unixtime 等函数返回的结果。由此参数指定的时区是会话级别时区。有关详细信息，请参见[配置时区](../../../administration/timezone.md)。 |
| load_mem_limit   | No       | 可分配给加载作业的最大内存量。单位：字节。默认情况下，加载作业的最大内存大小为 2 GB。此参数的值不能超过可分配给每个 BE 的最大内存量。 |
| merge_condition  | No       | 指定您要用作确定更新是否生效的条件的列的名称。仅当源数据记录在指定列中具有大于或等于目标数据记录的值时，来自源记录的更新才会生效。StarRocks 自 v2.5 开始支持条件更新。有关详细信息，请参见[通过加载更改数据](../../../loading/Load_to_Primary_Key_tables.md)。<br/>**注意**<br/>您指定的列不能是主键列。此外，只有使用主键表的表支持条件更新。 |

## 列映射

### 配置 CSV 数据加载的列映射

- 如果数据文件的列可以按顺序一对一地映射到 StarRocks 表的列，您不需要配置数据文件和 StarRocks 表之间的列映射。

- 如果数据文件的列无法按顺序一对一地映射到 StarRocks 表的列，您需要使用 `columns` 参数配置数据文件和 StarRocks 表之间的列映射。这包括以下两种用例：

  - **列数相同但列顺序不同。** **并且，在加载到匹配的 StarRocks 表列之前，数据文件中的数据不需要通过函数计算。**

    在 `columns` 参数中，您需要按照数据文件列的排列顺序指定 StarRocks 表列的名称。

    例如，StarRocks 表由三列组成，顺序分别为 `col1`、`col2` 和 `col3`，数据文件也由三列组成，可以按顺序映射到 StarRocks 表列 `col3`、`col2` 和 `col1`。在这种情况下，您需要指定 `"columns: col3, col2, col1"`。

  - **列数不同且列顺序不同。另外，数据文件中的数据在加载到匹配的 StarRocks 表列之前需要通过函数计算。**

    在 `columns` 参数中，您需要按照数据文件列的排列顺序指定 StarRocks 表列的名称，并指定希望使用的函数来计算数据。以下是两个示例：

    - StarRocks 表由三列组成，分别为 `col1`、`col2` 和 `col3`。数据文件由四列组成，其中前三列可以按顺序映射到 StarRocks 表列 `col1`、`col2` 和 `col3`，第四列无法映射到任何 StarRocks 表列。在这种情况下，您需要临时指定数据文件的第四列的名称，且临时名称必须与任何 StarRocks 表列名不同。例如，您可以指定 `"columns: col1, col2, col3, temp"`，其中数据文件的第四列临时命名为 `temp`。
    - StarRocks 表由三列组成，分别为 `year`、`month` 和 `day`。数据文件只包含一列，其中包含 `yyyy-mm-dd hh:mm:ss` 格式的日期和时间值。在这种情况下，您可以指定 `"columns: col, year = year(col), month=month(col), day=day(col)"`，其中 `col` 是数据文件列的临时名称，且函数 `year = year(col)`、`month=month(col)` 和 `day=day(col)` 用于从数据文件列 `col` 中提取数据，并将其加载到映射的 StarRocks 表列中。例如，`year = year(col)` 用于从数据文件列 `col` 中提取 `yyyy` 数据，并将其加载到 StarRocks 表列 `year` 中。

有关详细示例，请参见[配置列映射](#配置列映射)。

### 配置 JSON 数据加载的列映射

- 如果 JSON 文档的键与 StarRocks 表的列具有相同的名称，您可以使用简单模式加载 JSON 格式数据。在简单模式下，您无需指定`jsonpaths` 参数。此模式要求 JSON 格式数据必须是由花括号 `{}` 指示的对象，例如 `{"category": 1, "author": 2, "price": "3"}`。在此示例中，`category`、`author` 和 `price` 是键名，并且这些键可以按名称一对一地映射到 StarRocks 表的 `category`、`author` 和 `price` 列。

- 如果 JSON 文档的键与 StarRocks 表的列具有不同的名称，您可以使用匹配模式加载 JSON 格式数据。在匹配模式下，您需要使用 `jsonpaths` 和 `COLUMNS` 参数来指定 JSON 文档和 StarRocks 表之间的列映射：

  - 在 `jsonpaths` 参数中，按照它们在 JSON 文档中的排列顺序指定 JSON 键。
  - 在 `COLUMNS` 参数中，指定 JSON 键与 StarRocks 表列之间的映射关系：
    - 在 `COLUMNS` 参数中指定的列名在顺序上一一映射到 JSON 键。
    - 在 `COLUMNS` 参数中指定的列名按名称一一映射到 StarRocks 表列。

有关使用匹配模式加载 JSON 格式数据的示例，请参见[使用匹配模式加载 JSON 数据](#使用匹配模式加载-json-数据)。

## 返回值

加载作业完成后，StarRocks以JSON格式返回作业结果。示例：

```JSON
{
    "TxnId": 1003,
    "Label": "label123",
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 1000000,
    "NumberLoadedRows": 999999,
    "NumberFilteredRows": 1,
    "NumberUnselectedRows": 0,
    "LoadBytes": 40888898,
    "LoadTimeMs": 2144,
    "BeginTxnTimeMs": 0,
"StreamLoadPlanTimeMs": 1,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 11,
    "CommitAndPublishTimeMs": 16,
}

下表描述了返回的作业结果中的参数。

| 参数                    | 描述                                  |
| ---------------------- | ------------------------------------- |
| TxnId                  | 装载作业的事务 ID。                    |
| Label                  | 装载作业的标签。                       |
| Status                 | 数据装载的最终状态。<ul><li>`Success`：数据已成功装载，可进行查询。</li><li>`Publish Timeout`：装载作业已成功提交，但数据仍无法查询。您无需重试装载数据。</li><li>`Label Already Exists`：装载作业的标签已被用于另一个装载作业。数据可能已成功装载或正在装载中。</li><li>`Fail`：数据未能装载。您可以重试装载作业。</li></ul> |
| Message                | 装载作业的状态。如果装载作业失败，则返回详细的失败原因。 |
| NumberTotalRows        | 读取的数据记录总数。                   |
| NumberLoadedRows       | 成功装载的数据记录总数。仅当 `Status` 返回值为 `Success` 时，此参数有效。 |
| NumberFilteredRows     | 由于数据质量不足而被过滤掉的数据记录数。 |
| NumberUnselectedRows   | 由 `WHERE` 子句过滤掉的数据记录数。    |
| LoadBytes              | 装载的数据量。单位：字节。              |
| LoadTimeMs             | 装载作业所需的时间量。单位：毫秒。      |
| BeginTxnTimeMs         | 运行装载作业事务所需的时间量。           |
| StreamLoadPlanTimeMs   | 生成装载作业执行计划所需的时间量。       |
| ReadDataTimeMs         | 读取装载作业数据所需的时间量。            |
| WriteDataTimeMs       | 写入装载作业数据所需的时间量。            |
| CommitAndPublishTimeMs | 提交和发布装载数据所需的时间量。          |

如果装载作业失败，StarRocks 还会返回 `ErrorURL`。例如：

```JSON
{"ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"}
```

`ErrorURL` 提供了一个 URL，您可以从中获取已被过滤掉的不合格数据记录的详细信息。您可以在提交装载作业时设置可选参数 `log_rejected_record_num`，来指定最大可记录的不合格数据行数。

您可以运行 `curl "url"` 直接查看已被过滤掉的不合格数据记录的详细信息。您也可以运行 `wget "url"` 导出这些数据记录的详细信息：

```Bash
wget http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be
```

导出的数据记录详细信息将保存到本地文件中，文件名类似为 `_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be`。您可以使用 `cat` 命令来查看文件。

然后，您可以调整装载作业的配置，并重新提交装载作业。

## 示例

### 装载 CSV 数据

本节以 CSV 数据为例，描述如何使用各种参数设置和组合来满足不同的装载需求。

#### 设置超时时间

您的 StarRocks 数据库 `test_db` 包含名为 `table1` 的表。该表由三列组成，分别是 `col1`、`col2` 和 `col3`。

您的数据文件 `example1.csv` 也包含三列，可以依次映射到 `table1` 的 `col1`、`col2` 和 `col3`。

如果您希望在 100 秒内将 `example1.csv` 中的所有数据装载到 `table1` 中，运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### 设置错误容忍度

您的 StarRocks 数据库 `test_db` 包含名为 `table2` 的表。该表由三列组成，分别是 `col1`、`col2` 和 `col3`。

您的数据文件 `example2.csv` 也包含三列，可以依次映射到 `table2` 的 `col1`、`col2` 和 `col3`。

如果您希望在 `example2.csv` 中的所有数据装载到 `table2` 中，并设置最大错误容忍度为 `0.2`，运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### 配置列映射

您的 StarRocks 数据库 `test_db` 包含名为 `table3` 的表。该表由三列组成，分别是 `col1`、`col2` 和 `col3`。

您的数据文件 `example3.csv` 也包含三列，可以依次映射到 `table3` 的 `col2`、`col1` 和 `col3`。

如果您希望将 `example3.csv` 中的所有数据装载到 `table3` 中，运行以下命令：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **注意**
>
> 在上面的示例中，`example3.csv` 的列无法按照它们在 `table3` 中排列的顺序映射到 `table3` 的列。因此，您需要使用 `columns` 参数来定义 `example3.csv` 列与 `table3` 列之间的临时映射关系。

#### 设置筛选条件

您的 StarRocks 数据库 `test_db` 包含名为 `table4` 的表。该表由三列组成，分别是 `col1`、`col2` 和 `col3`。

您的数据文件 `example4.csv` 也包含三列，可以依次映射到 `table4` 的 `col1`、`col2` 和 `col3`。

如果您希望仅将 `example4.csv` 中第一列的值等于 `20180601` 的数据记录装载到 `table4` 中，运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label4" \
    -H "Expect:100-continue" \
    -H "columns: col1, col2, col3" \
    -H "where: col1 = 20180601" \
    -T example4.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
```

> **注意**
>
> 在上面的示例中，`example4.csv` 和 `table4` 具有相同数量的列且可以按顺序映射，但您需要使用 `WHERE` 子句来指定基于列的筛选条件。因此，您需要使用 `columns` 参数为 `example4.csv` 的列定义临时列名。

#### 设置目标分区

您的 StarRocks 数据库 `test_db` 包含名为 `table5` 的表。该表由三列组成，分别是 `col1`、`col2` 和 `col3`。

您的数据文件 `example5.csv` 也包含三列，可以依次映射到 `table5` 的 `col1`、`col2` 和 `col3`。

如果您希望将 `example5.csv` 中的所有数据装载到 `table5` 的 `p1` 和 `p2` 分区中，运行以下命令：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \

```plaintext
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### 设置严格模式和时区

您的StarRocks数据库test_db包含一个名为table6的表。该表包含三列，分别是col1、col2和col3。

您的数据文件example6.csv也包含三列，可以按顺序映射到table6的col1、col2和col3。

如果您想要使用严格模式和时区Africa/Abidjan将example6.csv中的所有数据加载到table6中，运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### 将数据加载到包含HLL类型列的表中

您的StarRocks数据库test_db包含一个名为table7的表。该表包含两个HLL类型列，分别是col1和col2。

您的数据文件example7.csv还包含两列，其中第一列可以映射到table7的col1，而第二列无法映射到任何table7的列。在将example7.csv中的第一列数据加载到table7的col1之前，需要使用函数将其转换为HLL类型数据。

如果您想要将example7.csv中的数据加载到table7中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```

> **注意**
>
> 在上述示例中，使用"columns"参数按顺序将example7.csv的两列命名为"temp1"和"temp2"。然后，使用函数将数据转换如下：
>
> - 使用"hll_hash"函数将example7.csv的"temp1"列的值转换为HLL类型数据，并映射到"table7"的"col1"列。
>
> - 使用"hll_empty"函数将指定的默认值填充到"table7"的"col2"列中。

有关"hll_hash"和"hll_empty"函数的使用，请参见[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)和[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)。

#### 将数据加载到包含BITMAP类型列的表中

您的StarRocks数据库test_db包含一个名为table8的表。该表包含两个BITMAP类型列，分别是col1和col2。

您的数据文件example8.csv也包含两列，其中第一列可以映射到table8的col1，而第二列无法映射到任何table8的列。在将example8.csv中的第一列数据加载到table8的col1之前，需要使用函数将其转换。

如果您想要将example8.csv中的数据加载到table8中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **注意**
>
> 在上述示例中，使用"columns"参数按顺序将example8.csv的两列命名为"temp1"和"temp2"。然后，使用函数将数据转换如下：
>
> - 使用"to_bitmap"函数将example8.csv的"temp1"列的值转换为BITMAP类型数据，并映射到"table8"的"col1"列。
>
> - 使用"bitmap_empty"函数将指定的默认值填充到"table8"的"col2"列中。

有关"to_bitmap"和"bitmap_empty"函数的使用，请参见[to_bitmap](../../../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)和[bitmap_empty](../../../sql-reference/sql-functions/bitmap-functions/bitmap_empty.md)。

#### 设置"skip_header"、"trim_space"、"enclose"和"escape"

您的StarRocks数据库test_db包含一个名为table9的表。该表包含三列，分别是col1、col2和col3。

您的数据文件example9.csv也包含三列，按顺序映射到table13的col2、col1和col3。

如果您想要加载example9.csv中的所有数据到table9，并且跳过example9.csv的前五行，删除列分隔符之前和之后的空格，并设置"enclose"为"\"和"escape"为"\"，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:3875" \
    -H "Expect:100-continue" \
    -H "trim_space: true" -H "skip_header: 5" \
    -H "column_separator:," -H "enclose:\"" -H "escape:\\" \
    -H "columns: col2, col1, col3" \
    -T example9.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl9/_stream_load
```

### 加载JSON数据

本节介绍加载JSON数据时需要注意的参数设置。

您的StarRocks数据库test_db包含一个名为tbl1的表，其架构如下：

```SQL
`category` varchar(512) NULL COMMENT "",`author` varchar(512) NULL COMMENT "",`title` varchar(512) NULL COMMENT "",`price` double NULL COMMENT ""
```

#### 使用简单模式加载JSON数据

假设您的数据文件example1.json包含以下数据：

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

要加载example1.json中的所有数据到tbl1，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label6" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -T example1.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注意**
>
> 在上述示例中，未指定"columns"和"jsonpaths"参数。因此，example1.json中的键将按名称映射到tbl1的列。

为了增加吞吐量，Stream Load支持一次性加载多个数据记录。例如：

```JSON
[{"category":"C++","author":"avc","title":"C++ primer","price":89.5},{"category":"Java","author":"avc","title":"Effective Java","price":95},{"category":"Linux","author":"avc","title":"Linux kernel","price":195}]
```

#### 使用匹配模式加载JSON数据

StarRocks执行以下步骤来匹配和处理JSON数据：

1.（可选）根据`strip_outer_array`参数设置的指示去除JSON数据的最外层数组结构。

   > **注意**
   >
   > 仅当JSON数据的最外层为由一对方括号"[]"指示的数组结构时，此步骤才会执行。您需要将`strip_outer_array`设置为`true`。

2.（可选）根据`json_root`参数设置指示JSON数据的根元素。

   > **注意**
   >
   > 仅当JSON数据有根元素时，此步骤才会执行。您需要使用`json_root`参数指定根元素。

3.根据`jsonpaths`参数设置提取指定的JSON数据。

##### 使用未指定根元素的匹配模式加载JSON数据

假设您的数据文件example2.json包含以下数据：

```JSON
```JSON
{"category":"xuxb111","author":"1avc","price":895},{"category":"xuxb222","author":"2avc","price":895},{"category":"xuxb333","author":"3avc","price":895}
```

加载example2.json中的`category`，`author`和`price`字段，执行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label7" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.category\",\"$.price\",\"$.author\"]" \
    -H "columns: category, price, author" \
    -T example2.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注意**
>
> 在上述例子中，JSON数据的最外层是由一对方括号`[]`表示的数组结构。该数组结构包含多个JSON对象，每个对象代表一个数据记录。因此，您需要将`strip_outer_array`设置为`true`，以剥离最外层的数组结构。在加载过程中会忽略您不想加载的`title`字段。

##### 使用匹配模式和指定根元素加载JSON数据

假设您的数据文件`example3.json`包含以下数据：

```JSON
{"id": 10001,"RECORDS":[{"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},{"category":"22","author":"2avc","price":895,"timestamp":1589191487},{"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}],"comments": ["3 records", "there will be 3 rows"]}
```

加载example3.json中的`category`，`author`和`price`字段，执行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "format: json" \
    -H "json_root: $.RECORDS" \
    -H "strip_outer_array: true" \
    -H "jsonpaths: [\"$.category\",\"$.price\",\"$.author\"]" \
    -H "columns: category, price, author" -H "label:label8" \
    -T example3.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注意**
>
> 在上述例子中，JSON数据的最外层是由一对方括号`[]`表示的数组结构。该数组结构包含多个JSON对象，每个对象代表一个数据记录。因此，您需要将`strip_outer_array`设置为`true`，以剥离最外层的数组结构。在加载过程中会忽略您不想加载的`title`和`timestamp`字段。另外，`json_root`参数用于指定JSON数据的根元素，即数组。