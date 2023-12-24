---
displayed_sidebar: English
---

# 流加载

## 描述

StarRocks提供了基于HTTP的Stream Load加载方法，帮助您从本地文件系统或流式数据源加载数据。在您提交加载作业后，StarRocks会同步运行该作业，并在作业完成后返回作业结果。您可以根据作业结果来判断作业是否成功。有关Stream Load的应用场景、限制、原理和支持的数据文件格式的信息，请参见[通过流加载从本地文件系统加载](../../../loading/StreamLoad.md#loading-from-a-local-file-system-via-stream-load)。

> **注意**
>
> - 使用Stream Load将数据加载到StarRocks表中后，该表上创建的物化视图的数据也会被更新。
> - 您只能以在那些StarRocks表上具有INSERT权限的用户身份将数据加载到StarRocks表中。如果您没有INSERT权限，请按照[GRANT](../account-management/GRANT.md)中的说明，为您连接到StarRocks集群的用户授予INSERT权限。

## 语法

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

本主题以curl为例，介绍了如何使用Stream Load加载数据。除了curl，您还可以使用其他兼容HTTP的工具或语言来执行Stream Load。与加载相关的参数包含在HTTP请求标头字段中。在输入这些参数时，请注意以下几点：

- 您可以使用分块传输编码，就像本主题中所演示的那样。如果您不选择分块传输编码，那么您必须输入一个`Content-Length`标头字段，以指示要传输的内容长度，从而确保数据的完整性。

  > **注意**
  >
  > 如果您使用curl执行Stream Load，StarRocks会自动添加一个`Content-Length`标头字段，因此您无需手动输入。

- 您必须添加一个`Expect`标头字段，并将其值指定为`100-continue`，就像`"Expect:100-continue"`中所示。这有助于防止不必要的数据传输，并在您的作业请求被拒绝时减少资源开销。

需要注意的是，在StarRocks中，一些文字被SQL语言用作保留关键字。不要直接在SQL语句中使用这些关键字。如果您想在SQL语句中使用这样的关键字，请将其括在一对反引号（`）中。请参见[关键字](../../../sql-reference/sql-statements/keywords.md)。

## 参数

### 用户名和密码

指定您用于连接到StarRocks集群的帐户的用户名和密码。这是一个必需的参数。如果您使用一个未设置密码的帐户，那么您只需要输入`<username>:`。

### XPUT

指定HTTP请求方法。这是一个必需的参数。Stream Load仅支持PUT方法。

### 网址

指定StarRocks表的URL。语法：

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

下表描述了URL中的参数。

| 参数     | 必填 | 描述                                                  |
| ------------- | -------- | ------------------------------------------------------------ |
| fe_host       | 是      | StarRocks集群中FE节点的IP地址。<br/>**注意**<br/>如果您将加载作业提交到特定的BE节点，那么您必须输入BE节点的IP地址。 |
| fe_http_port  | 是      | StarRocks集群中FE节点的HTTP端口号。默认端口号为`8030`。<br/>**注意**<br/>如果您将加载作业提交到特定的BE节点，那么您必须输入BE节点的HTTP端口号。默认端口号为`8030`。 |
| database_name | 是      | StarRocks表所属的数据库名称。 |
| table_name    | 是      | StarRocks表的名称。                             |

> **注意**
>
> 您可以使用[SHOW FRONTENDS](../Administration/SHOW_FRONTENDS.md)来查看FE节点的IP地址和HTTP端口。

### data_desc

描述您想要加载的数据文件。`data_desc`描述符可以包括数据文件的名称、格式、列分隔符、行分隔符、目标分区以及与StarRocks表的列映射关系。语法：

```Bash
-T <file_path>
-H "format: CSV | JSON"
-H "column_separator: <column_separator>"
-H "row_delimiter: <row_delimiter>"
-H "columns: <column1_name>[, <column2_name>, ... ]"
-H "partitions: <partition1_name>[, <partition2_name>, ...]"
-H "temporary_partitions: <temporary_partition1_name>[, <temporary_partition2_name>, ...]"
-H "jsonpaths: [ \"<json_path1>\"[, \"<json_path2>\", ...] ]"
-H "strip_outer_array:  true | false"
-H "json_root: <json_path>"
```

`data_desc`描述符中的参数可以分为三种类型：公共参数、CSV参数和JSON参数。

#### 公共参数

| 参数  | 必填 | 描述                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| file_path  | 是      | 数据文件的保存路径。您可以选择包括文件名在内的扩展名。 |
| format     | 否       | 数据文件的格式。有效值：`CSV`和`JSON`。默认值：`CSV`。 |
| partitions | 否       | 您要将数据文件加载到的分区。默认情况下，如果您不指定此参数，StarRocks会将数据文件加载到StarRocks表的所有分区中。 |
| temporary_partitions|  否       | 您要将数据文件加载到的[临时分区](../../../table_design/Temporary_partition.md)的名称。您可以指定多个临时分区，这些分区之间必须用逗号（,）分隔。|
| columns    | 否       | 数据文件与StarRocks表的列映射关系。<br/>如果数据文件中的字段可以依次映射到StarRocks表中的列上，那么您就不需要指定此参数。相反，您可以使用此参数来实现数据转换。例如，如果您加载一个CSV数据文件，该文件由两列组成，可以依次映射到StarRocks表的两列`id`和`city`上，那么您可以指定`"columns: city,tmp_id, id = tmp_id * 100"`。有关详细信息，请参见本主题中的“[列映射](#column-mapping)”部分。 |

#### CSV参数

| 参数        | 必填 | 描述                                                  |
| ---------------- | -------- | ------------------------------------------------------------ |
| column_separator | 否       | 数据文件中用于分隔字段的字符。如果您不指定此参数，那么此参数默认为`\t`，表示制表符。<br/>请确保您使用此参数指定的列分隔符与数据文件中使用的列分隔符相同。<br/>**注意**<br/>对于CSV数据，您可以使用UTF-8字符串，比如逗号（,）、制表符或者竖线（\|）等，作为文本分隔符，其长度不超过50个字节。 |
| row_delimiter    | 否       | 数据文件中用于分隔行的字符。如果您不指定此参数，那么此参数默认为`\n`。 |
| skip_header      | 否       | 指定当数据文件为CSV格式时是否跳过数据文件的前几行。类型：整数。默认值：`0`。<br />在某些CSV格式的数据文件中，开头的前几行用于定义元数据，比如列名和列数据类型。通过设置`skip_header`参数，您可以让StarRocks在加载数据时跳过数据文件的前几行。例如，如果您将此参数设置为`1`，那么StarRocks在加载数据时会跳过数据文件的第一行。<br />数据文件开头的前几行必须使用您在加载命令中指定的行分隔符进行分隔。 |
| trim_space       | 否       | 指定当数据文件为CSV格式时，是否删除数据文件中列分隔符前后的空格。类型：布尔值。默认值：`false`。<br />对于某些数据库，当您将数据导出为CSV格式的数据文件时，会在列分隔符前添加空格。这些空格被称为前导空格或尾随空格，具体取决于它们的位置。通过设置`trim_space`参数，您可以让StarRocks在加载数据时去除这些不必要的空格。<br />需要注意的是，StarRocks不会删除用`enclose`指定的字符包裹的字段中的空格（包括前导空格和尾随空格）。例如，以下字段值使用竖线（<code class="language-text">&#124;</code>）作为列分隔符，双引号（`"`）作为`enclose`指定字符：<br /><code class="language-text">&#124;“爱StarRocks”&#124;</code> <br /><code class="language-text">&#124;“ 爱StarRocks ”&#124;</code> <br /><code class="language-text">&#124; “爱StarRocks” &#124;</code> <br />如果您将`trim_space`设置为`true`，那么StarRocks会按照以下方式处理前述字段值：<br /><code class="language-text">&#124;“爱StarRocks”&#124;</code> <br /><code class="language-text">&#124;“ 爱StarRocks ”&#124;</code> <br /><code class="language-text">&#124;“爱StarRocks”&#124;</code> |
| enclose          | 否       | 指定数据文件为CSV格式时，根据[RFC4180](https://www.rfc-editor.org/rfc/rfc4180)将字段值包裹的字符。类型：单字节字符。默认值：`NONE`。最常见的字符是单引号（`'`）和双引号（`"`）。<br />所有使用`enclose`指定的字符包裹的特殊字符（包括行分隔符和列分隔符）都被视为普通符号。StarRocks可以做得比RFC4180更多，因为它允许您将任何单字节字符指定为`enclose`指定的字符。<br />如果字段值包含一个`enclose`指定的字符，那么您可以使用相同的字符对该`enclose`指定的字符进行转义。例如，如果您将`enclose`设置为`"`，并且一个字段值为`a "quoted" c`。在这种情况下，您可以在数据文件中输入字段值`"a ""quoted"" c"`。 |
| 转义           | 否       |指定用于转义各种特殊字符的字符，例如行分隔符、列分隔符、转义字符和`enclose`指定的字符，这些字符被StarRocks视为普通字符，并作为它们所在的字段值的一部分进行解析。类型：单字节字符。默认值：`NONE`。最常见的字符是斜杠（\），在SQL语句中必须写成双斜杠（\\）。<br />**注意**<br />：`escape`指定的字符应用于`enclose`指定的每对字符的内部和外部。以下是两个例子：<ul><li>当您将`enclose`设置为`"`，并将`escape`设置为`\`时，StarRocks将`"say \"Hello world\""`解析为`say "Hello world"`。</li><li>假设列分隔符为逗号（,）。当您将`escape`设置为`\`时，StarRocks将`a, b\, c`解析为两个单独的字段值：`a`和`b, c`。</li></ul> |

> **注意**
>
> - 对于CSV数据，您可以使用长度不超过50个字节的UTF-8字符串，例如逗号（,）、制表符或竖线（|），作为文本分隔符。
> - 空值使用`\N`表示。例如，数据文件由三列组成，该数据文件中的记录在第一列和第三列保存数据，但在第二列中不保存任何数据。在这种情况下，您需要在第二列使用`\N`表示空值。这意味着记录必须编译为`a,\N,b`而不是`a,,b`。`a,,b`表示记录的第二列包含一个空字符串。
> - v3.0及更高版本支持`skip_header`格式选项，包括`trim_space`、`enclose`、`escape`和。

#### JSON参数

| 参数         | 必需 | 描述                                                  |
| ----------------- | -------- | ------------------------------------------------------------ |
| jsonpaths（JSON路径）         | 否       | 要从JSON数据文件加载的键的名称。仅在使用匹配模式加载JSON数据时才需要指定此参数。此参数的值采用JSON格式。请参阅[为JSON数据加载配置列映射](#configure-column-mapping-for-json-data-loading)。           |
| strip_outer_array | 否       | 指定是否去除最外层的数组结构。有效值：`true`和`false`。默认值：`false`。<br/>在实际业务场景中，JSON数据可能具有最外层的数组结构，如一对方括号所示`[]`。在这种情况下，建议您将该参数设置为`true`，这样StarRocks就会去掉最外层的方括号`[]`，并将每个内部数组作为单独的数据记录加载。如果将该参数设置为`false`，StarRocks会将整个JSON数据文件解析为一个数组，并将该数组作为单个数据记录加载。<br/>例如，JSON数据为`[{"category" : 1, "author" : 2}, {"category" : 3, "author" : 4}]`。如果将该参数设置为`true`，`{"category" : 1, "author" : 2}`解析为单独的数据记录，这些数据记录将加载到单独的StarRocks表行中。`{"category" : 3, "author" : 4}` |
| json_root         | 否       | 要从JSON数据文件加载的JSON数据的根元素。仅在使用匹配模式加载JSON数据时才需要指定此参数。此参数的值是有效的JsonPath字符串。默认情况下，该参数的值为空，表示将加载JSON数据文件的所有数据。有关详细信息，请参阅本主题的“[使用指定了根元素的匹配模式加载JSON数据](#load-json-data-using-matched-mode-with-root-element-specified)”部分。 |
| ignore_json_size  | 否       | 指定是否检查HTTP请求中JSON正文的大小。<br/>**注意**<br/>默认情况下，HTTP请求中的JSON正文大小不能超过100MB。如果JSON正文大小超过100MB，则会出现错误“此批处理的大小超出了[104857600]json类型数据[8617627793]的最大大小。将ignore_json_size设置为跳过检查，尽管这可能会导致大量内存消耗。”。为了避免出现此错误，您可以添加`"ignore_json_size:true"`HTTP请求头，指示StarRocks不要检查JSON体大小。 |

加载JSON数据时，还需注意，每个JSON对象的大小不能超过4GB。如果JSON数据文件中的单个JSON对象大小超过4GB，则会报告错误“此解析器无法支持这么大的文档”。

### opt_properties

指定一些可选参数，这些参数将应用于整个加载作业。语法：

```Bash
-H "label: <label_name>"
-H "where: <condition1>[, <condition2>, ...]"
-H "max_filter_ratio: <num>"
-H "timeout: <num>"
-H "strict_mode: true | false"
-H "timezone: <string>"
-H "load_mem_limit: <num>"
-H "merge_condition: <column_name>"
```

可选参数说明如下表所示。

| 参数        | 必需 | 描述                                                  |
| ---------------- | -------- | ------------------------------------------------------------ |
| label            | 否       | 加载作业的标签。如果不指定该参数，StarRocks会自动为加载作业生成标签。<br/>StarRocks不允许使用一个标签多次加载一个数据批次。因此，StarRocks可以防止重复加载相同的数据。有关标签命名约定，请参阅[系统限制](../../../reference/System_limit.md)。<br/>默认情况下，StarRocks会保留最近3天内成功完成的加载作业的标签。您可以使用[FE参数](../../../administration/FE_configuration.md)`label_keep_max_second`更改标签保留期。 |
| where            | 否       | StarRocks筛选预处理数据所依据的条件。StarRocks只加载满足WHERE子句中指定过滤条件的预处理数据。 |
| max_filter_ratio | 否       | 加载作业的最大容错范围。容错是由于加载作业请求的所有数据记录中的数据质量不足而可以过滤掉的数据记录的最大百分比。有效值：`0`设置为`1`。默认值：`0`。<br/>我们建议您保留默认值`0`。这样，如果检测到不合格的数据记录，加载作业就会失败，从而确保数据的正确性。<br/>如果要忽略不合格的数据记录，可以将此参数设置为大于`0`的值。这样，即使数据文件包含不合格的数据记录，加载作业也可以成功。<br/>**注意：**<br/>非限定数据记录不包括通过WHERE子句过滤掉的数据记录。 |
| log_rejected_record_num | 否           | 指定可以记录的最大非限定数据行数。从v3.1开始支持此参数。有效值：`0`、`-1`和任何非零正整数。默认值：`0`。<ul><li>该值`0`指定不会记录筛选出的数据行。</li><li>该值`-1`指定将记录筛选出的所有数据行。</li><li>非零正整数（例如）`n`指定`n`每个BE上最多可以记录过滤掉的数据行。</li></ul> |
| timeout          | 否       | 加载作业的超时时间。有效值：`1`设置为`259200`。单位：秒。默认值：`600`。<br/>**注意**除了该参数`timeout`，您还可以使用[FE参数](../../../administration/FE_configuration.md)`stream_load_default_timeout_second`来集中控制StarRocks集群中所有Stream Load作业的超时时间。如果指定该参数`timeout`，则以该参数指定的超时时间`timeout`为准。如果不指定该参数，则以`timeout`该参数指定的超时时间为`stream_load_default_timeout_second`准。 |
| strict_mode      | 否       | 指定是否启用[严格模式](../../../loading/load_concept/strict_mode.md)。有效值：`true`和`false`。默认值：`false`。该`true`值指定启用严格模式，该值指定`false`禁用严格模式。 |
| 时区         | 否       | 加载作业使用的时区。默认值：`Asia/Shanghai`。此参数的值会影响strftime、alignment_timestamp和from_unixtime等函数返回的结果。此参数指定的时区是会话级别的时区。有关详细信息，请参阅[配置时区](../../../administration/timezone.md)。 |
| load_mem_limit   | 否       | 可以预配到加载作业的最大内存量。单位：字节。默认情况下，加载作业的最大内存大小为2GB。此参数的值不能超过可以为每个BE预配的最大内存量。 |
| merge_condition  | 否       | 指定要用作确定更新是否可以生效的条件的列的名称。仅当源数据记录的值大于或等于指定列中的目标数据记录时，从源记录到目标记录的更新才会生效。StarRocks从v2.5开始支持条件更新。有关详细信息，请参阅[通过加载更改数据](../../../loading/Load_to_Primary_Key_tables.md)。<br/>**注意**<br/>：您指定的列不能是主键列。此外，只有使用主键表的表才支持条件更新。 |

## 列映射

### 为CSV数据加载配置列映射

如果数据文件的列可以依次映射到StarRocks表的列，则无需配置数据文件和StarRocks表的列映射关系。

如果数据文件的列无法依次映射到StarRocks表的列，则需要使用`columns`参数配置数据文件和StarRocks表的列映射关系。这包括以下两个用例：

- **列数相同，但列顺序不同。** **此外，数据文件中的数据在加载到匹配的StarRocks表列中之前，不需要通过函数进行计算。**

  在`columns`参数中，需要按照与数据文件列排列顺序相同的顺序指定StarRocks表列的名称。

  例如，StarRocks表由三列组成，分别是`col1`、`col2`和`col3`，数据文件也由三列组成，可以映射到StarRocks表列`col3`、`col2`和`col1`的顺序。在这种情况下，您需要指定`"columns: col3, col2, col1"`。

- **不同的列数和不同的列顺序。此外，数据文件中的数据需要经过函数计算，然后才能加载到匹配的StarRocks表列中。**


  在 `columns` 参数中，您需要按照数据文件列的排列顺序指定 StarRocks 表的列名称，并指定要用于计算数据的函数。以下是两个示例：

  - StarRocks 表由三列组成，分别为 `col1`、 `col2` 和 `col3`。数据文件由四列组成，其中前三列可以依次映射到 StarRocks 表的 `col1`、 `col2` 和 `col3` 列，而第四列无法映射到 StarRocks 表的任何列。在这种情况下，您需要为数据文件的第四列临时指定一个名称，并且该临时名称必须与 StarRocks 表的任何列名称都不同。例如，您可以指定 `"columns: col1, col2, col3, temp"`，其中数据文件的第四列临时命名为 `temp`。
  - StarRocks 表由三列组成，分别为 `year`、 `month` 和 `day`。数据文件仅包含一列，该列以 `yyyy-mm-dd hh:mm:ss` 格式容纳日期和时间值。在这种情况下，您可以指定 `"columns: col, year=year(col), month=month(col), day=day(col)"`，其中 `col` 是数据文件列的临时名称，函数 `year=year(col)`、 `month=month(col)` 和 `day=day(col)` 用于从数据文件列 `col` 中提取数据并将数据加载到映射的 StarRocks 表列中。例如，`year=year(col)` 用于从数据文件列 `col` 中提取 `yyyy` 数据并将数据加载到 StarRocks 表列 `year` 中。

有关详细示例，请参阅 [配置列映射](#configure-column-mapping)。

### 针对 JSON 数据加载配置列映射

如果 JSON 文档的键与 StarRocks 表的列名称相同，您可以使用简单模式加载 JSON 格式的数据。在简单模式下，无需指定 `jsonpaths` 参数。此模式要求 JSON 格式的数据必须是用大括号 `{}` 表示的对象，例如 `{"category": 1, "author": 2, "price": "3"}`。在此示例中，`category`、 `author` 和 `price` 是键名，这些键可以按名称一一对应映射到 StarRocks 表的 `category`、 `author` 和 `price` 列。

如果 JSON 文档的键与 StarRocks 表的列名称不同，您可以使用匹配模式加载 JSON 格式的数据。在匹配模式下，需要使用 `jsonpaths` 和 `COLUMNS` 参数指定 JSON 文档和 StarRocks 表之间的列映射关系：

- 在 `jsonpaths` 参数中，按照 JSON 文档中的排列顺序指定 JSON 键。
- 在 `COLUMNS` 参数中，指定 JSON 键与 StarRocks 表列的映射关系：
  - 在 `COLUMNS` 参数中指定的列名将按顺序一一映射到 JSON 键。
  - 在 `COLUMNS` 参数中指定的列名将按名称一一对应到 StarRocks 表列中。

有关使用匹配模式加载 JSON 格式数据的示例，请参阅 [使用匹配模式加载 JSON 数据](#load-json-data-using-matched-mode)。

## 返回值

加载作业完成后，StarRocks 以 JSON 格式返回作业结果。例如：

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
    "CommitAndPublishTimeMs": 16
}
```

返回的作业结果中的参数说明如下表所示。

| 参数                   | 描述                                                  |
| ---------------------- | ------------------------------------------------------------ |
| TxnId                  | 加载作业的事务 ID。                          |
| Label                  | 加载作业的标签。                                   |
| Status                 | 加载数据的最终状态。<ul><li>`Success`：数据成功加载，可查询。</li><li>`Publish Timeout`：加载作业成功提交，但数据仍无法查询。您无需重试加载数据。</li><li>`Label Already Exists`：加载作业的标签已用于另一个加载作业。数据可能已成功加载或正在加载。</li><li>`Fail`：数据加载失败。您可以重试加载作业。</li></ul> |
| Message                | 加载作业的状态。如果加载作业失败，则返回详细的失败原因。 |
| NumberTotalRows        | 读取的数据记录总数。              |
| NumberLoadedRows       | 成功加载的数据记录总数。仅当 `Status` 返回的值为 `Success` 时，此参数才有效。 |
| NumberFilteredRows     | 由于数据质量不足而被过滤掉的数据记录数。 |
| NumberUnselectedRows   | 由 WHERE 子句筛选掉的数据记录数。 |
| LoadBytes              | 加载的数据量。单位：字节。              |
| LoadTimeMs             | 加载作业所花费的时间量。单位：毫秒  |
| BeginTxnTimeMs         | 运行加载作业的事务所花费的时间。 |
| StreamLoadPlanTimeMs   | 为加载作业生成执行计划所花费的时间。 |
| ReadDataTimeMs         | 读取加载作业的数据所花费的时间量。 |
| WriteDataTimeMs        | 为加载作业写入数据所花费的时间。 |
| CommitAndPublishTimeMs | 提交和发布加载作业的数据所花费的时间。 |

如果加载作业失败，StarRocks 还会返回 `ErrorURL`。例如：

```JSON
{"ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"}
```

`ErrorURL` 提供了一个 URL，您可以从中获取有关已筛选出的不合格数据记录的详细信息。您可以使用可选参数指定可以记录的最大非限定数据行数 `log_rejected_record_num`，该参数是在提交加载作业时设置的。

您可以运行 `curl "url"` 以直接查看有关筛选出的不合格数据记录的详细信息。您还可以运行 `wget "url"` 以导出有关这些数据记录的详细信息：

```Bash
wget http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be
```

导出的数据记录详细信息将保存到名称类似于 `_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be` 的本地文件中。您可以使用 `cat` 命令查看该文件。

然后，您可以调整加载作业的配置，并重新提交加载作业。

## 例子

### 加载 CSV 数据

本节以 CSV 数据为例，介绍如何采用各种参数设置和组合来满足不同的加载要求。

#### 设置超时期限

您的 StarRocks 数据库 `test_db` 包含一个名为 `table1` 的表。该表由三列组成，分别为 `col1`、 `col2` 和 `col3` 按顺序排列。

您的数据文件 `example1.csv` 也包含三列，可以按顺序映射到 `table1` 的 `col1`、 `col2` 和 `col3`。

如果要在最多 100 秒内将 `example1.csv` 中的所有数据加载到 `table1` 中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### 设置容错

您的 StarRocks 数据库 `test_db` 包含一个名为 `table2` 的表。该表由三列组成，分别为 `col1`、 `col2` 和 `col3` 按顺序排列。

您的数据文件 `example2.csv` 也包含三列，可以按顺序映射到 `table2` 的 `col1`、 `col2` 和 `col3`。

如果要以最大容错率为 `0.2` 将 `example2.csv` 中的所有数据加载到 `table2` 中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### 配置列映射

您的 StarRocks 数据库 `test_db` 包含一个名为 `table3` 的表。该表由三列组成，分别为 `col1`、 `col2` 和 `col3` 按顺序排列。

您的数据文件 `example3.csv` 也包含三列，可以按顺序映射到 `table3` 的 `col2`、 `col1` 和 `col3`。

如果要将 `example3.csv` 中的所有数据加载到 `table3` 中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **注意**
>
> 在前面的示例中，`example3.csv` 的列无法按照与这些列在 `table3` 中的排列顺序相同的顺序映射到 `table3` 的列。因此，您需要使用 `columns` 参数来配置 `example3.csv` 和 `table3` 之间的列映射。

#### 设置筛选条件

您的 StarRocks 数据库 `test_db` 包含一个名为 `table4` 的表。该表由三列组成，分别为 `col1`、 `col2` 和 `col3` 按顺序排列。

您的数据文件 `example4.csv` 也包含三列，可以按顺序映射到 `table4` 的 `col1`、 `col2` 和 `col3`。

如果只想加载 `example4.csv` 中第一列值等于 `20180601` 的数据记录到 `table4` 中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label4" \
    -H "Expect:100-continue" \
    -H "columns: col1, col2, col3]"\
    -H "where: col1 = 20180601" \
    -T example4.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
```

> **注意**
>
> 在上面的示例中，`example4.csv`和`table4`具有相同数量的列，可以按顺序进行映射，但是需要使用WHERE子句来指定基于列的筛选条件。因此，您需要使用`columns`参数为`example4.csv`的列定义临时名称。

#### 设置目标分区

您的StarRocks数据库`test_db`中包含一个名为`table5`的表。该表由三列组成，分别为`col1`、`col2`和`col3`。

您的数据文件`example5.csv`也包含三列，可以按顺序映射到`table5`的`col1`、`col2`和`col3`上。

如果要将`example5.csv`中的所有数据加载到`table5`的`p1`和`p2`分区中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### 设置严格模式和时区

您的StarRocks数据库`test_db`中包含一个名为`table6`的表。该表由三列组成，分别为`col1`、`col2`和`col3`。

您的数据文件`example6.csv`也包含三列，可以按顺序映射到`table6`的`col1`、`col2`和`col3`上。

如果要使用严格模式和时区`Africa/Abidjan`将`example6.csv`中的所有数据加载到`table6`中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### 将数据加载到包含HLL类型列的表中

您的StarRocks数据库`test_db`中包含一个名为`table7`的表。该表由两个HLL类型列组成，分别为`col1`和`col2`。

您的数据文件`example7.csv`也包含两列，其中第一列可以映射到`table7`的`col1`上，而第二列无法映射到`table7`的任何列上。在加载到`table7`的`col1`之前，`example7.csv`中第一列的值可以通过函数转换为HLL类型数据。

如果要将`example7.csv`中的数据加载到`table7`中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```

> **注意**
>
> 在上面的示例中，使用`columns`参数按顺序为`example7.csv`的两列命名为`temp1`和`temp2`。然后，使用函数进行数据转换，具体如下：
>
> - 使用`hll_hash`函数将`example7.csv`中的`temp1`的值转换为HLL类型数据，并将`example7.csv`中的`temp1`映射到`table7`的`col1`上。
>
> - 使用`hll_empty`函数将指定的默认值填充到`table7`的`col2`中。

有关`hll_hash`和`hll_empty`函数的用法，请参见[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)和[hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)。

#### 将数据加载到包含BITMAP类型列的表中

您的StarRocks数据库`test_db`中包含一个名为`table8`的表。该表由两个BITMAP类型列组成，分别为`col1`和`col2`。

您的数据文件`example8.csv`也包含两列，其中第一列可以映射到`table8`的`col1`上，而第二列无法映射到`table8`的任何列上。在加载到`table8`的`col1`之前，`example8.csv`中第一列的值可以通过函数转换。

如果要将`example8.csv`中的数据加载到`table8`中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **注意**
>
> 在上面的示例中，使用`columns`参数按顺序为`example8.csv`的两列命名为`temp1`和`temp2`。然后，使用函数进行数据转换，具体如下：
>
> - 使用`to_bitmap`函数将`example8.csv`中的`temp1`的值转换为BITMAP类型数据，并将`example8.csv`中的`temp1`映射到`table8`的`col1`上。
>
> - 使用`bitmap_empty`函数将指定的默认值填充到`table8`的`col2`中。

有关`to_bitmap`和`bitmap_empty`函数的用法，请参见[to_bitmap](../../../sql-reference/sql-functions/bitmap-functions/to_bitmap.md)和[bitmap_empty](../../../sql-reference/sql-functions/bitmap-functions/bitmap_empty.md)。

#### 设置`skip_header`、`trim_space`、`enclose`和`escape`

您的StarRocks数据库`test_db`中包含一个名为`table9`的表。该表由三列组成，分别为`col1`、`col2`和`col3`。

您的数据文件`example9.csv`也包含三列，它们依次映射到`table13`的`col2`、`col1`和`col3`上。

如果要将`example9.csv`中的所有数据加载到`table9`中，并且打算跳过`example9.csv`的前五行，删除列分隔符前后的空格，并将`enclose`设置为`\`，`escape`设置为`\`，请运行以下命令：

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

本节描述加载JSON数据时需要注意的参数设置。

您的StarRocks数据库`test_db`中包含一个名为`tbl1`的表，其架构如下：

```SQL
`category` varchar(512) NULL COMMENT "",`author` varchar(512) NULL COMMENT "",`title` varchar(512) NULL COMMENT "",`price` double NULL COMMENT ""
```

#### 使用简单模式加载JSON数据

假设您的数据文件`example1.json`包含以下数据：

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

要将`example1.json`中的所有数据加载到`tbl1`中，请运行以下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label6" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -T example1.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **注意**
>
> 在上面的示例中，未指定`columns`和`jsonpaths`参数。因此，`example1.json`中的键按名称映射到`tbl1`的列。

为了提高吞吐量，Stream Load支持一次加载多个数据记录。例如：

```JSON
[{"category":"C++","author":"avc","title":"C++ primer","price":89.5},{"category":"Java","author":"avc","title":"Effective Java","price":95},{"category":"Linux","author":"avc","title":"Linux kernel","price":195}]
```

#### 使用匹配模式加载JSON数据

StarRocks执行以下步骤来匹配和处理JSON数据：

1. （可选）根据`strip_outer_array`参数设置的指示去除JSON数据的最外层数组结构。

   > **注意**
   >
   > 仅当JSON数据的最外层是由一对方括号`[]`指示的数组结构时，才会执行此步骤。您需要将`strip_outer_array`设置为`true`。

2. （可选）根据`json_root`参数设置的指示匹配JSON数据的根元素。

   > **注意**
   >
   > 仅当JSON数据具有根元素时，才会执行此步骤。您需要使用`json_root`参数指定根元素。

3. 根据`jsonpaths`参数设置的指示提取指定的JSON数据。

##### 使用未指定根元素的匹配模式加载JSON数据

假设您的数据文件`example2.json`包含以下数据：

```JSON
[{"category":"xuxb111","author":"1avc","title":"SayingsoftheCentury","price":895},{"category":"xuxb222","author":"2avc","title":"SayingsoftheCentury","price":895},{"category":"xuxb333","author":"3avc","title":"SayingsoftheCentury","price":895}]
```

要仅加载`example2.json`中的`category`、`author`和`price`，请运行以下命令：

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
> 在上面的示例中，JSON数据的最外层是由一对方括号`[]`指示的数组结构。数组结构由多个JSON对象组成，每个对象表示一条数据记录。因此，您需要将`strip_outer_array`设置为`true`以剥离最外层的数组结构。在加载过程中，将忽略不想加载的`title`键。

##### 使用指定根元素的匹配模式加载JSON数据

假设您的数据文件`example3.json`包含以下数据：

```JSON

{"id": 10001,"RECORDS":[{"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},{"category":"22","author":"2avc","price":895,"timestamp":1589191487},{"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}],"comments": ["3 records", "there will be 3 rows"]}
```

若要从 `example3.json` 中仅加载 `category`、 `author` 和 `price`，请运行以下命令：

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
> 在上述示例中，JSON 数据的最外层是一个数组结构，由一对方括号 `[]` 表示。该数组结构由多个 JSON 对象组成，每个对象表示一条数据记录。因此，您需要将 `strip_outer_array` 设置为 `true`，以剥离最外层的数组结构。在加载过程中，将忽略您不想加载的 `title` 和 `timestamp` 键。此外，`json_root` 参数用于指定 JSON 数据的根元素，即数组。