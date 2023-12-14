---
displayed_sidebar: "Chinese"
---

# 代理商Load

import InsertPrivNote from '../../../assets/commonMarkdown/insertPrivNote.md'

## 描述

StarRocks提供了基于MySQL的加载方法Broker Load。在提交加载作业后，StarRocks会异步运行该作业。您可以使用`SELECT * FROM information_schema.loads'来查询作业结果。此功能从v3.1版本开始支持。有关背景信息、原理、支持的数据文件格式、如何执行单表加载和多表加载以及如何查看作业结果的更多信息，请参见[从HDFS加载数据](../../../loading/hdfs_load.md) 和[从云存储加载数据](../../../loading/cloud_storage_load.md)。

<InsertPrivNote />

## 语法

```SQL
LOAD LABEL [<database_name>.]<label_name>
(
    data_desc[, data_desc ...]
)
WITH BROKER
(
    StorageCredentialParams
)
[PROPERTIES
(
    opt_properties
)
]
```

请注意，在StarRocks中，一些文字用作SQL语言的保留关键字。请勿直接在SQL语句中使用这些关键字。如果要在SQL语句中使用此类关键字，请将其括在一对重音符（`）中。请参见[关键字](../../../sql-reference/sql-statements/keywords.md)。

## 参数

### database_name 和 label_name

`label_name`指定加载作业的标签。

`database_name`可选地指定目标表所属的数据库的名称。

每个加载作业都有一个标签，在整个数据库中是唯一的。您可以使用加载作业的标签查看加载作业的执行状态，并防止重复加载相同的数据。当加载作业进入 **FINISHED** 状态时，其标签不可再次使用。只有已进入 **CANCELLED** 状态的加载作业的标签才能被重用。在大多数情况下，加载作业的标签是用来重试该加载作业并加载相同数据，从而实现仅一次语义。

有关标签命名约定，请参见[系统限制](../../../reference/System_limit.md)。

### data_desc

要加载的一批数据的描述。每个 `data_desc` 描述符声明了数据源、ETL函数、目标 StarRocks 表和目标分区等信息。

Broker Load支持一次加载多个数据文件。在一个加载作业中，您可以使用多个 `data_desc` 描述符来声明要加载的多个数据文件，或者使用一个 `data_desc` 描述符来声明要从中加载所有数据文件的一个文件路径。Broker Load还可以确保运行以加载多个数据文件的每个加载作业的事务原子性。原子性指的是一个加载作业中多个数据文件的加载必须全部成功或全部失败。绝不会发生某些数据文件的加载成功而其他文件的加载失败。

`data_desc` 支持以下语法：

```SQL
DATA INFILE ("<file_path>"[, "<file_path>" ...])
[NEGATIVE]
INTO TABLE <table_name>
[PARTITION (<partition1_name>[, <partition2_name> ...])]
[TEMPORARY PARTITION (<temporary_partition1_name>[, <temporary_partition2_name> ...])]
[COLUMNS TERMINATED BY "<column_separator>"]
[ROWS TERMINATED BY "<row_separator>"]
[FORMAT AS "CSV | Parquet | ORC"]
[(fomat_type_options)]
[(column_list)]
[COLUMNS FROM PATH AS (<partition_field_name>[, <partition_field_name> ...])]
[SET <k1=f1(v1)>[, <k2=f2(v2)> ...]]
[WHERE predicate]
```

`data_desc` 必须包含以下参数：

