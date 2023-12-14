```yaml
displayed_sidebar: "Chinese"
```

# STREAM LOAD

## Function

Stream Load is a synchronous import method based on the HTTP protocol, which supports importing local files or data streams into StarRocks. After you submit the import job, StarRocks will execute the import job synchronously and return the result information of the import job. You can determine whether the import job is successful based on the returned result information. For information about the application scenarios, usage restrictions, basic principles, and supported data file formats of Stream Load, please refer to [Import from Local using Stream Load](../../../loading/StreamLoad.md#使用-stream-load-从本地导入).

> **Note**
>
> - The Stream Load operation updates the materialized views related to the original table in StarRocks at the same time.
> - The Stream Load operation requires INSERT permission on the target table. If your user account does not have INSERT permission, please refer to [GRANT](../account-management/GRANT.md) for user permission assignment.

## Syntax

```Bash
curl --location-trusted -u <username>:<password> -XPUT <url>
(
    data_desc
)
[opt_properties]        
```

This article describes how to use Stream Load to import data using the curl tool. In addition to using the curl tool, you can also submit import jobs to import data using other tools or languages that support the HTTP protocol. The import-related parameters are located in the request header of the HTTP request. When passing these import-related parameters, the following points should be noted:

- It is recommended to use the **chunked upload** method, as shown in the example in this article. If the **non-chunked upload** method is used, the request header field `Content-Length` must be used to indicate the length of the content to be uploaded in order to ensure data integrity.

  > **Note**
  >
  > When using the curl tool to submit import jobs, the `Content-Length` field will be automatically added, so it is not necessary to specify `Content-Length` manually.

- The `Expect` field in the request header of the HTTP request must specify `100-continue`, that is, `"Expect:100-continue"`. In this way, if the server rejects the import job request, unnecessary data transmission can be avoided, thereby reducing unnecessary resource overhead.

Note that in StarRocks, some words are reserved keywords in the SQL language and cannot be used directly in SQL statements. If you want to use these reserved keywords in SQL statements, you must enclose them in backticks (```). See [Keywords](../../../sql-reference/sql-statements/keywords.md) for details.

## Parameter Description

### username and password

Used to specify the username and password of the StarRocks cluster account. Mandatory parameters. If the account does not have a password set, only `<username>:` needs to be passed here.

### XPUT

Used to specify the HTTP request method. Mandatory parameter. Stream Load currently only supports the PUT method.

### url

Used to specify the URL address of the StarRocks table. Mandatory parameter. The syntax is as follows:

```Plain
http://<fe_host>:<fe_http_port>/api/<database_name>/<table_name>/_stream_load
```

The parameters in `url` are described in the table below.

| Parameter Name | Mandatory | Parameter Description                                          |
| -------------- | --------- | -------------------------------------------------------------- |
| fe_host        | Yes       | Specifies the IP address of the FE in the StarRocks cluster. <br />**Note**<br />If you directly submit an import job to a specific BE node, you need to pass the IP address of that BE. |
| fe_http_port   | Yes       | Specifies the HTTP port number of the FE in the StarRocks cluster. The default port number is 8030. <br />**Note**<br />If you directly submit an import job to a specific BE node, you need to pass the HTTP port number of that BE. The default port number is 8040. |
| database_name  | Yes       | Specifies the name of the database where the target StarRocks table is located. |
| table_name     | Yes       | Specifies the name of the target StarRocks table.               |

> **Note**
>
> You can use the [SHOW FRONTENDS](../Administration/SHOW_FRONTENDS.md) command to view the IP address and HTTP port number of the FE nodes.

### data_desc

Used to describe the source data file, including the name, format, column separator, row delimiter, target partitions, and the corresponding relationships with the columns in the StarRocks table. The syntax is as follows:

```Bash
-T <file_path>
-H "format: CSV | JSON"
-H "column_separator: <column_separator>"
-H "row_delimiter: <row_delimiter>"
-H "columns: <column1_name>[, <column2_name>，... ]"
-H "partitions: <partition1_name>[, <partition2_name>, ...]"
-H "temporary_partitions: <temporary_partition1_name>[, <temporary_partition2_name>, ...]"
-H "jsonpaths: [ \"<json_path1>\"[, \"<json_path2>\", ...] ]"
-H "strip_outer_array:  true | false"
-H "json_root: <json_path>"
```

The parameters in `data_desc` can be divided into three categories: common parameters, parameters applicable to CSV, and parameters applicable to JSON.

#### Common Parameters

| **Parameter Name** | **Mandatory** | **Parameter Description**                                      |
| ------------------ | ------------- | -------------------------------------------------------------- |
| file_path          | Yes           | Specifies the storage path of the source data file. The file name can optionally include or not include the extension. |
| format             | No            | Specifies the format of the data to be imported. The possible values are `CSV` and `JSON`. Default value: `CSV`. |
| partitions         | No            | Specifies which partitions to import the data to. If this parameter is not specified, the data will be imported to all partitions in the StarRocks table by default. |
| temporary_partitions | No            | Specifies which [temporary partitions](../../../table_design/Temporary_partition.md) to import the data to. |
| columns            | No            | Specifies the corresponding relationships between the columns in the source data file and the StarRocks table. If the columns in the source data file correspond one-to-one to the columns in the StarRocks table in order, the `columns` parameter does not need to be specified. You can achieve data transformation through the `columns` parameter. For example, to import a CSV format data file with two columns, which can correspond to the `id` and `city` columns in the target StarRocks table. If you want to realize the transformation of multiplying the data in the first column of the data file by 100 before loading it into the StarRocks table, you can specify `"columns: city,tmp_id, id = tmp_id * 100"`. Please refer to the "Column Mapping" section in this article for details. |

#### Parameters Applicable to CSV

| **Parameter Name** | **Mandatory** | **Parameter Description**                                      |
| ------------------ | ------------- | -------------------------------------------------------------- |
| column_separator   | No            | Specifies the column delimiter in the source data file. If this parameter is not specified, it defaults to `\t`, which is Tab. It is necessary to ensure that the column delimiter specified here is consistent with the column delimiter in the source data file. <br />**Note**<br />StarRocks supports setting a UTF-8 encoded string with a maximum length of 50 bytes as the column delimiter, including common delimiters such as comma (,), Tab, and Pipe (\|). |
| row_delimiter      | No            | Specifies the row delimiter in the source data file. If this parameter is not specified, it defaults to `\n`. |
| skip_header      | 否           | 用于指定跳过 CSV 文件最开头的几行数据。取值类型：INTEGER。默认值：`0`。<br />在某些 CSV 文件里，最开头的几行数据会用来定义列名、列类型等元数据信息。通过设置该参数，可以使 StarRocks 在导入数据时忽略 CSV 文件的前面几行。例如，如果设置该参数为 `1`，则 StarRocks 会在导入数据时忽略 CSV 文件的第一行。<br />这里的行所使用的分隔符须与您在导入命令中所设定的行分隔符一致。 |
| trim_space       | 否           | 用于指定是否去除 CSV 文件中列分隔符前后的空格。取值类型：BOOLEAN。默认值：`false`。<br />有些数据库在导出数据为 CSV 文件时，会在列分隔符的前后添加一些空格。根据位置的不同，这些空格可以称为“前导空格”或者“尾随空格”。通过设置该参数，可以使 StarRocks 在导入数据时删除这些不必要的空格。<br />需要注意的是，StarRocks 不会去除被 `enclose` 指定字符括起来的字段内的空格（包括字段的前导空格和尾随空格）。例如，列分隔符是竖线 (<code class="language-text">&#124;</code>)，`enclose` 指定的字符是双引号 (`"`)：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124; "Love StarRocks" &#124;</code> <br />如果设置 `trim_space` 为 `true`，则 StarRocks 处理后的结果数据如下：<br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> <br /><code class="language-text">&#124;" Love StarRocks "&#124;</code> <br /><code class="language-text">&#124;"Love StarRocks"&#124;</code> |
| enclose          | 否           | 根据 [RFC4180](https://www.rfc-editor.org/rfc/rfc4180)，用于指定把 CSV 文件中的字段括起来的字符。取值类型：单字节字符。默认值：`NONE`。最常用 `enclose` 字符为单引号 (`'`) 或双引号 (`"`)。<br />被 `enclose` 指定字符括起来的字段内的所有特殊字符（包括行分隔符、列分隔符等）均看做是普通符号。比 RFC4180 标准更进一步的是，StarRocks 提供的 `enclose` 属性支持设置任意单个字节的字符。<br />如果一个字段内包含了 `enclose` 指定字符，则可以使用同样的字符对 `enclose` 指定字符进行转义。例如，在设置了`enclose` 为双引号 (`"`) 时，字段值 `a "quoted" c` 在 CSV 文件中应该写作 `"a ""quoted"" c"`。 |
| escape           | 否           | 指定用于转义的字符。用来转义各种特殊字符，比如行分隔符、列分隔符、转义符、`enclose` 指定字符等，使 StarRocks 把这些特殊字符当做普通字符而解析成字段值的一部分。取值类型：单字节字符。默认值：`NONE`。最常用的 `escape` 字符为斜杠 (`\`)，在 SQL 语句中应该写作双斜杠 (`\\`)。<br />**说明**<br />`escape` 指定字符同时作用于 `enclose` 指定字符的内部和外部。<br />以下为两个示例：<ul><li>当设置 `enclose` 为双引号 (`"`) 、`escape` 为斜杠 (`\`) 时，StarRocks 会把 `"say \"Hello world\""` 解析成一个字段值 `say "Hello world"`。</li><li>假设列分隔符为逗号 (`,`) ，当设置 `escape` 为斜杠 (`\`) ，StarRocks 会把 `a, b\\, c` 解析成 `a` 和 `b, c` 两个字段值。</li></ul> |

> **说明**
>
> 对于 CSV 格式的数据，需要注意以下两点：
>
> - StarRocks 支持设置长度最大不超过 50 个字节的 UTF-8 编码字符串作为列分隔符，包括常见的逗号 (,)、Tab 和 Pipe (|)。
> - 空值 (null) 用 `\N` 表示。比如，数据文件一共有三列，其中某行数据的第一列、第三列数据分别为 `a` 和 `b`，第二列没有数据，则第二列需要用 `\N` 来表示空值，写作 `a,\N,b`，而不是 `a,,b`。`a,,b` 表示第二列是一个空字符串。
> - `skip_header`、`trim_space`、`enclose` 和 `escape` 等属性在 3.0 及以后版本支持。

#### JSON 适用参数

| **参数名称**      | **是否必选** | **参数说明**                                                 |
| ----------------- | ------------ | ------------------------------------------------------------ |
| jsonpaths         | 否           | 用于指定待导入的字段的名称。仅在使用匹配模式导入 JSON 数据时需要指定该参数。参数取值为 JSON 格式。参见[导入 JSON 数据时配置列映射关系](#导入-json-数据时配置列映射关系).     |
| strip_outer_array | 否           | 用于指定是否裁剪最外层的数组结构。取值范围：`true` 和 `false`。默认值：`false`。真实业务场景中，待导入的 JSON 数据可能在最外层有一对表示数组结构的中括号 `[]`。这种情况下，一般建议您指定该参数取值为 `true`，这样 StarRocks 会剪裁掉外层的中括号 `[]`，并把中括号 `[]` 里的每个内层数组都作为一行单独的数据导入。如果您指定该参数取值为 `false`，则 StarRocks 会把整个 JSON 数据文件解析成一个数组，并作为一行数据导入。例如，待导入的 JSON 数据为 `[ {"category" : 1, "author" : 2}, {"category" : 3, "author" : 4} ]`，如果指定该参数取值为 `true`，则 StarRocks 会把 `{"category" : 1, "author" : 2}` 和 `{"category" : 3, "author" : 4}` 解析成两行数据，并导入到目标 StarRocks 表中对应的数据行。 |
| ignore_json_size | 否   | 用于指定是否检查 HTTP 请求中 JSON Body 的大小。<br />**说明**<br />HTTP 请求中 JSON Body 的大小默认不能超过 100 MB。如果 JSON Body 的大小超过 100 MB，会提示 "The size of this batch exceed the max size [104857600] of json type data data [8617627793]. Set ignore_json_size to skip check, although it may lead huge memory consuming." 错误。为避免该报错，可以在 HTTP 请求头中添加 `"ignore_json_size:true"` 设置，忽略对 JSON Body 大小的检查。 |

另外，导入 JSON 格式的数据时，需要注意单个 JSON 对象的大小不能超过 4 GB。如果 JSON 文件中单个 JSON 对象的大小超过 4 GB，会提示 "This parser can't support a document that big." 错误。

### opt_properties

用于指定一些导入相关的可选参数。指定的参数设置作用于整个导入作业。语法如下：

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

参数说明如下表所述。

| **参数名称**     | **是否必选** | **参数说明**                                                 |
| ---------------- | ------------ | ------------------------------------------------------------ |
| label            | 否           | 用于指定导入作业的标签。如果您不指定标签，StarRocks 会自动为导入作业生成一个标签。相同标签的数据无法多次成功导入，这样可以避免一份数据重复导入。有关标签的命名规范，请参见[系统限制](../../../reference/System_limit.md)。StarRocks 默认保留最近 3 天内成功的导入作业的标签。您可以通过 [FE 配置参数](../../../administration/Configuration.md#导入和导出) `label_keep_max_second` 设置默认保留时长。 |
| where            | 否           | 用于指定过滤条件。如果指定该参数，StarRocks 会按照指定的过滤条件对转换后的数据进行过滤。只有符合 WHERE 子句中指定的过滤条件的数据才会导入。 |
| max_filter_ratio | 否           | 用于指定导入作业的最大容错率，即导入作业能够容忍的因数据质量不合格而过滤掉的数据行所占的最大比例。取值范围：`0`~`1`。默认值：`0` 。<br />建议您保留默认值 `0`。这样的话，当导入的数据行中有错误时，导入作业会失败，从而保证数据的正确性。<br />如果希望忽略错误的数据行，可以设置该参数的取值大于 `0`。这样的话，即使导入的数据行中有错误，导入作业也能成功。<br />**说明**<br />这里因数据质量不合格而过滤掉的数据行，不包括通过 WHERE 子句过滤掉的数据行。 |
| log_rejected_record_num | 否           | 指定最多允许记录多少条因数据质量不合格而过滤掉的数据行数。该参数自 3.1 版本起支持。取值范围：`0`、`-1`、大于 0 的正整数。默认值：`0`。<ul><li>取值为 `0` 表示不记录过滤掉的数据行。</li><li>取值为 `-1` 表示记录所有过滤掉的数据行。</li><li>取值为大于 0 的正整数（比如 `n`）表示每个 BE 节点上最多可以记录 `n` 条过滤掉的数据行。</li></ul> |
| timeout          | 否           | 用于导入作业的超时时间。取值范围：1 ~ 259200。单位：秒。默认值：`600`。<br />**说明**<br />除了 `timeout` 参数可以控制该导入作业的超时时间外，您还可以通过 [FE 配置参数](../../../administration/Configuration.md#导入和导出) `stream_load_default_timeout_second` 来统一控制 Stream Load 导入作业的超时时间。如果指定了`timeout` 参数，则该导入作业的超时时间以 `timeout` 参数为准；如果没有指定 `timeout` 参数，则该导入作业的超时时间以`stream_load_default_timeout_second` 为准。 |
| strict_mode      | 否           | 用于指定是否开严格模式。取值范围：`true` 和 `false`。默认值：`false`。`true` 表示开启，`false` 表示关闭。<br />关于该模式的介绍，参见 [严格模式](../../../loading/load_concept/strict_mode.md)。|
| timezone         | 否           | 用于指定导入作业所使用的时区。默认为东八区 (Asia/Shanghai)。<br />该参数的取值会影响所有导入涉及的、跟时区设置有关的函数所返回的结果。受时区影响的函数有 strftime、alignment_timestamp 和 from_unixtime 等，具体请参见[设置时区](../../../administration/timezone.md)。导入参数 `timezone` 设置的时区对应“[设置时区](../../../administration/timezone.md)”中所述的会话级时区。 |
| load_mem_limit   | 否           | 导入作业的内存限制，最大不超过 BE 的内存限制。单位：字节。默认内存限制为 2 GB。 |
| merge_condition  | 否           | 用于指定作为更新生效条件的列名。这样只有当导入的数据中该列的值大于等于当前值的时候，更新才会生效。StarRocks v2.5 起支持条件更新。参见[通过导入实现数据变更](../../../loading/Load_to_Primary_Key_tables.md)。 <br/>**说明**<br/>指定的列必顈为非主键列，且仅主键模型表支持条件更新。  |

## 列映射

### 导入 CSV 数据时配置列映射关系

如果源数据文件中的列与目标表中的列按顺序一一对应，您不需要指定列映射和转换关系。

如果源数据文件中的列与目标表中的列不能按顺序一一对应，包括数量或顺序不一致，则必须通过 `COLUMNS` 参数来指定列映射和转换关系。一般包括如下两种场景：

- **列数量一致、但是顺序不一致，并且数据不需要通过函数计算、可以直接落入目标表中对应的列。** 这种场景下，您需要在 `COLUMNS` 参数中按照源数据文件中的列顺序、使用目标表中对应的列名来配置列映射和转换关系。

  例如，目标表中有三列，按顺序依次为 `col1`、`col2` 和 `col3`；源数据文件中也有三列，按顺序依次对应目标表中的 `col3`、`col2` 和 `col1`。这种情况下，需要指定 `COLUMNS(col3, col2, col1)`。

- **The number and order of columns are inconsistent, and the data in some columns needs to be calculated by a function before being mapped to the corresponding columns in the target table.** In this scenario, you not only need to configure the column mapping relationship in the `COLUMNS` parameter according to the column order in the source data file and use the corresponding column names in the target table, but also need to specify the functions involved in data calculation. Here are two examples:

  - There are three columns in the target table, namely `col1`, `col2`, and `col3` in order; the source data file has four columns, and the first three columns correspond to the `col1`, `col2`, and `col3` in the target table in order, with the fourth column having no corresponding column in the target table. In this case, you need to specify `COLUMNS(col1, col2, col3, temp)`, where the last column can be arbitrarily specified with a name (such as `temp`) for a placeholder.
  - There are three columns in the target table, namely `year`, `month`, and `day`. The source data file only contains a column with time data in the format of `yyyy-mm-dd hh:mm:ss`. In this case, you can specify `COLUMNS(col, year = year(col), month=month(col), day=day(col))`. Where `col` is the temporary name of the column contained in the source data file, and `year = year(col)`, `month=month(col)`, and `day=day(col)` are used to specify the extraction of corresponding data from the `col` column in the source data file and map it to the corresponding columns in the target table, e.g., `year = year(col)` indicates extracting the `yyyy` part of the data in the `col` column in the source data file using the `year` function and mapping it to the `year` column in the target table.

For examples of operations, refer to [Setting Column Mapping Relationships](#setting-column-mapping-relationships).

### Configuring Column Mapping Relationships When Importing JSON Data

If the key names in the JSON file are the same as the column names in the target table, you can use the simple mode to import data. In simple mode, the `jsonpaths` parameter does not need to be set. This mode requires JSON data to be in the form of curly braces {} representing objects, for example, in `{"category": 1, "author": 2, "price": "3"}`, `category`, `author`, and `price` are the names of the keys, which directly correspond to the three columns `category`, `author`, and `price` in the target table by name.

If the key names in the JSON file are not the same as the column names in the target table, you need to use the matching mode to import data. In matching mode, you need to specify the mapping and conversion relationship between the keys in the JSON file and the columns in the target table through the `jsonpaths` and `COLUMNS` parameters:

- The `jsonpaths` parameter specifies the keys to be imported in the order of the keys in the JSON file.
- The `COLUMNS` parameter specifies the mapping and conversion relationship between the keys in the JSON file and the columns in the target table.
  - The column names specified in the `COLUMNS` parameter correspond one-to-one in order with the keys specified in the `jsonpaths` parameter.
  - The column names specified in the `COLUMNS` parameter correspond one-to-one by name with the columns in the target table.

For examples of importing JSON data using matching mode, refer to [Importing Data Using Matching Mode](#importing-data-using-matching-mode).

## Return Value

After the import is completed, the result information of the import job will be returned in JSON format as follows:

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
```

The parameters in the returned result are described in the following table.

| **Parameter Name**      | **Description**                                              |
| ----------------------- | ------------------------------------------------------------ |
| TxnId                   | The transaction ID of the import job.                        |
| Label                   | The label of the import job.                                 |
| Status                  | The final status of the imported data.<ul><li>`Success`: Indicates that the data import was successful and the data is visible.</li><li>`Publish Timeout`: Indicates that the import job has been successfully submitted, but the data cannot be immediately visible for some reason. It can be regarded as a success and does not need to be retried.</li><li>`Label Already Exists`: Indicates that the label has been used by other import jobs. The data may have been imported successfully, or it may be currently importing.</li><li>`Fail`: Indicates that the data import has failed. You can specify the label for retrying the import job.</li></ul> |
| Message                 | Details of the status of the import job. If the import job fails, the specific reason for the failure will be returned here. |
| NumberTotalRows         | The total number of rows read.                               |
| NumberLoadedRows        | The total number of rows successfully imported. Only valid when the `Status` in the return result is `Success`. |
| NumberFilteredRows      | The number of rows filtered out during the import due to data quality issues. |
| NumberUnselectedRows    | The number of rows filtered out during the import based on the conditions specified by the WHERE clause. |
| LoadBytes               | The amount of data imported in this import. Unit: bytes.     |
| LoadTimeMs              | The time taken for this import. Unit: milliseconds.          |
| BeginTxnTimeMs          | The time it takes for the import job to start the transaction. |
| StreamLoadPlanTimeMs    | The time it takes for the import job to generate the execution plan. |
| ReadDataTimeMs          | The time it takes for the import job to read the data.        |
| WriteDataTimeMs         | The time it takes for the import job to write the data.       |
| CommitAndPublishTimeMs  | The time it takes for the import job to commit and publish the data. |

If the import job fails, an `ErrorURL` will also be returned, as shown below:

```JSON
{
    "ErrorURL": "http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be"
}
```

You can view the specific information of the error data rows filtered out during the import due to data quality issues through `ErrorURL`. When submitting the import job, you can specify the maximum number of error data rows to be recorded through the optional parameter `log_rejected_record_num`.

You can directly view the information of error data rows by executing the `curl "url"` command. You can also export the information of error data rows by executing the `wget "url"` command, as shown below:

```Bash
wget http://172.26.195.68:8045/api/_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be
```

The exported information of the error data rows will be saved to a local file named `_load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be`. You can view the content of this file by executing the `cat _load_error_log?file=error_log_3a4eb8421f0878a6_9a54df29fd9206be` command.

You can adjust the import job based on the error information and then resubmit the import job.

## Example

### Importing Data in CSV Format

本小节以 CSV 格式的数据为例，重点阐述在创建导入作业的时候，如何运用各种参数配置来满足不同业务场景下的各种导入要求。

#### **设置超时时间**

StarRocks 数据库 `test_db` 里的表 `table1` 包含三列，按顺序依次为 `col1`、`col2`、`col3`。

数据文件 `example1.csv` 也包含三列，按顺序一一对应 `table1` 中的三列 `col1`、`col2`、`col3`。

如果要把 `example1.csv` 中所有的数据都导入到 `table1` 中，并且要求超时时间最大不超过 100 秒，可以执行如下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label1" \
    -H "Expect:100-continue" \
    -H "timeout:100" \
    -H "max_filter_ratio:0.2" \
    -T example1.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table1/_stream_load
```

#### **设置最大容错率**

StarRocks 数据库 `test_db` 里的表 `table2` 包含三列，按顺序依次为 `col1`、`col2`、`col3`。

数据文件 `example2.csv` 也包含三列，按顺序一一对应 `table2` 中的三列 `col1`、`col2`、`col3`。

如果要把 `example2.csv` 中所有的数据都导入到 `table2` 中，并且要求容错率最大不超过 `0.2`，可以执行如下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label2" \
    -H "Expect:100-continue" \
    -H "max_filter_ratio:0.2" \
    -T example2.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table2/_stream_load
```

#### **设置列映射关系**

StarRocks 数据库 `test_db` 里的表 `table3` 包含三列，按顺序依次为 `col1`、`col2`、`col3`。

数据文件 `example3.csv` 也包含三列，按顺序依次对应 `table3` 中 `col2`、`col1`、`col3`。

如果要把 `example3.csv` 中所有的数据都导入到 `table3` 中，可以执行如下命令：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label3" \
    -H "Expect:100-continue" \
    -H "columns: col2, col1, col3" \
    -T example3.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table3/_stream_load
```

> **说明**
>
> 上述示例中，因为 `example3.csv` 和 `table3` 所包含的列不能按顺序依次对应，因此需要通过 `columns` 参数来设置 `example3.csv` 和 `table3` 之间的列映射关系。

#### **设置筛选条件**

StarRocks 数据库 `test_db` 里的表 `table4` 包含三列，按顺序依次为 `col1`、`col2`、`col3`。

数据文件 `example4.csv` 也包含三列，按顺序一一对应 `table4` 中的三列 `col1`、`col2`、`col3`。

如果只想把 `example4.csv` 中第一列的值等于 `20180601` 的数据行导入到 `table4` 中，可以执行如下命令：

```Bash
curl --location-trusted -u <username>:<password> -H "label:label4" \
    -H "Expect:100-continue" \
    -H "columns: col1, col2，col3]"\
    -H "where: col1 = 20180601" \
    -T example4.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table4/_stream_load
```

> **说明**
>
> 上述示例中，虽然 `example4.csv` 和 `table4` 所包含的列数目相同、并且按顺序一一对应，但是因为需要通过 WHERE 子句指定基于列的过滤条件，因此需要通过 `columns` 参数对 `example4.csv` 中的列进行临时命名。

#### **设置目标分区**

StarRocks 数据库 `test_db` 里的表 `table5` 包含三列，按顺序依次为 `col1`、`col2`、`col3`。

数据文件 `example5.csv` 也包含三列，按顺序一一对应 `table5` 中的三列 `col1`、`col2`、`col3`。

如果要把 `example5.csv` 中所有的数据都导入到 `table5` 所在的分区 `p1` 和 `p2`，可以执行如下命令：

```Bash
curl --location-trusted -u <username>:<password>  -H "label:label5" \
    -H "Expect:100-continue" \
    -H "partitions: p1, p2" \
    -T example5.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table5/_stream_load
```

#### **设置严格模式和时区**

StarRocks 数据库 `test_db` 里的表 `table6` 包含三列，按顺序依次为 `col1`、`col2`、`col3`。

数据文件 `example6.csv` 也包含三列，按顺序一一对应 `table6` 中的三列 `col1`、`col2`、`col3`。

如果要把 `example6.csv` 中所有的数据导入到 `table6` 中，并且要求进严格模式的过滤、使用时区 `Africa/Abidjan`，可以执行如下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "strict_mode: true" \
    -H "timezone: Africa/Abidjan" \
    -T example6.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table6/_stream_load
```

#### **导入数据到含有 HLL 类型列的表**

StarRocks 数据库 `test_db` 里的表 `table7` 包含两个 HLL 类型的列，按顺序依次为 `col1`、`col2`。

数据文件 `example7.csv` 也包含两列，第一列对应 `table7` 中  HLL 类型的列`col1`，可以通过函数转换成 HLL 类型的数据并落入 `col1` 列；第二列跟 `table7` 中任何一列都不对应。

如果要把 `example7.csv` 中对应的数据导入到 `table7` 中，可以执行如下命令：

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=hll_hash(temp1), col2=hll_empty()" \
    -T example7.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table7/_stream_load
```

> **说明**
>
> 上述示例中，通过 `columns` 参数，把 `example7.csv` 中的两列临时命名为 `temp1`、`temp2`，然后使用函数指定数据转换规则，包括：
>
> - 使用 `hll_hash` 函数把 `example7.csv` 中的 `temp1` 列转换成 HLL 类型的数据并映射到 `table7`中的 `col1` 列。
> - 使用 `hll_empty` 函数给导入的数据行在 `table7` 中的第二列补充默认值。

Please refer to [hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) and [hll_empty](../../sql-functions/aggregate-functions/hll_empty.md) for the usage of the `hll_hash` and `hll_empty` functions.

#### **Importing Data into a Table with BITMAP Type Columns**

The table `table8` in the StarRocks database `test_db` contains two columns of BITMAP type, namely `col1` and `col2` in order.

The data file `example8.csv` also contains two columns, where the first column corresponds to the BITMAP type column `col1` in `table8` and can be transformed into BITMAP type data using a function and saved into the `col1` column. The second column does not correspond to any column in `table8`.

To import the data from `example8.csv` into `table8`, you can execute the following command:

```Bash
curl --location-trusted -u <username>:<password> \
    -H "Expect:100-continue" \
    -H "columns: temp1, temp2, col1=to_bitmap(temp1), col2=bitmap_empty()" \
    -T example8.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/table8/_stream_load
```

> **Note**
>
> In the above example, the `columns` parameter is used to temporarily name the two columns in `example8.csv` as `temp1` and `temp2`. Then, the data transformation rules are specified using functions as follows:
>
> - Use the `to_bitmap` function to transform the `temp1` column in `example8.csv` into BITMAP type data and save it into the `col1` column in `table8`.
>
> - Use the `bitmap_empty` function to supply a default value for the second column in the imported data in `table8`.

Please refer to [to_bitmap](../../sql-functions/bitmap-functions/to_bitmap.md) and [bitmap_empty](../../sql-functions/bitmap-functions/bitmap_empty.md) for the usage of the `to_bitmap` and `bitmap_empty` functions.

#### Setting `skip_header`, `trim_space`, `enclose` and `escape`

The table `table9` in the StarRocks database `test_db` contains three columns, namely `col1`, `col2`, and `col3` in order.

The data file `example9.csv` also contains three columns which correspond to `col2`, `col1`, and `col3` in `table9` respectively.

If you want to import all the data from `example9.csv` into `table9`, and skip the first five rows of data in `example9.csv`, remove leading and trailing spaces of the column delimiter, specify the `enclose` character as backslash (`\`), and specify the `escape` character also as backslash (`\`), you can execute the following command:

```Bash
curl --location-trusted -u <username>:<password> -H "label:3875" \
    -H "Expect:100-continue" \
    -H "trim_space: true" -H "skip_header: 5" \
    -H "column_separator:," -H "enclose:\"" -H "escape:\\" \
    -H "columns: col2, col1, col3" \
    -T example9.csv -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl9/_stream_load
```

### **Importing Data in JSON Format**

This section mainly describes some parameter configurations that need to be considered when importing data in JSON format.

The table `tbl1` in the StarRocks database `test_db` has the following table structure:

```SQL
`category` varchar(512) NULL COMMENT "",
`author` varchar(512) NULL COMMENT "",
`title` varchar(512) NULL COMMENT "",
`price` double NULL COMMENT ""
```

#### **Importing Data using Simple Mode**

Suppose the data file `example1.json` contains the following data:

```JSON
{"category":"C++","author":"avc","title":"C++ primer","price":895}
```

You can import the data from `example1.json` into `tbl1` with the following command:

```Bash
curl --location-trusted -u <username>:<password> -H "label:label6" \
    -H "Expect:100-continue" \
    -H "format: json" \
    -T example1.json -XPUT \
    http://<fe_host>:<fe_http_port>/api/test_db/tbl1/_stream_load
```

> **Note**
>
> As shown in the above example, when `columns` and `jsonpaths` parameters are not specified, the fields in the JSON data file will be matched with the column names in the StarRocks table.

To increase throughput, Stream Load supports importing multiple data entries at once. For example, you can import the following multiple data entries from a JSON data file at once:

```JSON
[
    {"category":"C++","author":"avc","title":"C++ primer","price":89.5},
    {"category":"Java","author":"avc","title":"Effective Java","price":95},
    {"category":"Linux","author":"avc","title":"Linux kernel","price":195}
]
```

#### **Importing Data using Matching Mode**

StarRocks matches and processes the data in the following order:

1. (Optional) Use the `strip_outer_array` parameter to trim the outermost array structure.

    > **Note**
    >
    > This is only relevant when the JSON data has an outermost array structure represented by square brackets `[]`. In this case, set `strip_outer_array` to `true`.

2. (Optional) Match the root node of the JSON data according to the `json_root` parameter.

    > **Note**
    >
    > This is only relevant when the JSON data has a root node. In this case, specify the root node using the `json_root` parameter.

3. Extract the JSON data to be imported according to the `jsonpaths` parameter.

##### **Importing Data without Specifying the JSON Root Node using Matching Mode**

Suppose the data file `example2.json` contains the following data:

```JSON
[
    {"category":"xuxb111","author":"1avc","title":"SayingsoftheCentury","price":895},
    {"category":"xuxb222","author":"2avc","title":"SayingsoftheCentury","price":895},
    {"category":"xuxb333","author":"3avc","title":"SayingsoftheCentury","price":895}
]
```

You can use the `jsonpaths` parameter to precisely import data, for example, importing only the `category`, `author`, and `price` fields:

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

> **Note**
>
> In the above example, the outermost structure of the JSON data is an array represented by square brackets `[]`, and each JSON object in the array represents a data record. Therefore, `strip_outer_array` is set to `true` to trim the outermost array structure. The fields not specified in the import process, such as `title`, will be ignored.

##### **Importing Data specifying the JSON Root Node using Matching Mode**

Suppose the data file `example3.json` contains the following data:

```JSON
{
    "id": 10001,
    "RECORDS":[
        {"category":"11","title":"SayingsoftheCentury","price":895,"timestamp":1589191587},
        {"category":"22","author":"2avc","price":895,"timestamp":1589191487},
        {"category":"33","author":"3avc","title":"SayingsoftheCentury","timestamp":1589191387}
    ],
    "comments": ["3 records", "there will be 3 rows"]
}
```

您可以通过指定 `jsonpaths` 进行精准导入，例如只导入 `category`、`author`、`price` 三个字段的数据：

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

> **说明**
>
> 上述示例中，JSON 数据的最外层是一个通过中括号 [] 表示的数组结构，并且数组结构中的每个 JSON 对象都表示一条数据记录。因此，需要设置 `strip_outer_array` 为 `true`来裁剪最外层的数组结构。导入过程中，未指定的字段 `title` 和 `timestamp` 会被忽略掉。另外，示例中还通过 `json_root` 参数指定了需要真正导入的数据为 `RECORDS` 字段对应的值，即一个 JSON 数组。