- `file_path`

  指定要加载的一个或多个数据文件的保存路径。

  您可以将此参数指定为一个数据文件的保存路径。例如，您可以将此参数指定为`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/20210411"`，以从HDFS服务器上的路径`/user/data/tablename`加载名为`20210411`的数据文件。

  您还可以使用通配符 `?`、`*`、`[]`、`{}` 或 `^` 来指定多个数据文件的保存路径。请参见[通配符参考](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html#globStatus-org.apache.hadoop.fs.Path-)。例如，您可以将此参数指定为`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/*/*"`或`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/dt=202104*/*"`，以从HDFS服务器上的路径`/user/data/tablename`中的所有分区或仅`202104`分区加载数据文件。

  > **注意**
  >
  > 通配符也可用于指定中间路径。

  在前面的示例中，`hdfs_host` 和 `hdfs_port` 参数描述如下：

  - `hdfs_host`：HDFS集群中NameNode主机的IP地址。

  - `hdfs_port`：HDFS集群中NameNode主机的FS端口。默认端口号为`9000`。

  > **注意**
  >
  > - Broker Load支持根据S3或S3A协议访问AWS S3。因此，在从AWS S3加载数据时，您可以包含`s3://`或`s3a://`作为文件路径所传递的S3 URI的前缀。
  > - Broker Load仅支持根据gs协议访问Google GCS。因此，在从Google GCS加载数据时，您必须在GCS URI的前缀中包含gs://。
  > - 在从Blob Storage加载数据时，您必须使用wasb或wasbs协议来访问您的数据：
  >   - 如果您的存储帐户允许通过HTTP访问，请使用wasb协议，并将文件路径写为`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`。
  >   - 如果您的存储帐户允许通过HTTPS访问，请使用wasbs协议，并将文件路径写为`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>`。
  > - 在从Data Lake Storage Gen1加载数据时，您必须使用adl协议来访问您的数据，并将文件路径写为`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`。
  > - 在从Data Lake Storage Gen2加载数据时，您必须使用abfs或abfss协议来访问您的数据：
  >   - 如果您的存储帐户允许通过HTTP访问，请使用abfs协议，并将文件路径写为`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。
  >   - 如果您的存储帐户允许通过HTTPS访问，请使用abfss协议，并将文件路径写为`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

- `INTO TABLE`

  指定目标StarRocks表的名称。

`data_desc` 还可以可选地包括以下参数：

- `NEGATIVE`

  撤销特定批次数据的加载。要实现这一点，您需要使用指定了`NEGATIVE`关键字的相同批次数据进行加载。

  > **注意**
  >
  > 仅当StarRocks表是聚合表并且其所有值列都由`sum`函数计算时，此参数才有效。

- `PARTITION`

  指定要加载数据的分区。默认情况下，如果未指定此参数，则源数据将加载到StarRocks表的所有分区中。

- `TEMPORARY PARTITION`

  指定要加载数据的[临时分区](../../../table_design/Temporary_partition.md)的名称。您可以指定多个临时分区，它们必须用逗号（,）分隔。

- `COLUMNS TERMINATED BY`

  指定数据文件中使用的列分隔符。默认情况下，如果不指定此参数，则此参数默认为`\t`，表示制表符。使用此参数指定的列分隔符必须与实际在数据文件中使用的列分隔符相同。否则，由于数据质量不足，加载作业将失败，并且其`State`将为`CANCELLED`。

  Broker Load作业是根据MySQL协议提交的。StarRocks和MySQL都会在加载请求中转义字符。因此，如果列分隔符是制表符等不可见字符，您必须在列分隔符之前添加反斜杠（\）。例如，如果列分隔符是`\t`，则必须输入`\\t`，如果列分隔符是`\n`，则必须输入`\\n`。Apache Hive™文件使用`\x01`作为列分隔符，因此如果数据文件来自Hive，则必须输入`\\x01`。

  > **注意**
  >
  > - 对于CSV数据，您可以使用不超过50字节的UTF-8字符串，例如逗号（,）、制表符或竖线（|）作为文本分隔符。
- 空值使用`\N`来表示。例如，一个数据文件包含三列，其中一个记录在数据文件中的第一列和第三列包含数据，但是第二列没有数据。在这种情况下，需要使用`\N`来表示第二列的空值。这意味着记录必须编写为`a,\N,b`，而不是`a,,b`。`a,,b`表示记录的第二列包含一个空字符串。

- `ROWS TERMINATED BY`

  指定数据文件中使用的行分隔符。默认情况下，如果您没有指定此参数，该参数默认为`\n`，表示换行符。您使用此参数指定的行分隔符必须与数据文件中实际使用的行分隔符相同。否则，由于数据质量不足，加载作业将失败，其`State`将为`CANCELLED`。从v2.5.4版本开始支持此参数。

  有关行分隔符的用法说明，请参阅前面的`COLUMNS TERMINATED BY`参数的用法说明。

- `FORMAT AS`

  指定数据文件的格式。有效值: `CSV`、`Parquet` 和 `ORC`。默认情况下，如果您没有指定此参数，StarRocks将根据`file_path`参数指定的文件扩展名 **.csv**、**.parquet** 或 **.orc** 来确定数据文件格式。

- `format_type_options`

  当`FORMAT AS`设置为`CSV`时，指定CSV格式选项的语法为：

  ```JSON
  (
      key = value
      key = value
      ...
  )
  ```

  > **注**
  >
  > 从v3.0版本开始支持`format_type_options`。

  以下表格描述了各选项。

  | **参数**     | **描述**                                                   |
  | ------------ | ---------------------------------------------------------- |
  | skip_header  | 指定在数据文件为CSV格式时是否跳过数据文件的前几行。类型: INTEGER。默认值: `0`。<br />在某些CSV格式的数据文件中，开头的前几行用于定义元数据，如列名和列数据类型。通过设置`skip_header`参数，您可以使StarRocks在数据加载过程中跳过数据文件的前几行。例如，如果您将此参数设置为`1`，则StarRocks在数据加载过程中跳过数据文件的第一行。<br />数据文件中的前几行必须使用您在加载语句中指定的行分隔符进行分隔。 |
  | trim_space   | 指定在数据文件为CSV格式时是否移除数据文件中列分隔符前后的空格。类型: BOOLEAN。默认值: `false`。<br />对于某些数据库，在将数据导出为CSV格式的数据文件时会添加空格到列分隔符。此类空格根据其位置被称为前导空格或尾随空格。通过设置`trim_space`参数，您可以使StarRocks在数据加载过程中移除此类不必要的空格。<br />请注意，StarRocks不会去除使用一对`enclose`指定字符包裹的字段中的空格（包括前导空格和尾随空格）。例如，以下字段值使用竖线 (<code class="language-text">&#124;</code>) 作为列分隔符，双引号(`"`) 作为`enclose`指定字符：<br /><code class="language-text">&#124;"爱StarRocks"&#124;</code> <br /><code class="language-text">&#124;" 爱StarRocks "&#124;</code> <br /><code class="language-text">&#124; "爱StarRocks" &#124;</code> <br />如果将`trim_space`设置为`true`，则StarRocks将处理前面的字段值如下：<br /><code class="language-text">&#124;"爱StarRocks"&#124;</code> <br /><code class="language-text">&#124;" 爱StarRocks "&#124;</code> <br /><code class="language-text">&#124;"爱StarRocks"&#124;</code> |
  | enclose      | 当数据文件为CSV格式时，指定用于根据[RF C4180](https://www.rfc-editor.org/rfc/rfc4180)包裹数据文件中的字段值的字符。类型: 单字节字符。默认值: `NONE`。最常见的字符是单引号(`'`)和双引号(`"`)。 所有使用`enclose`指定字符包裹的特殊字符（包括行分隔符和列分隔符）都被认为是普通符号。StarRocks不仅遵循RFC4180并允许您指定任何单字节字符作为`enclose`指定字符。如果字段值包含一个`enclose`指定字符，您可以使用相同的字符来转义该`enclose`指定字符。例如，将`enclose`设置为`"`，字段值为`a "quoted" c`。在此情况下，您可以在数据文件中输入字段值`"a ""quoted"" c"`来转义数据文件中的字段值为`"a "quoted" c"`。 |
  | escape       | 指定用于转义各种特殊字符（如行分隔符、列分隔符、转义字符和`enclose`指定字符）的字符。这些特殊字符在数据文件中将被StarRocks视为普通字符并作为它们所在的字段值的一部分进行解析。类型: 单字节字符。默认值: `NONE`。最常见的字符是斜杠（`\`），在SQL语句中必须写作双斜杠（`\\`）。<br />**注**<br />由`escape`指定的字符会应用于`enclose`指定字符的内部和外部。以下是两个示例：<ul><li>当您将`enclose`设置为`"`，并将`escape`设置为`\`时，StarRocks会将`"说\"Hello world\""`解析为`说"Hello world"`。</li><li>假设列分隔符是逗号（`,`）。当您将`escape`设置为`\`时，StarRocks将`a, b\, c`解析为两个独立的字段值：`a` 和 `b, c`。</li></ul> |

- `column_list`

  指定数据文件和StarRocks表之间的列映射。语法: `(<column_name>[, <column_name> ...])`。在`column_list`中声明的列将根据名称映射到StarRocks表的列。

  > **注**
  >
  > 如果数据文件的列按顺序映射到StarRocks表的列，则不需要指定`column_list`。

  如果要跳过数据文件的特定列，只需将该列暂时命名为与任何StarRocks表列不同的名称。更多信息，请参见[加载时数据转换](../../../loading/Etl_in_loading.md)。

- `COLUMNS FROM PATH AS`

  从您指定的文件路径中提取一个或多个分区字段的信息。此参数仅在文件路径包含分区字段时有效。

  例如，如果数据文件存储在路径 `/path/col_name=col_value/file1` 中，其中 `col_name` 是一个分区字段，并且可以映射到StarRocks表的某一列，您可以将此参数指定为 `col_name`。这样，StarRocks从路径中提取 `col_value` 值，并将其加载到与`col_name`映射到的StarRocks表列中。

  > **注**
  >
  > 仅当从HDFS加载数据时才可用。

- `SET`

  指定您要使用的一个或多个函数来转换数据文件的列。示例:

  - StarRocks表由三列组成，依次为 `col1`、`col2` 和 `col3`。数据文件包含四列，其中前两列按顺序映射到StarRocks表的 `col1` 和 `col2`，最后两列的和映射到StarRocks表的 `col3`。在这种情况下，您需要将`column_list`指定为`(col1,col2,tmp_col3,tmp_col4)`，并在SET子句中指定`(col3=tmp_col3+tmp_col4)`来实现数据转换。
  - StarRocks表由三列组成，依次为 `year`、`month` 和 `day`。数据文件仅包含一个列，其中包含`yyyy-mm-dd hh:mm:ss`格式的日期和时间值。在这种情况下，您需要将`column_list`指定为`(tmp_time)`，并在SET子句中指定`(year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))`来实现数据转换。

- `WHERE`

  指定基于其条件要过滤源数据的条件。StarRocks仅加载符合WHERE子句中指定的筛选条件的源数据。

### WITH BROKER

在v2.3及更早版本，输入`WITH BROKER "<broker_name>"`来指定要使用的经纪人。从v2.5版本开始，您不再需要指定经纪人，但仍然需要保留`WITH BROKER`关键字。

> **注**
>
v2.4及之前版本，StarRocks在运行 Broker Load 任务时，依赖于 Broker 来建立StarRocks集群和外部存储系统之间的连接。这称为“基于 Broker 的加载”。Broker 是与文件系统接口集成的独立状态无关服务。使用 Broker，StarRocks可以访问和读取存储在外部存储系统中的数据文件，并可以利用自身的计算资源来预处理和加载这些数据文件的数据。

从v2.5开始，StarRocks删除了对 Broker 的依赖，并实施了“无 Broker 加载”。

您可以使用 [SHOW BROKER](../Administration/SHOW_BROKER.md) 语句来检查在您的 StarRocks 集群中部署的 Broker。如果没有部署 Broker，则可以按照 [部署 Broker](../../../deployment/deploy_broker.md) 中提供的说明部署 Broker。

### StorageCredentialParams

StarRocks用于访问您的存储系统的身份验证信息。

#### HDFS

开源 HDFS 支持两种认证方法：简单认证和 Kerberos 认证。Broker Load 默认使用简单认证。开源 HDFS 还支持为 NameNode 配置 HA 机制。如果您选择开源 HDFS 作为您的存储系统，则可以按照以下方式指定认证配置和 HA 配置：

- 认证配置

  - 如果使用简单认证，请按以下方式配置 `StorageCredentialParams`：

    ```Plain
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
    ```

    以下表格描述了 `StorageCredentialParams` 中的参数。

    | 参数                        | 描述                                             |
    | -------------------------- | ------------------------------------------------- |
    | hadoop.security.authentication | 认证方法。有效值：`simple` 和 `kerberos`。默认值：`simple`。`simple` 表示简单认证，即无认证；`kerberos` 表示 Kerberos 认证。 |
    | username                   | 要用于访问 HDFS 集群的 NameNode 的帐户的用户名。 |
    | password                   | 要用于访问 HDFS 集群的 NameNode 的帐户的密码。    |

  - 如果使用 Kerberos 认证，请按以下方式配置 `StorageCredentialParams`：

    ```Plain
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "nn/zelda1@ZELDA.COM",
    "kerberos_keytab" = "/keytab/hive.keytab",
    "kerberos_keytab_content" = "YWFhYWFh"
    ```

    以下表格描述了 `StorageCredentialParams` 中的参数。

    | 参数                        | 描述                                             |
    | -------------------------- | ------------------------------------------------- |
    | hadoop.security.authentication | 认证方法。有效值：`simple` 和 `kerberos`。默认值：`simple`。`simple` 表示简单认证，即无认证；`kerberos` 表示 Kerberos 认证。 |
    | kerberos_principal         | 要认证的 Kerberos principal。每个 principal 需包含以下三部分，以确保它在 HDFS 集群中是唯一的：<ul><li>`username`或`servicename`：principal 的名称。</li><li>`instance`：托管要在 HDFS 集群中认证的节点的服务器名称。服务器名称有助于确保 principal 是唯一的，例如，当 HDFS 集群由多个独立认证的 DataNode 组成时。</li><li>`realm`：领域名称。领域名称必须大写。示例：`nn/[zelda1@ZELDA.COM](mailto:zelda1@ZELDA.COM)`。</li></ul> |
    | kerberos_keytab            | Kerberos keytab 文件的保存路径。                      |
    | kerberos_keytab_content    | Kerberos keytab 文件的 Base64 编码内容。您可以选择指定 `kerberos_keytab` 或 `kerberos_keytab_content` 中的一个。 |

    如果您已配置多个 Kerberos 用户，请确保部署了至少一个独立的 [broker 分组](../../../deployment/deploy_broker.md)，在 load 语句中必须输入 `WITH BROKER "<broker_name>"` 以指定您要使用的 broker 分组。此外，您必须打开 broker 启动脚本文件 **start_broker.sh**，并修改该文件的第 42 行以启用 brokers 读取 **krb5.conf** 文件。例如：

    ```Plain
    export JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx1024m -Dfile.encoding=UTF-8 -Djava.security.krb5.conf=/etc/krb5.conf"
    ```

    > **注意**
    >
    > - 在上述示例中，`/etc/krb5.conf` 可以替换为 **krb5.conf** 文件的实际保存路径。确保该 broker 对该文件具有读取权限。如果 broker 分组由多个 broker 组成，您必须在每个 broker 节点上修改 **start_broker.sh** 文件，然后重新启动 broker 节点以使修改生效。
    > - 您可以使用 [SHOW BROKER](../Administration/SHOW_BROKER.md) 语句来检查在您的 StarRocks 集群中部署的 Broker。

- HA 配置

  您可以为 HDFS 集群的 NameNode 配置 HA 机制。这样，如果 NameNode 切换至另一个节点，StarRocks可以自动识别作为新的 NameNode 服务的新节点。情景包括以下内容：

  - 如果您从配置了单个 Kerberos 用户的单个 HDFS 集群中加载数据，既支持基于 broker 的加载，也支持基于非 broker 的加载。
  
    - 要执行基于 broker 的加载，请确保部署了至少一个独立的 [broker 分组](../../../deployment/deploy_broker.md)，并将 `hdfs-site.xml` 文件放置在作为 HDFS 集群 NameNode 服务的 broker 节点的 `{deploy}/conf` 路径下。StarRocks将在 broker 启动时将 `{deploy}/conf` 路径添加到环境变量 `CLASSPATH` 中，从而允许 brokers 读取有关 HDFS 集群节点的信息。
  
    - 要执行基于非 broker 的加载，请将 `hdfs-site.xml` 文件放置在每个 FE 节点和每个 BE 节点的 `{deploy}/conf` 路径下。

  - 如果您从配置了多个 Kerberos 用户的单个 HDFS 集群中加载数据，仅支持基于 broker 的加载。请确保部署了至少一个独立的 [broker 分组](../../../deployment/deploy_broker.md)，并将 `hdfs-site.xml` 文件放置在作为 HDFS 集群 NameNode 服务的 broker 节点的 `{deploy}/conf` 路径下。StarRocks将在 broker 启动时将 `{deploy}/conf` 路径添加到环境变量 `CLASSPATH` 中，从而允许 brokers 读取有关 HDFS 集群节点的信息。

  - 如果您从多个 HDFS 集群加载数据（不论配置了一个还是多个 Kerberos 用户），仅支持基于 broker 的加载。请确保为每个这些 HDFS 集群部署了至少一个独立的 [broker 分组](../../../deployment/deploy_broker.md)，并采取以下的行动之一，以使 brokers 能读取有关 HDFS 集群节点的信息：

    - 将 `hdfs-site.xml` 文件放置在作为每个 HDFS 集群 NameNode 服务的 broker 节点的 `{deploy}/conf` 路径下。StarRocks将在 broker 启动时将 `{deploy}/conf` 路径添加到环境变量 `CLASSPATH` 中，从而允许 brokers 读取有关该 HDFS 集群节点的信息。

    - 在 job 创建时添加以下 HA 配置：

      ```Plain
      "dfs.nameservices" = "ha_cluster",
      "dfs.ha.namenodes.ha_cluster" = "ha_n1,ha_n2",
      "dfs.namenode.rpc-address.ha_cluster.ha_n1" = "<hdfs_host>:<hdfs_port>",
      "dfs.namenode.rpc-address.ha_cluster.ha_n2" = "<hdfs_host>:<hdfs_port>",
      "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
      ```

      以下表格描述了 HA 配置中的参数。

      | 参数                         | 描述                   |
      | --------------------------- | ---------------------- |
      | dfs.nameservices            | HDFS 集群的名称。    |
      | dfs.ha.namenodes.XXX        | HDFS 集群中的 NameNode 的名称。如果指定多个 NameNode 名称，请使用逗号（`,`）分隔它们。`XXX` 是您在 `dfs.nameservices` 中指定的 HDFS 集群名称。 |
      | dfs.namenode.rpc-address.XXX.NN | HDFS 集群中 NameNode 的 RPC 地址。`NN` 是您在 `dfs.ha.namenodes.XXX` 中指定的 NameNode 名称。 |
      | dfs.client.failover.proxy.provider | 客户端将连接的 NameNode 的提供者。默认值：`org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。 |

  > **注意**
  >
  > 您可以使用 [SHOW BROKER](../Administration/SHOW_BROKER.md) 语句来检查在您的 StarRocks 集群中部署的 Broker。

#### AWS S3

如果选择 AWS S3 作为您的存储系统，则可以执行以下操作：

"aws.s3.use_instance_profile" = "true",

"aws.s3.region" = "<aws_s3_region>"
```

- 要选择基于假定角色的身份验证方法，请将 `StorageCredentialParams` 配置如下：

```SQL

"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<iam_role_arn>",
"aws.s3.region" = "<aws_s3_region>"
```


- 要选择基于 IAM 用户的身份验证方法，请将 `StorageCredentialParams` 配置如下：

```SQL
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                         | 必需      | 描述                                                         |
| ---------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile  | 是       | 指定是否启用凭证方法实例配置文件和假定角色。有效值：`true` 和 `false`。默认值：`false`。 |
| aws.s3.iam_role_arn          | 否       | 具有对您的 AWS S3 存储桶权限的 IAM 角色的 ARN。如果您选择了假定角色作为访问 AWS S3 的凭证方法，则必须指定此参数。 |
| aws.s3.region                | 是       | 您的 AWS S3 存储桶所在的区域。示例：`us-west-1`。           |
| aws.s3.access_key            | 否       | 您的 IAM 用户的访问密钥。如果您选择 IAM 用户作为访问 AWS S3 的凭证方法，则必须指定此参数。 |
| aws.s3.secret_key            | 否       | 您的 IAM 用户的秘密密钥。如果您选择 IAM 用户作为访问 AWS S3 的凭证方法，则必须指定此参数。 |

有关如何选择访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅 [访问 AWS S3 的身份验证参数](../../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

#### Google GCS

如果您选择 Google GCS 作为您的存储系统，请执行以下操作之一：

- 要选择基于 VM 的身份验证方法，请将 `StorageCredentialParams` 配置如下：

```SQL
"gcp.gcs.use_compute_engine_service_account" = "true"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

| **参数**                                     | **默认值**      | **值示例** | **描述**                                                     |
| -------------------------------------------- | -------------- | ----------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false          | true        | 指定是否直接使用绑定到您的计算引擎的服务帐号。              |

- 要选择基于服务帐号的身份验证方法，请将 `StorageCredentialParams` 配置如下：

```SQL
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

| **参数**                               | **默认值**      | **值示例**                                                    | **描述**                                                     |
| --------------------------------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""             | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建服务帐号时生成的 JSON 文件中的电子邮件地址。           |
| gcp.gcs.service_account_private_key_id | ""             | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                 | 在创建服务帐号时生成的 JSON 文件中的私有密钥 ID。             |
| gcp.gcs.service_account_private_key    | ""             | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建服务帐号时生成的 JSON 文件中的私有密钥。               |

- 要选择基于模拟的身份验证方法，请将 `StorageCredentialParams` 配置如下：

  - 使 VM 实例冒充一个服务帐号：

```SQL
"gcp.gcs.use_compute_engine_service_account" = "true",
"gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

| **参数**                                     | **默认值**      | **值示例** | **描述**                                                     |
| --------------------------------------------- | -------------- | ----------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account    | false          | true        | 指定是否直接使用绑定到您的计算引擎的服务帐号。              |
| gcp.gcs.impersonation_service_account         | ""             | "hello"     | 您要冒充的服务帐号。                                         |

  - 使一个服务帐号（称为元服务帐号）冒充另一个服务帐号（称为数据服务帐号）：

```SQL
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
"gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
"gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

| **参数**                               | **默认值**      | **值示例**                                                    | **描述**                                                     |
| --------------------------------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""             | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建元服务帐号时生成的 JSON 文件中的电子邮件地址。         |
| gcp.gcs.service_account_private_key_id | ""             | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                 | 在创建元服务帐号时生成的 JSON 文件中的私有密钥 ID。           |
| gcp.gcs.service_account_private_key    | ""             | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建元服务帐号时生成的 JSON 文件中的私有密钥。             |
| gcp.gcs.impersonation_service_account  | ""             | "hello"                                                      | 您要冒充的数据服务帐号。                                     |

#### 其他兼容 S3 的存储系统

如果您选择其他兼容 S3 的存储系统，例如 MinIO，请将 `StorageCredentialParams` 配置如下：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                              | 必需      | 描述                                                         |
| --------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                 | 是       | 指定是否启用 SSL 连接。有效值：`true` 和 `false`。默认值：`true`。 |
| aws.s3.enable_path_style_access   | 是       | 指定是否启用路径样式 URL 访问。有效值：`true` 和 `false`。默认值：`false`。对于 MinIO，您必须将该值设置为 `true`。 |
| aws.s3.endpoint                   | 是       | 用于连接到您的兼容 S3 存储系统而不是 AWS S3 的端点。          |
| aws.s3.access_key                 | 是       | 您的 IAM 用户的访问密钥。                                     |
| aws.s3.secret_key                 | 是       | 您的 IAM 用户的秘密密钥。                                     |

#### Microsoft Azure 存储

##### Azure Blob 存储

如果您选择 Blob 存储作为您的存储系统，请执行以下操作之一：

- 要选择共享密钥身份验证方法，请将 `StorageCredentialParams` 配置如下：

```SQL
"azure.blob.storage_account" = "<blob_storage_account_name>",
"azure.blob.shared_key" = "<blob_storage_account_shared_key>"
```

以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数。

| **参数**           | **必需** | **描述**                           |
| ------------------ | -------- | ---------------------------------- |
| azure.blob.storage_account  | 是       | 您的 Blob 存储帐户的用户名。       |
| azure.blob.shared_key | 是       | 您的 Blob 存储帐户的共享密钥。     |

- 要选择 SAS 令牌身份验证方法，请将 `StorageCredentialParams` 配置如下：

```SQL
"azure.blob.account_name" = "<blob_storage_account_name>",
```
"azure.blob.container_name" = "<blob_container_name>",
"azure.blob.sas_token" = "<blob_storage_account_SAS_token>"

以下表格描述了您需要在`StorageCredentialParams`中配置的参数：

| **参数**                    | **必填** | **描述**                                                     |
| --------------------------- | -------- | ------------------------------------------------------------ |
| azure.blob.account_name     | 是       | 您的 Blob 存储帐户的用户名。                                   |
| azure.blob.container_name   | 是       | 存储数据的 Blob 容器的名称。                                  |
| azure.blob.sas_token        | 是       | 用于访问您的 Blob 存储帐户的 SAS 令牌。                      |

##### Azure 数据湖存储Gen1

如果您选择数据湖存储Gen1作为存储系统，请执行以下操作之一：

- 要选择托管服务标识验证方法，请将 `StorageCredentialParams` 配置如下：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数：

  | **参数**                                | **必填** | **描述**                                                     |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是       | 指定是否启用托管服务标识验证方法。将值设置为 `true`。          |

- 要选择服务主体验证方法，请将 `StorageCredentialParams` 配置如下：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数：

  | **参数**                   | **必填** | **描述**                                                     |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id | 是       | .服务主体的客户端（应用程序）ID。                           |
  | azure.adls1.oauth2_credential | 是       | 新创建的客户端（应用程序）密钥的值。                         |
  | azure.adls1.oauth2_endpoint  | 是       | 服务主体或应用程序的 OAuth 2.0 令牌端点（v1）的值。          |

##### Azure 数据湖存储Gen2

如果您选择数据湖存储Gen2作为存储系统，请执行以下操作之一：

- 要选择托管身份验证方法，请将 `StorageCredentialParams` 配置如下：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数：

  | **参数**                               | **必填** | **描述**                                                     |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | 是       | 指定是否启用托管标识验证方法。将值设置为 `true`。            |
  | azure.adls2.oauth2_tenant_id            | 是       | 要访问其数据的租户的 ID。                                   |
  | azure.adls2.oauth2_client_id            | 是       | 托管标识的客户端（应用程序）ID。                             |

- 要选择共享密钥验证方法，请将 `StorageCredentialParams` 配置如下：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数：

  | **参数**               | **必填** | **描述**                                                     |
  | ---------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | 是       | 您的数据湖存储Gen2存储帐户的用户名。                       |
  | azure.adls2.shared_key  | 是       | 您的数据湖存储Gen2存储帐户的共享密钥。                     |

- 要选择服务主体验证方法，请将 `StorageCredentialParams` 配置如下：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下表格描述了您需要在 `StorageCredentialParams` 中配置的参数：

  | **参数**                         | **必填** | **描述**                                                     |
  | -------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id     | 是       | 服务主体的客户端（应用程序）ID。                            |
  | azure.adls2.oauth2_client_secret | 是       | 新创建的客户端（应用程序）密钥的值。                        |
  | azure.adls2.oauth2_client_endpoint | 是      | 服务主体或应用程序的 OAuth 2.0 令牌端点（v1）的值。         |

### opt_properties

指定应用于整个加载作业的一些可选参数的设置。语法：

```Plain
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

支持以下参数：

- `timeout`

  指定加载作业的超时期。单位：秒。默认超时期为4小时。我们建议您指定短于6小时的超时期。如果加载作业在超时期内无法完成，StarRocks将取消加载作业，加载作业的状态变为**CANCELLED**。

  > **注意**
  >
  > 在大多数情况下，您不需要设置超时期。我们建议只有在加载作业无法在默认超时期内完成时，才设置超时期。

  使用以下公式推断超时期：

  **超时期 > （要加载的数据文件的总大小 x 要加载的数据文件的总数和在数据文件上创建的物化视图的总数）/平均加载速度**

  > **注意**
  >
  > “平均加载速度”是整个StarRocks集群的平均加载速度。平均加载速度因每个集群的服务器配置和允许的并发查询任务的最大数量而异。您可以基于历史加载作业的加载速度推断平均加载速度。

  假设您希望将1GB的数据文件及其上创建的两个物化视图加载到平均加载速度为10 MB/s的StarRocks集群中。所需的数据加载时间大约为102秒。

  （1 x 1024 x 3）/10 = 307.2（秒）

  对于此示例，我们建议您将超时期设置为大于308秒的值。

- `max_filter_ratio`

  指定加载作业的最大错误容忍度。最大错误容忍度是可以因数据质量不佳而被过滤掉的行的最大百分比。有效值：`0`~`1`。默认值：`0`。

  - 如果将此参数设置为`0`，则StarRocks在加载时不会忽略不合格行。因此，如果源数据包含不合格行，则加载作业将失败。这有助于确保将正确的数据加载到StarRocks中。

  - 如果将此参数设置为大于`0`的值，则StarRocks在加载时可以忽略不合格行。因此，即使源数据包含不合格行，加载作业也可以成功。

    > **注意**
    >
    > 由于数据质量不佳而被过滤掉的行不包括被WHERE子句过滤掉的行。

  如果加载作业因最大错误容忍度设置为`0`而失败，则您可以使用 [SHOW LOAD](../../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 查看作业结果。然后，确定是否可以过滤掉不合格行。如果可以过滤掉不合格行，则根据作业结果返回的`dpp.abnorm.ALL`和`dpp.norm.ALL`的值计算最大错误容忍度，调整最大错误容忍度，并重新提交加载作业。计算最大错误容忍度的公式如下：

  `max_filter_ratio` = [`dpp.abnorm.ALL`/(`dpp.abnorm.ALL` + `dpp.norm.ALL`)]

  `dpp.abnorm.ALL`和`dpp.norm.ALL`的值之和即为要加载的总行数。

- `log_rejected_record_num`

  指定可记录的未合格数据行的最大数量。此参数从v3.1版本开始受支持。有效值：`0`、`-1`和任何非零正整数。默认值：`0`。

  - 值`0`指定被过滤掉的数据行不会被记录。
  - 值`-1`指定所有被过滤掉的数据行都会被记录。
  - 例如`n`这样的非零正整数指定每个BE最多可以记录`n`个被过滤掉的数据行。

```

指定加载作业可以提供的内存上限。单位：字节 默认内存限制为2GB。

- `strict_mode`

  指定是否启用[严格模式](../../../loading/load_concept/strict_mode.md)。有效值为：`true`和`false`。默认值为：`false`。`true`表示启用严格模式，`false`表示禁用严格模式。

- `timezone`

  指定加载作业的时区。默认值为：`Asia/Shanghai`。时区设置会影响如strftime、alignment_timestamp和from_unixtime等函数返回的结果。有关更多信息，请参见[配置时区](../../../administration/timezone.md)。`timezone`参数中指定的时区是会话级时区。

- `priority`

  指定加载作业的优先级。有效值为：`LOWEST`、`LOW`、`NORMAL`、`HIGH`和`HIGHEST`。默认值为：`NORMAL`。Broker Load提供了[FE参数](../../../administration/Configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`，确定了在StarRocks集群中可以同时运行的Broker Load作业的最大数量。如果在指定时间段内提交的Broker Load作业数量超过最大数量，超出的作业将根据其优先级等待调度。

  您可以使用[ALTER LOAD](../../../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md)语句来更改处于`QUEUEING`或`LOADING`状态的现有加载作业的优先级。

  StarRocks允许自v2.5起为Broker Load作业设置`priority`参数。

## 列映射

如果数据文件的列可以按顺序一对一地映射到StarRocks表的列，则无需配置数据文件与StarRocks表之间的列映射关系。

如果数据文件的列无法按顺序一对一地映射到StarRocks表的列，则需要使用`columns`参数配置数据文件与StarRocks表之间的列映射关系。具体包括以下两种用例：

- **列数相同但列顺序不同。** **此外，加载到匹配的StarRocks表列之前，数据文件中的数据无需通过函数计算。**

  在`columns`参数中，您需要按照数据文件列的排列顺序指定StarRocks表列的名称。

  例如，StarRocks表由三列组成，顺序为`col1`、`col2`和`col3`，数据文件也由三列组成，可以按顺序映射到StarRocks表列`col3`、`col2`和`col1`。在这种情况下，您需要指定`"columns: col3, col2, col1"`。

- **列数不同且列顺序不同。此外，加载到匹配的StarRocks表列之前，数据文件中的数据需要通过函数计算。**

  在`columns`参数中，您需要按照数据文件列的排列顺序指定StarRocks表列的名称，并指定您要用于计算数据的函数。以下是两个示例：

  - StarRocks表由三列组成，顺序为`col1`、`col2`和`col3`。数据文件由四列组成，其中前三列可以按顺序映射到StarRocks表列`col1`、`col2`和`col3`，第四列无法映射到任何StarRocks表列。在这种情况下，您需要临时为数据文件的第四列指定一个名称，且临时名称必须与任何StarRocks表列名称都不同。例如，您可以指定`"columns: col1, col2, col3, temp"`，其中数据文件的第四列暂时命名为`temp`。
  - StarRocks表由三列组成，顺序为`year`、`month`和`day`。数据文件仅包含一列，其中存储了`yyyy-mm-dd hh:mm:ss`格式的日期和时间值。在这种情况下，您可以指定`"columns: col, year = year(col), month=month(col), day=day(col)"`，其中`col`是数据文件列的临时名称，并且函数`year = year(col)`、`month=month(col)`和`day=day(col)`用于提取数据文件列`col`中的数据并将数据加载到相应的StarRocks表列中。例如，`year = year(col)`用于从数据文件列`col`中提取`yyyy`数据并将数据加载到StarRocks表列`year`中。

有关详细示例，请参阅[配置列映射](#configure-column-mapping)。

## 相关配置项

[FE配置项](../../../administration/Configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`指定了在您的StarRocks集群中可以同时运行的Broker Load作业的最大数量。

在StarRocks v2.4及更早版本中，如果在特定时间段内提交的Broker Load作业总数超过了最大数量，多余的作业将被排队并根据其提交时间进行调度。

自StarRocks v2.5以来，如果在特定时间段内提交的Broker Load作业总数超过了最大数量，多余的作业将根据其优先级被排队并调度。您可以使用上述描述的`priority`参数为作业指定优先级。您可以使用[ALTER LOAD](../data-manipulation/ALTER_LOAD.md)来修改处于**QUEUEING**或**LOADING**状态的现有作业的优先级。

## 示例

本节以HDFS为示例，描述了各种加载配置。

### 加载CSV数据

本节以CSV为示例，解释了可以使用的各种参数配置，以满足各种加载要求。

#### 设置超时期限

您的StarRocks数据库`test_db`中包含名为`table1`的表。该表由三列组成，顺序为`col1`、`col2`和`col3`。

您的数据文件`example1.csv`也包含三列，并按顺序映射到`table1`的`col1`、`col2`和`col3`。

如果您希望在最长3600秒内将`example1.csv`中的所有数据加载到`table1`中，运行以下命令：

```SQL
LOAD LABEL test_db.label1
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example1.csv")
    INTO TABLE table1
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

#### 设置最大错误容忍度

您的StarRocks数据库`test_db`中包含名为`table2`的表。该表由三列组成，顺序为`col1`、`col2`和`col3`。

您的数据文件`example2.csv`也包含三列，并按顺序映射到`table2`的`col1`、`col2`和`col3`。

如果您希望将`example2.csv`中的所有数据加载到`table2`中，并设置最大错误容忍度为`0.1`，运行以下命令：

```SQL
LOAD LABEL test_db.label2
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example2.csv")
    INTO TABLE table2

)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
)
PROPERTIES
(
    "max_filter_ratio" = "0.1"
);
```

#### 加载文件路径下的所有数据文件

您的StarRocks数据库`test_db`中包含名为`table3`的表。该表由三列组成，顺序为`col1`、`col2`和`col3`。

您的HDFS集群的`/user/starrocks/data/input/`路径下存储的所有数据文件也都包含三列，并按顺序映射到`table3`的`col1`、`col2`和`col3`。这些数据文件中使用的列分隔符是`\x01`。

如果您希望将存储在`hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/`路径下的所有这些数据文件中的数据加载到`table3`中，运行以下命令：

```SQL
LOAD LABEL test_db.label3
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/*")
    INTO TABLE table3
    COLUMNS TERMINATED BY "\\x01"
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

#### 设置NameNode HA机制

您的StarRocks数据库`test_db`包含一个名为`table4`的表。表包含三列，分别是顺序为`col1`，`col2`和`col3`。

您的数据文件`example4.csv`也包含三列，这三列映射到`table4`的`col1`，`col2`和`col3`上。

如果您要用配置了NameNode的HA机制将`example4.csv`中的全部数据加载到`table4`中，请运行以下命令：

```SQL
LOAD LABEL test_db.label4
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example4.csv")
    INTO TABLE table4
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>",
    "dfs.nameservices" = "my_ha",
    "dfs.ha.namenodes.my_ha" = "my_namenode1, my_namenode2","dfs.namenode.rpc-address.my_ha.my_namenode1" = "nn1_host:rpc_port",
    "dfs.namenode.rpc-address.my_ha.my_namenode2" = "nn2_host:rpc_port",
    "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
);
```

#### 设置Kerberos认证

您的StarRocks数据库`test_db`包含一个名为`table5`的表。表包含三列，分别是顺序为`col1`，`col2`和`col3`。

您的数据文件`example5.csv`也包含三列，这三列按顺序映射到`table5`的`col1`，`col2`和`col3`上。

如果您要用配置了Kerberos认证并指定了keytab文件路径的方式，将`example5.csv`中的全部数据加载到`table5`中，请运行以下命令：

```SQL
LOAD LABEL test_db.label5
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example5.csv")
    INTO TABLE table5
    COLUMNS TERMINATED BY "\t"
)
WITH BROKER
(
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "starrocks@YOUR.COM",
    "kerberos_keytab" = "/home/starRocks/starRocks.keytab"
);
```

#### 撤销数据加载

您的StarRocks数据库`test_db`包含一个名为`table6`的表。表包含三列，分别是顺序为`col1`，`col2`和`col3`。

您的数据文件`example6.csv`也包含三列，这三列按顺序映射到`table6`的`col1`，`col2`和`col3`上。

您已经通过运行Broker Load作业将`example6.csv`中的全部数据加载到`table6`中。

如果您要撤销已加载的数据，请运行以下命令：

```SQL
LOAD LABEL test_db.label6
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example6.csv")
    NEGATIVE
    INTO TABLE table6
    COLUMNS TERMINATED BY "\t"
)
WITH BROKER
(
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "starrocks@YOUR.COM",
    "kerberos_keytab" = "/home/starRocks/starRocks.keytab"
);
```

#### 指定目标分区

您的StarRocks数据库`test_db`包含一个名为`table7`的表。表包含三列，分别是顺序为`col1`，`col2`和`col3`。

您的数据文件`example7.csv`也包含三列，这三列按顺序映射到`table7`的`col1`，`col2`和`col3`上。

如果您要将`example7.csv`中的全部数据加载到`table7`的`p1`和`p2`两个分区中，请运行以下命令：

```SQL
LOAD LABEL test_db.label7
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example7.csv")
    INTO TABLE table7
    PARTITION (p1, p2)
    COLUMNS TERMINATED BY ","
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

#### 配置列映射

您的StarRocks数据库`test_db`包含一个名为`table8`的表。表包含三列，分别是顺序为`col1`，`col2`和`col3`。

您的数据文件`example8.csv`也包含三列，这三列按顺序映射到`table8`的`col2`，`col1`和`col3`上。

如果您要将`example8.csv`中的全部数据加载到`table8`中，请运行以下命令：

```SQL
LOAD LABEL test_db.label8
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example8.csv")
    INTO TABLE table8
    COLUMNS TERMINATED BY ","
    (col2, col1, col3)
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> 在前面的示例中，`example8.csv`的列与`table8`中的列不能以相同顺序进行映射。因此，您需要使用`column_list`来配置`example8.csv`和`table8`之间的列映射。

#### 设置过滤条件

您的StarRocks数据库`test_db`包含一个名为`table9`的表。表包含三列，分别是顺序为`col1`，`col2`和`col3`。

您的数据文件`example9.csv`也包含三列，这三列按顺序映射到`table9`的`col1`，`col2`和`col3`上。

如果您只想从`example9.csv`中加载第一列值大于`20180601`的数据记录到`table9`中，请运行以下命令：

```SQL
LOAD LABEL test_db.label9
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example9.csv")
    INTO TABLE table9
    (col1, col2, col3)
    where col1 > 20180601
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> 在前面的示例中，`example9.csv`的列可以按照它们在`table9`中的顺序进行映射。但是，您需要使用`WHERE`子句来指定基于列的过滤条件。因此，您需要使用`column_list`来配置`example9.csv`和`table9`之间的列映射。

#### 将数据加载到包含HLL类型列的表中

您的StarRocks数据库`test_db`包含一个名为`table10`的表。表包含四列，分别为顺序为`id`，`col1`，`col2`和`col3`。`col1`和`col2`被定义为HLL类型列。

您的数据文件`example10.csv`包含三列，其中第一列映射到`table10`的`id`，第二列和第三列按顺序映射到`table10`的`col1`和`col2`上。`example10.csv`中第二列和第三列的值可以在加载到`table10`的`col1`和`col2`之前通过函数转换为HLL类型数据。

如果您要将`example10.csv`中的全部数据加载到`table10`中，请运行以下命令：

```SQL
LOAD LABEL test_db.label10
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example10.csv")
    INTO TABLE table10
    COLUMNS TERMINATED BY ","
    (id, temp1, temp2)
    SET
    (
        col1 = hll_hash(temp1),
        col2 = hll_hash(temp2),
        col3 = empty_hll()
     )
 )
 WITH BROKER
 (
     "username" = "<hdfs_username>",
     "password" = "<hdfs_password>"
 );
```

> **注意**
>
> 在前面的示例中，`example10.csv`的三列使用`column_list`命名为`id`，`temp1`和`temp2`。然后，函数用于将数据转换如下：
>
> - 使用`hll_hash`函数将`example10.csv`中的`temp1`和`temp2`的值转换为HLL类型数据，并将`example10.csv`的`temp1`和`temp2`映射到`table10`的`col1`和`col2`上。
- 函数 `hll_empty` 用于将指定的默认值填充到 `table10` 的 `col3` 中。

有关函数 `hll_hash` 和 `hll_empty` 的用法，请参见 [hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) 和 [hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)。

#### 从文件路径提取分区字段值

Broker Load 支持基于目的地 StarRocks 表的列定义解析文件路径中包含的特定分区字段的值。这个 StarRocks 的功能类似于 Apache Spark™ 的分区发现功能。

你的 StarRocks 数据库 `test_db` 包含名为 `table11` 的表。该表包括五列，依次为 `col1`、`col2`、`col3`、`city` 和 `utc_date`。

你的 HDFS 集群上的文件路径 `/user/starrocks/data/input/dir/city=beijing` 包含以下数据文件：

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv`

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv`

这些数据文件分别包括三列，依次映射到 `table11` 的 `col1`、`col2` 和 `col3`。

如果你想从文件路径 `/user/starrocks/data/input/dir/city=beijing/utc_date=*/*` 中的所有数据文件加载数据到 `table11` 中，并且同时你想要提取文件路径中包含的分区字段 `city` 和 `utc_date` 的值，并将提取的值加载到 `table11` 的 `city` 和 `utc_date` 中，请运行以下命令：

```SQL
LOAD LABEL test_db.label11
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/dir/city=beijing/*/*")
    INTO TABLE table11
    FORMAT AS "csv"
    (col1, col2, col3)
    COLUMNS FROM PATH AS (city, utc_date)
    SET (uniq_id = md5sum(k1, city))
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

#### 从包含 `%3A` 的文件路径提取分区字段值

在 HDFS 中，文件路径不能包含冒号（:）。所有冒号（:）将被转换为 `%3A`。

你的 StarRocks 数据库 `test_db` 包含名为 `table12` 的表。该表包括三列，依次为 `data_time`、`col1` 和 `col2`。表模式如下：

```SQL
data_time DATETIME,
col1        INT,
col2        INT
```

你的 HDFS 集群上的文件路径 `/user/starrocks/data` 包含以下数据文件：

- `/user/starrocks/data/data_time=2020-02-17 00%3A00%3A00/example12.csv`

- `/user/starrocks/data/data_time=2020-02-18 00%3A00%3A00/example12.csv`

如果你想要将 `example12.csv` 中的所有数据加载到 `table12` 中，并且同时你想要从文件路径中提取分区字段 `data_time` 的值，并将提取的值加载到 `table12` 的 `data_time` 中，请运行以下命令：

```SQL
LOAD LABEL test_db.label12
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/*/example12.csv")
    INTO TABLE table12
    COLUMNS TERMINATED BY ","
    FORMAT AS "csv"
    (col1,col2)
    COLUMNS FROM PATH AS (data_time)
    SET (data_time = str_to_date(data_time, '%Y-%m-%d %H%%3A%i%%3A%s'))
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> 在前述示例中，从分区字段 `data_time` 中提取的值是包含 `%3A` 的字符串，例如 `2020-02-17 00%3A00%3A00`。因此，在加载到 `table8` 的 `data_time` 之前，你需要使用 `str_to_date` 函数将字符串转换为 DATETIME 类型的数据。

#### 设置 `format_type_options`

你的 StarRocks 数据库 `test_db` 包含名为 `table13` 的表。该表包括三列，依次为 `col1`、`col2` 和 `col3`。

你的数据文件 `example13.csv` 也包括三列，依次映射到 `table13` 的 `col2`、`col1` 和 `col3`。

如果你想要将 `example13.csv` 中的所有数据加载到 `table13` 中，并且想要跳过 `example13.csv` 中的前两行，删除列分隔符前后的空格，并将 `enclose` 设置为 `\`，`escape` 设置为 `\`，请运行以下命令：

```SQL
LOAD LABEL test_db.label13
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/*/example13.csv")
    INTO TABLE table13
    COLUMNS TERMINATED BY ","
    FORMAT AS "CSV"
    (
        skip_header = 2
        trim_space = TRUE
        enclose = "\""
        escape = "\\"
    )
    (col2, col1, col3)
)
WITH BROKER
(
    "username" = "hdfs_username",
    "password" = "hdfs_password"
)
PROPERTIES
(
    "timeout" = "3600"
);
```

### 加载 Parquet 数据

本节描述了加载 Parquet 数据时需要注意的一些参数设置。

你的 StarRocks 数据库 `test_db` 包含名为 `table13` 的表。该表包括三列，依次为 `col1`、`col2` 和 `col3`。

你的数据文件 `example13.parquet` 也包括三列，依次映射到 `table13` 的 `col1`、`col2` 和 `col3`。

如果你想要将 `example13.parquet` 中的所有数据加载到 `table13` 中，请运行以下命令：

```SQL
LOAD LABEL test_db.label13
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example13.parquet")
    INTO TABLE table13
    FORMAT AS "parquet"
    (col1, col2, col3)
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> 默认情况下，当加载 Parquet 数据时，StarRocks 会根据文件名是否包含扩展名 **.parquet** 来确定数据文件格式。如果文件名不包含扩展名 **.parquet**，则必须使用 `FORMAT AS` 来指定数据文件格式为 `Parquet`。

### 加载 ORC 数据

本节描述了加载 ORC 数据时需要注意的一些参数设置。

你的 StarRocks 数据库 `test_db` 包含名为 `table14` 的表。该表包括三列，依次为 `col1`、`col2` 和 `col3`。

你的数据文件 `example14.orc` 也包括三列，依次映射到 `table14` 的 `col1`、`col2` 和 `col3`。

如果你想要将 `example14.orc` 中的所有数据加载到 `table14` 中，请运行以下命令：

```SQL
LOAD LABEL test_db.label14
(
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/example14.orc")
    INTO TABLE table14
    FORMAT AS "orc"
    (col1, col2, col3)
)
WITH BROKER
(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

> **注意**
>
> - 默认情况下，当加载 ORC 数据时，StarRocks 会根据文件名是否包含扩展名 **.orc** 来确定数据文件格式。如果文件名不包含扩展名 **.orc**，则必须使用 `FORMAT AS` 来指定数据文件格式为 `ORC`。
>
> - 在 StarRocks v2.3 及更早版本中，如果数据文件包含 ARRAY 类型列，则必须确保 ORC 数据文件的列名称与 StarRocks 表中的映射列名称相同，且这些列不能在 SET 子句中指定。