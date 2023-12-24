---
displayed_sidebar: English
---

# 代理加载

从 '../../../assets/commonMarkdown/insertPrivNote.md' 导入InsertPrivNote

## 描述

StarRocks提供了基于MySQL的加载方法Broker Load。在提交加载作业后，StarRocks会异步运行该作业。您可以使用`SELECT * FROM information_schema.loads`来查询作业结果。此功能从v3.1版本开始受支持。有关背景信息、原理、支持的数据文件格式、如何执行单表加载和多表加载、以及如何查看作业结果的更多信息，请参见[HDFS加载数据](../../../loading/hdfs_load.md)和[云存储加载数据](../../../loading/cloud_storage_load.md)。

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

请注意，在StarRocks中，一些文字被SQL语言用作保留关键字。不要在SQL语句中直接使用这些关键字。如果要在SQL语句中使用此类关键字，请将其括在一对反引号（`）中。请参阅[关键字](../../../sql-reference/sql-statements/keywords.md)。

## 参数

### database_name和label_name

`label_name`指定加载作业的标签。

`database_name`（可选）指定目标表所属的数据库的名称。

每个加载作业都有一个标签，该标签在整个数据库中是唯一的。您可以使用加载作业的标签来查看加载作业的执行状态，防止重复加载相同的数据。当加载作业进入**FINISHED**状态时，其标签无法重复使用。只有已进入**CANCELLED**状态的加载作业的标签才能被重用。在大多数情况下，重用加载作业的标签来重试该加载作业并加载相同的数据，从而实现Exactly-Once语义。

有关标签命名约定，请参见[系统限制](../../../reference/System_limit.md)。

### data_desc

需要加载的一批数据的描述。每个`data_desc`描述符都声明了数据源、ETL函数、目标StarRocks表和目标分区等信息。

Broker Load支持一次加载多个数据文件。在一个加载作业中，可以使用多个描述符来声明要加载的多个数据文件`data_desc`，也可以使用一个描述符`data_desc`来声明一个文件路径，从中加载其中的所有数据文件。Broker Load还可以确保为加载多个数据文件而运行的每个加载作业的事务原子性。原子性意味着在一个加载作业中加载多个数据文件必须全部成功或失败。某些数据文件的加载成功而其他文件的加载失败的情况永远不会发生。

`data_desc`支持以下语法：

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

`data_desc`必须包含以下参数：

- `file_path`

  指定要加载的一个或多个数据文件的保存路径。

  您可以将该参数指定为一个数据文件的保存路径。例如，您可以指定该参数，以加载`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/20210411"`从`20210411` HDFS服务器上`/user/data/tablename`的路径命名的数据文件。

  也可以使用通配符 、 、 `?`、 `*`或 `[]` 指定此参数作为多个数据文件的保存路径 `{}` `^`。请参阅[通配符引用](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/fs/FileSystem.html#globStatus-org.apache.hadoop.fs.Path-)。例如，可以将该参数指定为`"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/*/*"` `"hdfs://<hdfs_host>:<hdfs_port>/user/data/tablename/dt=202104*/*"`或从HDFS服务器`202104`路径中的所有`/user/data/tablename`分区或仅加载数据文件。

  > **注意**
  >
  > 通配符还可用于指定中间路径。

  在前面的示例中，`hdfs_host`和`hdfs_port`参数说明如下：

  - `hdfs_host`：HDFS集群中NameNode主机的IP地址。

  - `hdfs_host`：HDFS集群中NameNode主机的FS端口。缺省端口号为`9000`。

  > **通知**
  >
  > - Broker Load支持根据S3或S3A协议访问AWS S3。因此，当您从AWS S3加载数据时，您可以在`s3://` `s3a://`作为文件路径传递的S3 URI中包含或作为前缀。
  > - Broker Load仅支持根据gs协议访问Google GCS。因此，当您从Google GCS加载数据时，您必须将作为`gs://`前缀包含在作为文件路径传递的GCS URI中。
  > - 从Blob存储加载数据时，必须使用wasb或wasbs协议来访问数据：
  >   - 如果存储帐户允许通过HTTP进行访问，请使用wasb协议并将文件路径写入`wasb://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`.
  >   - 如果存储帐户允许通过HTTPS进行访问，请使用wasbs协议并将文件路径写入`wasbs://<container>@<storage_account>.blob.core.windows.net/<path>/<file_name>/*`
  > - 从Data Lake Storage Gen1加载数据时，必须使用adl协议访问数据，并将文件路径写入`adl://<data_lake_storage_gen1_name>.azuredatalakestore.net/<path>/<file_name>`.
  > - 从Data Lake Storage Gen2加载数据时，必须使用abfs或abfss协议来访问数据：
  >   - 如果存储帐户允许通过HTTP进行访问，请使用abfs协议并将文件路径写入`abfs://<container>@<storage_account>.dfs.core.windows.net/<file_name>`.
  >   - 如果存储帐户允许通过HTTPS进行访问，请使用abfss协议并将文件路径写入`abfss://<container>@<storage_account>.dfs.core.windows.net/<file_name>`。

- `INTO TABLE`

  指定目标StarRocks表的名称。

`data_desc`还可以选择包含以下参数：

- `NEGATIVE`

  撤消特定批次数据的加载。为此，您需要使用指定的关键字加载同一批数据`NEGATIVE`。

  > **注意**
  >
  > 仅当StarRocks表为Aggregate表且其所有值列均由该函数计算时，该参数才有效`sum`。

- `PARTITION`

   指定要将数据加载到的分区。默认情况下，如果不指定该参数，源数据将被加载到StarRocks表的所有分区中。

- `TEMPORARY PARTITION`

  指定[临时分区](../../../table_design/Temporary_partition.md)要将数据加载到其中的名称。您可以指定多个临时分区，这些分区必须用逗号（，）分隔。

- `COLUMNS TERMINATED BY`

  指定数据文件中使用的列分隔符。默认情况下，如果不指定此参数，则此参数默认为`\t`，表示tab。使用此参数指定的列分隔符必须与数据文件中实际使用的列分隔符相同。否则，加载作业将因数据质量不足而失败，其`State` `CANCELLED`。

  Broker Load作业根据MySQL协议提交。StarRocks和MySQL都在加载请求中转义字符。因此，如果列分隔符是不可见字符（如制表符），则必须在列分隔符之前添加反斜杠（\）。例如，如果`\\t`列分隔符为，则必须输入，`\t`如果列分隔符为，`\\n`则必须输入`\n`。Apache Hive文件用作其列分隔符，因此必须输入数据文件是否来自Hive™`\x01` `\\x01`。

  > **注意**
  >
  > - 对于CSV数据，可以使用长度不超过50个字节的UTF-8字符串（例如逗号（，）、制表符或竖线（|））作为文本分隔符。
  > - 空值用表示`\N`。例如，数据文件由三列组成，该数据文件中的记录在第一列和第三列中保存数据，但在第二列中不保存任何数据。在这种情况下，您需要`\N`在第二列中使用null值来表示null值。这意味着必须将记录编译为`a,\N,b`而不是`a,,b`. `a,,b`表示记录的第二列包含一个空字符串。

- `ROWS TERMINATED BY`

  指定数据文件中使用的行分隔符。默认情况下，如果不指定此参数，则此参数默认为`\n`，表示换行符。使用此参数指定的行分隔符必须与数据文件中实际使用的行分隔符相同。否则，加载作业将因数据质量不足而失败，其`State` `CANCELLED`。从v2.5.4开始支持此参数。

  有关行分隔符的使用说明，请参阅上述参数的使用说明`COLUMNS TERMINATED BY`。

- `FORMAT AS`

  指定数据文件的格式。有效值：`CSV`、`Parquet`和`ORC`。默认情况下，如果不指定该参数，StarRocks会根据参数**中指定的**文件扩展名.csv**、.parquet**或**.orc来确定数据文件格式`file_path`。

- `format_type_options`

  指定CSV格式选项（`FORMAT AS`设置为`CSV`时）。语法：

  ```JSON
  (
      key = value
      key = value
      ...
  )
  ```

  > **注意**
  >
  > `format_type_options`在v3.0及更高版本中受支持。

  下表介绍了这些选项。

  | **参数** | **描述**                                              |
  | ------------- | ------------------------------------------------------------ |
  | skip_header   | 指定当数据文件为CSV格式时是否跳过数据文件的第一行。类型：INTEGER。默认值：`0`。<br />在某些CSV格式的数据文件中，开头的第一行用于定义元数据，例如列名和列数据类型。通过设置该`skip_header`参数，可以让StarRocks在数据加载时跳过数据文件的第一行。例如，如果将该参数设置为，`1`StarRocks在加载数据时会跳过数据文件的第一行。<br />必须使用在load语句中指定的行分隔符分隔数据文件开头的第一行。 |

  | trim_space    | 指定是否在数据文件为 CSV 格式时删除列分隔符前后的空格。类型：BOOLEAN。默认值：`false`。<br />对于某些数据库，当您将数据导出为 CSV 格式的数据文件时，会在列分隔符中添加空格。这些空格根据其位置分别称为前导空格或尾随空格。通过设置 `trim_space` 参数，您可以让 StarRocks 在数据加载过程中去除这些不必要的空格。<br />需要注意的是，StarRocks 不会移除字段中用 `enclose` 指定字符包裹的空格（包括前导空格和尾随空格）。例如，以下字段值使用竖线（<code class="language-text">&#124;</code>）作为列分隔符，双引号（`"`）作为 `enclose` 指定字符：<br /><code class="language-text">&#124;"爱上星石"&#124;</code> <br /><code class="language-text">&#124;" 爱上StarRocks "&#124;</code> <br /><code class="language-text">&#124; "爱上星石" &#124;</code> <br />如果将 `trim_space` 设置为 `true`，StarRocks 将按如下方式处理上述字段值：<br /><code class="language-text">&#124;"爱上星石"&#124;</code> <br /><code class="language-text">&#124;" 爱上StarRocks "&#124;</code> <br /><code class="language-text">&#124;"爱上星石"&#124;</code> |
  | enclose       | 指定数据文件为 CSV 格式时，用于根据 [RFC4180](https://www.rfc-editor.org/rfc/rfc4180) 将字段值包裹起来的字符。类型：单字节字符。默认值：`NONE`。最常用的字符是单引号（`'`）和双引号（`"`）。<br />使用 `enclose` 指定字符包裹的所有特殊字符（包括行分隔符和列分隔符）都被视为普通符号。StarRocks 不仅支持 RFC4180，还允许您指定任何单字节字符作为 `enclose` 指定字符。<br />如果字段值包含 `enclose` 指定字符，您可以使用相同的字符对该 `enclose` 指定字符进行转义。例如，将 `enclose` 设置为 `"`，字段值为 `a "quoted" c`。在这种情况下，您可以在数据文件中输入字段值 `"a ""quoted"" c"`。 |
  | escape        | 指定用于转义各种特殊字符的字符，例如行分隔符、列分隔符、转义字符和 `enclose` 指定字符，StarRocks 将其视为通用字符，并解析为它们所在的字段值的一部分。类型：单字节字符。默认值：`NONE`。最常用的字符是斜杠（\），在 SQL 语句中必须写成双斜杠（\\）。<br />**注意**<br />：`escape` 指定的字符适用于每对 `enclose` 指定字符的内部和外部。以下是两个例子：<ul><li>当您将 `enclose` 设置为 `"`，`escape` 设置为 `\` 时，StarRocks 将 `"say \"Hello world\""` 解析为 `say "Hello world"`。</li><li>假设列分隔符为逗号（`,`）。当您将 `escape` 设置为 `\` 时，StarRocks 将 `a, b\, c` 解析为两个独立的字段值：`a` 和 `b, c`。</li></ul> |

- `column_list`

  指定数据文件与 StarRocks 表之间的列映射关系。语法：`(<column_name>[, <column_name> ...])`。在 `column_list` 中声明的列按名称映射到 StarRocks 表的列上。

  > **注意**
  >
  > 如果数据文件的列按顺序映射到 StarRocks 表的列上，则无需指定 `column_list`。

  如果要跳过数据文件中的特定列，只需将该列临时命名为与 StarRocks 表的任何列不同的名称。有关详细信息，请参阅 [加载时转换数据](../../../loading/Etl_in_loading.md)。

- `COLUMNS FROM PATH AS`

  从指定的文件路径中提取有关一个或多个分区字段的信息。仅当文件路径包含分区字段时，此参数才有效。

  例如，如果数据文件存储在路径 `/path/col_name=col_value/file1` 中，其中 `col_name` 是一个分区字段，并且可以映射到 StarRocks 表的某一列，则可以将此参数指定为 `col_name`。因此，StarRocks 会从路径中提取 `col_value` 值，并将其加载到映射到 StarRocks 表中的 `col_name` 列上。

  > **注意**
  >
  > 仅当从 HDFS 加载数据时，此参数才可用。

- `SET`

  指定要用于转换数据文件列的一个或多个函数。例如：

  - StarRocks 表由三列组成，分别为 `col1`、`col2` 和 `col3`。数据文件由四列组成，其中前两列按顺序映射到 StarRocks 表的 `col1` 和 `col2`，后两列的和映射到 StarRocks 表的 `col3`。在这种情况下，需要在 SET 子句中指定 `column_list` 为 `(col1,col2,tmp_col3,tmp_col4)`，并指定 `(col3=tmp_col3+tmp_col4)` 来实现数据转换。
  - StarRocks 表由三列组成，分别为 `year`、`month` 和 `day`。数据文件仅包含一列，其中包含 `yyyy-mm-dd hh:mm:ss` 格式的日期和时间值。在这种情况下，需要在 SET 子句中指定 `column_list` 为 `(tmp_time)`，并指定 `(year = year(tmp_time), month=month(tmp_time), day=day(tmp_time))` 来实现数据转换。

- `WHERE`

  指定要根据其筛选源数据的条件。StarRocks 仅加载满足 WHERE 子句中指定的筛选条件的源数据。

### 使用经纪人

在 v2.3 及更早版本中，输入 `WITH BROKER "<broker_name>"` 以指定要使用的经纪人。从 v2.5 开始，您不再需要指定经纪人，但仍需保留 `WITH BROKER` 关键字。

> **注意**
>
> 在 v2.4 及更早版本中，StarRocks 在运行 Broker Load 作业时，需要经纪人来建立 StarRocks 集群与外部存储系统之间的连接。这称为“基于经纪人的加载”。经纪人是与文件系统接口集成的独立无状态服务。通过经纪人，StarRocks 可以访问和读取存储在外部存储系统中的数据文件，并可以使用自己的计算资源对这些数据文件的数据进行预处理和加载。
>
> 从 v2.5 开始，StarRocks 不再依赖经纪人，实现了“无经纪人加载”。
>
> 您可以使用 [SHOW BROKER](../Administration/SHOW_BROKER.md) 语句来查看您的 StarRocks 集群中部署了哪些经纪人。如果未部署经纪人，您可以按照 [部署经纪人](../../../deployment/deploy_broker.md) 中提供的说明来部署经纪人。

### StorageCredentialParams

StarRocks 用于访问您的存储系统的身份验证信息。

#### HDFS

开源 HDFS 支持两种身份验证方法：简单身份验证和 Kerberos 身份验证。Broker Load 默认使用简单身份验证。开源 HDFS 还支持为 NameNode 配置 HA 机制。如果选择开源 HDFS 作为您的存储系统，您可以按如下方式指定身份验证配置和 HA 配置：

- 身份验证配置

  - 如果使用简单身份验证，请按如下方式配置 `StorageCredentialParams`：

    ```Plain
    "hadoop.security.authentication" = "simple",
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
    ```

    下表描述了 `StorageCredentialParams` 中的参数。

    | 参数                       | 描述                                                  |
    | ------------------------------- | ------------------------------------------------------------ |
    | hadoop.security.authentication（hadoop.security.authentication）  | 身份验证方法。有效值： `simple` 和 `kerberos`。默认值：`simple`。 `simple` 表示简单身份验证，表示无身份验证，并表示 `kerberos` Kerberos 身份验证。 |
    | 用户名                        | 用于访问 HDFS 集群 NameNode 的帐户的用户名。 |
    | 密码                        | 用于访问 HDFS 集群 NameNode 的帐户的密码。 |

  - 如果使用 Kerberos 身份验证，请按如下方式配置 `StorageCredentialParams`：

    ```Plain
    "hadoop.security.authentication" = "kerberos",
    "kerberos_principal" = "nn/zelda1@ZELDA.COM",
    "kerberos_keytab" = "/keytab/hive.keytab",
    "kerberos_keytab_content" = "YWFhYWFh"
    ```

    下表描述了 `StorageCredentialParams` 中的参数。

    | 参数                       | 描述                                                  |
    | ------------------------------- | ------------------------------------------------------------ |
    | hadoop.security.authentication（hadoop.security.authentication）  | 身份验证方法。有效值： `simple` 和 `kerberos`。默认值：`simple`。 `simple` 表示简单身份验证，表示无身份验证，并表示 `kerberos` Kerberos 身份验证。 |
    | kerberos_principal              | 要进行身份验证的 Kerberos 主体。每个主体由以下三个部分组成，以确保它在 HDFS 集群中是唯一的： <ul><li>`username` 或 `servicename`：主体的名称。</li><li>`instance`：HDFS 集群中待认证节点的服务器名称。服务器名称有助于确保主体是唯一的，例如，当 HDFS 集群由多个 DataNode 组成时，每个 DataNode 都经过独立身份验证。</li><li>`realm`：领域名称。领域名称必须大写。示例：`nn/[zelda1@ZELDA.COM](mailto:zelda1@ZELDA.COM)`。</li></ul> |
    | kerberos_keytab                 | Kerberos 密钥表文件的保存路径。 |
    | kerberos_keytab_content         | Kerberos 密钥表文件的 Base64 编码内容。您可以选择指定 `kerberos_keytab` 或 `kerberos_keytab_content`。 |

    如果配置了多个 Kerberos 用户，请确保至少部署了一个独立的 [经纪人组](../../../deployment/deploy_broker.md)，并且在 load 语句中输入 `WITH BROKER "<broker_name>"` 以指定要使用的经纪人组。此外，您必须打开经纪人启动脚本文件 **start_broker.sh**，并修改该文件的第 42 行，以使经纪人能够读取 **krb5.conf** 文件。例如：

    ```Plain
    export JAVA_OPTS="-Dlog4j2.formatMsgNoLookups=true -Xmx1024m -Dfile.encoding=UTF-8 -Djava.security.krb5.conf=/etc/krb5.conf"
    ```

    > **注意**
    >

    > - 在前面的示例中，`/etc/krb5.conf`可以替换为实际的**krb5.conf**文件保存路径。确保代理对该文件具有读取权限。如果 Broker Group 包含多个 Broker，则必须修改每个 Broker 节点上的**start_broker.sh**文件，然后重新启动 Broker 节点以使修改生效。
    > - 您可以使用[SHOW BROKER](../Administration/SHOW_BROKER.md)语句来查看 StarRocks 集群中部署的 Broker。

- HA 配置

  您可以为 HDFS 集群的 NameNode 配置 HA 机制。这样一来，当 NameNode 切换到另一个节点时，StarRocks 就可以自动识别新的 NameNode 节点。具体包括以下方案：

  - 如果从配置了单个 Kerberos 用户的单个 HDFS 集群加载数据，则支持基于负载的加载和无负载加载。
  
    - 要执行基于负载的加载，请确保至少部署一个独立的[代理组](../../../deployment/deploy_broker.md)，并将`hdfs-site.xml`文件放置到为 HDFS 集群提供服务的代理节点的`{deploy}/conf`路径下。StarRocks 在 Broker 启动时会将`{deploy}/conf`路径添加到环境变量`CLASSPATH`中，以便 Broker 读取 HDFS 集群节点的信息。
  
    - 要执行无负载加载，请将`hdfs-site.xml`文件放置到每个 FE 节点和每个 BE 节点的`{deploy}/conf`路径下。

  - 如果从配置了多个 Kerberos 用户的单个 HDFS 集群加载数据，则仅支持基于代理的加载。确保至少部署一个独立的[代理组](../../../deployment/deploy_broker.md)，并将`hdfs-site.xml`文件放置到为 HDFS 集群提供服务的代理节点的`{deploy}/conf`路径下。StarRocks 在 Broker 启动时会将`{deploy}/conf`路径添加到环境变量`CLASSPATH`中，以便 Broker 读取 HDFS 集群节点的信息。

  - 如果从多个 HDFS 集群加载数据（无论配置了一个或多个 Kerberos 用户），则仅支持基于代理的加载。确保为每个 HDFS 集群至少部署一个独立的[代理组](../../../deployment/deploy_broker.md)，并执行以下操作之一以使代理能够读取有关 HDFS 集群节点的信息：

    - 将`hdfs-site.xml`文件放置到为每个 HDFS 集群提供服务的代理节点的`{deploy}/conf`路径下。StarRocks 在 Broker 启动时会将`{deploy}/conf`路径添加到环境变量`CLASSPATH`中，允许 Broker 读取该 HDFS 集群中节点的信息。

    - 在创建作业时添加以下 HA 配置：

      ```Plain
      "dfs.nameservices" = "ha_cluster",
      "dfs.ha.namenodes.ha_cluster" = "ha_n1,ha_n2",
      "dfs.namenode.rpc-address.ha_cluster.ha_n1" = "<hdfs_host>:<hdfs_port>",
      "dfs.namenode.rpc-address.ha_cluster.ha_n2" = "<hdfs_host>:<hdfs_port>",
      "dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
      ```

      HA 配置中的参数说明如下表所示。

      | 参数                          | 描述                                                  |
      | ---------------------------------- | ------------------------------------------------------------ |
      | dfs.nameservices                   | HDFS 集群的名称。                                |
      | dfs.ha.namenodes.XXX               | HDFS 集群中 NameNode 的名称。如果指定多个 NameNode 名称，请用逗号（,）分隔。`XXX` 是您在 `dfs.nameservices` 中指定的 HDFS 集群名称。 |
      | dfs.namenode.rpc-address.XXX.NN    | NameNode 在 HDFS 集群中的 RPC 地址。`NN` 是您在 `dfs.ha.namenodes.XXX` 中指定的 NameNode 名称。 |
      | dfs.client.failover.proxy.provider | 客户端将连接到的 NameNode 的提供程序。默认值： `org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider`。 |

  > **注意**
  >
  > 您可以使用[SHOW BROKER](../Administration/SHOW_BROKER.md)语句来查看 StarRocks 集群中部署的 Broker。

#### AWS S3

如果选择 AWS S3 作为存储系统，请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择假定的基于角色的身份验证方法，请按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择 IAM 基于用户的身份验证方法，请按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                   | 必填 | 描述                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是      | 指定是否启用实例配置文件和假定角色的凭据方法。有效值： `true` 和 `false`。默认值： `false`。 |
| aws.s3.iam_role_arn         | 否       | 拥有权限访问 AWS S3 存储桶的 IAM 角色的 ARN。如果选择假定角色作为访问 AWS S3 的凭据方法，则必须指定此参数。 |
| aws.s3.region               | 是      | 您的 AWS S3 存储桶所在的区域。示例： `us-west-1`。 |
| aws.s3.access_key           | 否       | IAM 用户的访问密钥。如果选择 IAM 用户作为访问 AWS S3 的凭据方法，则必须指定此参数。 |
| aws.s3.secret_key           | 否       | IAM 用户的私有密钥。如果选择 IAM 用户作为访问 AWS S3 的凭据方法，则必须指定此参数。 |

有关如何选择用于访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅[用于访问 AWS S3 的身份验证参数](../../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

#### Google GCS

如果选择 Google GCS 作为存储系统，请执行以下操作之一：

- 若要选择基于 VM 的身份验证方法，请按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                              | **默认值** | **示例值** | **描述**                                              |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用与 Compute Engine 绑定的服务帐号。 |

- 若要选择基于服务帐户的身份验证方法，请按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                          | **默认值** | **示例值**                                        | **描述**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建服务帐户时生成的 JSON 文件中的电子邮件地址。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 在创建服务帐户时生成的 JSON 文件中的私钥 ID。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 在创建服务帐户时生成的 JSON 文件中的私钥。 |

- 若要选择基于模拟的身份验证方法，请按如下方式配置 `StorageCredentialParams` ：

  - 使 VM 实例模拟服务帐户：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | **参数**                              | **默认值** | **示例值** | **描述**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用与 Compute Engine 绑定的服务帐号。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 要模拟的服务帐户。            |

  - 使服务帐户（称为元服务帐户）模拟另一个服务帐户（称为数据服务帐户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | **参数**                          | **默认值** | **示例值**                                        | **描述**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建元服务帐户时生成的 JSON 文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 在创建元服务帐户时生成的 JSON 文件中的私钥 ID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 在创建元服务帐户时生成的 JSON 文件中的私钥。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 要模拟的数据服务帐户。       |

#### 其他兼容 S3 的存储系统

如果选择其他兼容 S3 的存储系统，例如 MinIO，请按如下方式配置 `StorageCredentialParams` ：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                        | 必填 | 描述                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | 是      | 指定是否启用 SSL 连接。有效值： `true` 和 `false`。默认值： `true`。 |
| aws.s3.enable_path_style_access  | 是      | 指定是否启用路径样式 URL 访问。有效值： `true` 和 `false`。默认值： `false`。对于 MinIO，必须将值设置为 `true`。 |
| aws.s3.endpoint                  | 是      | 用于连接到与 S3 兼容的存储系统而不是 AWS S3的终端节点。 |
| aws.s3.access_key                | 是      | IAM 用户的访问密钥。 |
| aws.s3.secret_key                | 是      | IAM 用户的私有密钥。 |

#### Microsoft Azure 存储

##### Azure Blob 存储

如果选择 Blob 存储作为存储系统，请执行以下操作之一：

- 要选择共享密钥身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数              | 必填 | 描述                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | 是          | Blob 存储帐户的用户名。   |
  | azure.blob.shared_key      | 是          | Blob 存储帐户的共享密钥。 |

- 要选择 SAS 令牌身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数             | 必填 | 描述                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account| 是          | Blob 存储帐户的用户名。                   |
  | azure.blob.container      | 是          | 存储数据的 Blob 容器的名称。        |
  | azure.blob.sas_token      | 是          | 用于访问 Blob 存储帐户的 SAS 令牌。 |

##### Azure Data Lake Storage Gen1

如果选择 Data Lake Storage Gen1 作为存储系统，请执行以下操作之一：

- 要选择托管服务标识身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                            | 必填 | 描述                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是          | 指定是否启用托管服务标识身份验证方法。将值设置为 `true`。 |

- 要选择服务主体身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                 | 必填 | 描述                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是          | 客户端（应用程序）ID。                         |
  | azure.adls1.oauth2_credential | 是          | 新客户端（应用程序）机密的值。    |
  | azure.adls1.oauth2_endpoint   | 是          | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

##### Azure Data Lake Storage Gen2

如果选择 Data Lake Storage Gen2 作为存储系统，请执行以下操作之一：

- 要选择托管标识身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                           | 必填 | 描述                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | 是          | 指定是否启用托管标识身份验证方法。将值设置为 `true`。 |
  | azure.adls2.oauth2_tenant_id            | 是          | 要访问其数据的租户的 ID。          |
  | azure.adls2.oauth2_client_id            | 是          | 托管标识的客户端（应用程序）ID。         |

- 要选择共享密钥身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数               | 必填 | 描述                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | 是          | Data Lake Storage Gen2 存储帐户的用户名。 |
  | azure.adls2.shared_key      | 是          | Data Lake Storage Gen2 存储帐户的共享密钥。 |

- 要选择服务主体身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                      | 必填 | 描述                                              |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是          | 服务主体的客户端（应用程序）ID。        |
  | azure.adls2.oauth2_client_secret   | 是          | 新客户端（应用程序）机密的值。    |
  | azure.adls2.oauth2_client_endpoint | 是          | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

### opt_properties

指定一些可选参数，这些参数的设置将应用于整个加载作业。语法：

```Plain
PROPERTIES ("<key1>" = "<value1>"[, "<key2>" = "<value2>" ...])
```

支持以下参数：

- `timeout`

  指定加载作业的超时时间。单位：秒。默认超时期限为 4 小时。建议您指定短于 6 小时的超时时间。如果加载作业未在超时时间内完成，StarRocks 会取消加载作业，加载作业状态变为 **CANCELLED**。

  > **注意**
  >
  > 大多数情况下，您不需要设置超时时间。建议仅在加载作业无法在默认超时期限内完成时设置超时期限。

  使用以下公式推断超时期限：

  **超时时间>（要加载的数据文件总大小 x 要加载的数据文件和在数据文件上创建的物化视图总数）/平均加载速度**

  > **注意**
  >
  > “平均加载速度”是整个 StarRocks 集群的平均加载速度。每个集群的平均加载速度因服务器配置和集群允许的最大并发查询任务数而异。您可以根据历史加载作业的加载速度推断平均加载速度。

  假设您想将一个 1 GB 的数据文件加载到一个平均加载速度为 10 MB/s 的 StarRocks 集群中，该文件创建了两个物化视图。数据加载所需的时间约为 102 秒。

  （1 x 1024 x 3）/10 = 307.2（秒）

  对于此示例，我们建议您将超时时间设置为大于 308 秒的值。

- `max_filter_ratio`

  指定加载作业的最大容错范围。最大容错是由于数据质量不足而可以过滤掉的行的最大百分比。取值范围： `0`~`1`。默认值： `0`。

  - 如果将该参数设置为 `0`，StarRocks 在加载过程中不会忽略不合格的行。因此，如果源数据包含非限定行，则加载作业将失败。这有助于确保加载到 StarRocks 中的数据的正确性。

  - 如果将该参数设置为大于 `0`的值，StarRocks 在加载过程中可以忽略不合格的行。因此，即使源数据包含非限定行，加载作业也可以成功。

    > **注意**
    >
    > 由于数据质量不足而筛选出的行不包括由 WHERE 子句筛选出的行。

  如果加载作业因最大容错设置为 而失败，`0`则可以使用 [SHOW LOAD](../../../sql-reference/sql-statements/data-manipulation/SHOW_LOAD.md) 查看作业结果。然后，确定是否可以筛选出不合格的行。如果可以筛选出不合格的行，则根据作业结果`dpp.abnorm.ALL`中`dpp.norm.ALL`返回的值计算最大容错，调整最大容错，然后重新提交加载作业。最大误差容限的计算公式如下：

  `max_filter_ratio` = [`dpp.abnorm.ALL`/（`dpp.abnorm.ALL` + `dpp.norm.ALL`）]

  返回的值之和 `dpp.abnorm.ALL` `dpp.norm.ALL` 和是要加载的总行数。

- `log_rejected_record_num`

  指定可以记录的最大非限定数据行数。从 v3.1 开始支持此参数。有效值： `0`、 `-1`和任何非零正整数。默认值： `0`。
  
  - 该值 `0` 指定不会记录筛选出的数据行。
  - 该值 `-1` 指定将记录筛选出的所有数据行。
  - 非零正整数（例如） `n` 指定 `n` 每个 BE 上最多可以记录过滤掉的数据行。

- `load_mem_limit`

  指定可以提供给加载作业的最大内存量。单位：字节：默认内存限制为 2 GB。

- `strict_mode`

  指定是否启用[严格模式](../../../loading/load_concept/strict_mode.md)。有效值： `true` 和 `false`。默认值：`false`。 `true`指定启用严格模式，并指定 `false` 禁用严格模式。

- `timezone`

  指定加载作业的时区。默认值：`Asia/Shanghai`。时区设置会影响 strftime、alignment_timestamp 和 from_unixtime 等函数返回的结果。有关详细信息，请参阅[配置时区](../../../administration/timezone.md)。参数中指定的`timezone`时区是会话级别的时区。

- `priority`


  指定加载作业的优先级。有效值：`LOWEST`、`LOW`、`NORMAL`、`HIGH`和`HIGHEST`。默认值：`NORMAL`。Broker Load提供了[FE参数](../../../administration/FE_configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`，用于确定StarRocks集群中可以并发运行的最大Broker Load作业数量。如果在指定时间段内提交的Broker Load作业数量超过最大数量，则将根据其优先级等待调度过多的作业。

  您可以使用[ALTER LOAD](../../../sql-reference/sql-statements/data-manipulation/ALTER_LOAD.md)语句来更改处于`QUEUEING`或`LOADING`状态的现有加载作业的优先级。

  从StarRocks v2.5开始，StarRocks允许为Broker Load作业设置`priority`参数。

## 列映射

如果数据文件的列可以依次映射到StarRocks表的列，则无需配置数据文件和StarRocks表的列映射关系。

如果数据文件的列无法依次映射到StarRocks表的列，则需要使用`columns`参数配置数据文件和StarRocks表的列映射关系。这包括以下两个用例：

- **列数相同，但列顺序不同。** **此外，数据文件中的数据在加载到匹配的StarRocks表列中之前，不需要通过函数进行计算。**

  在`columns`参数中，需要按照与数据文件列排列顺序相同的顺序指定StarRocks表列的名称。

  例如，StarRocks表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列，数据文件也由三列组成，可以映射到StarRocks表列`col3`、`col2`和`col1`的顺序。在这种情况下，您需要指定`"columns: col3, col2, col1"`。

- **不同的列数和不同的列顺序。此外，数据文件中的数据需要经过函数计算，然后才能加载到匹配的StarRocks表列中。**

  在`columns`参数中，您需要按照与数据文件列排列相同的顺序指定StarRocks表列的名称，并指定要用于计算数据的函数。下面两个示例：

  - StarRocks表由三列组成，分别是`col1`、`col2`和`col3`。数据文件由四列组成，其中前三列可以依次映射到StarRocks表列`col1`、`col2`和`col3`，第四列不能映射到StarRocks表的任何列。这种情况下，您需要为数据文件的第四列临时指定一个名称，并且该临时名称必须与任何StarRocks表列名称不同。例如，可以指定`"columns: col1, col2, col3, temp"`，其中数据文件的第四列临时命名为`temp`。
  - StarRocks表由三列组成，分别是`year`、`month`和`day`。数据文件仅包含一列，该列以格式容纳日期和时间值`yyyy-mm-dd hh:mm:ss`。在这种情况下，您可以指定`"columns: col, year = year(col), month=month(col), day=day(col)"`，其中`col`是数据文件列的临时名称，函数`year = year(col)`、`month=month(col)`和`day=day(col)`用于从数据文件列`col`中提取数据，并将数据加载到映射的StarRocks表列中。例如，`year = year(col)`用于从`yyyy`数据文件列`col`中提取数据，并将数据加载到StarRocks表列`year`中。

有关详细示例，请参阅[配置列映射](#configure-column-mapping)。

## 相关配置项

[FE配置项](../../../administration/FE_configuration.md#fe-configuration-items)`max_broker_load_job_concurrency`指定了StarRocks集群中可以并发运行的最大Broker Load作业数量。

在StarRocks v2.4及之前版本中，如果特定时间段内提交的Broker Load作业总数超过最大数量，则过多的作业将根据提交时间进行排队和调度。

从StarRocks v2.5开始，如果在特定时间段内提交的Broker Load作业总数超过最大数量，则过多的作业会根据优先级进行排队和调度。您可以使用上述参数指定作业的优先级`priority`。您可以使用[ALTER LOAD](../data-manipulation/ALTER_LOAD.md)修改处于**QUEUEING**或**LOADING**状态的现有作业的优先级。

## 例子

本节以HDFS为例，介绍各种负载配置。

### 加载CSV数据

本节以CSV为例，介绍各种参数配置，满足各种负载需求。

#### 设置超时期限

您的StarRocks数据库`test_db`包含一个名为`table1`的表。该表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列。

您的数据文件`example1.csv`也包含三列，它们依次映射到`table1`的`col1`、`col2`和`col3`。

如果您希望在最多3600秒内将`example1.csv`中的所有数据加载到`table1`中，请运行以下命令：

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

#### 设置最大容错

您的StarRocks数据库`test_db`包含一个名为`table2`的表。该表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列。

您的数据文件`example2.csv`也包含三列，它们依次映射到`table2`的`col1`、`col2`和`col3`。

如果您希望将`example2.csv`中的所有数据加载到`table2`中，并设置最大容错为`0.1`，请运行以下命令：

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

#### 从文件路径加载所有数据文件

您的StarRocks数据库`test_db`包含一个名为`table3`的表。该表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列。

存储在HDFS集群路径`/user/starrocks/data/input/`中的所有数据文件也都由三列组成，它们依次映射到`table3`的`col1`、`col2`和`col3`。这些数据文件中使用的列分隔符是`\x01`。

如果您希望将存储在`hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/`路径中的所有这些数据文件加载到`table3`中，请运行以下命令：

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

您的StarRocks数据库`test_db`包含一个名为`table4`的表。该表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列。

您的数据文件`example4.csv`也包含三列，它们依次映射到`table4`的`col1`、`col2`和`col3`。

如果您希望使用NameNode的HA机制将`example4.csv`中的所有数据加载到`table4`中，请运行以下命令：

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

#### 设置Kerberos身份验证

您的StarRocks数据库`test_db`包含一个名为`table5`的表。该表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列。

您的数据文件`example5.csv`也包含三列，它们依次映射到`table5`的`col1`、`col2`和`col3`。

如果您希望在配置Kerberos身份验证并指定密钥表文件路径的情况下将`example5.csv`中的所有数据加载到`table5`中，请运行以下命令：

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

#### 撤消数据加载

您的StarRocks数据库`test_db`包含一个名为`table6`的表。该表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列。

您已通过运行Broker Load作业将`example6.csv`中的所有数据加载到`table6`中。

如果您希望撤销已加载的数据，请运行以下命令：

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

您的StarRocks数据库`test_db`包含一个名为`table7`的表。该表由三列组成，分别是`col1`、`col2`和`col3`按顺序排列。


您的数据文件 `example7.csv` 由三列组成，依次映射到 `table7` 的 `col1`、`col2` 和 `col3`。

如果要将 `example7.csv` 中的所有数据加载到 `table7` 的两个分区 `p1` 和 `p2` 中，请运行以下命令：

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

您的 StarRocks 数据库 `test_db` 包含一个名为 `table8` 的表。该表由三列组成，分别为 `col1`、`col2` 和 `col3`，按顺序排列。

您的数据文件 `example8.csv` 也包含三列，依次映射到 `table8` 的 `col2`、`col1` 和 `col3`。

如果要将 `example8.csv` 中的所有数据加载到 `table8` 中，请运行以下命令：

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
> 在前面的示例中，`example8.csv` 的列无法按照它们在 `table8` 中的排列顺序依次映射到 `table8` 的列。因此，您需要使用 `column_list` 来配置 `example8.csv` 和 `table8` 之间的列映射。

#### 设置筛选条件

您的 StarRocks 数据库 `test_db` 包含一个名为 `table9` 的表。该表由三列组成，分别为 `col1`、`col2` 和 `col3`，按顺序排列。

您的数据文件 `example9.csv` 也包含三列，依次映射到 `table9` 的 `col1`、`col2` 和 `col3`。

如果要仅加载 `example9.csv` 中第一列值大于 `20180601` 的数据记录到 `table9` 中，请运行以下命令：

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
> 在前面的示例中，`example9.csv` 的列可以按照它们在 `table9` 中的排列顺序依次映射到 `table9` 的列，但您需要使用 WHERE 子句来指定基于列的筛选条件。因此，您需要使用 `column_list` 来配置 `example9.csv` 和 `table9` 之间的列映射。

#### 将数据加载到包含 HLL 类型列的表中

您的 StarRocks 数据库 `test_db` 包含一个名为 `table10` 的表。该表由四列组成，分别为 `id`、`col1`、`col2` 和 `col3`，按顺序排列。`col1` 和 `col2` 被定义为 HLL 类型列。

您的数据文件 `example10.csv` 由三列组成，其中第一列映射到 `table10` 的 `id`，第二列和第三列依次映射到 `table10` 的 `col1` 和 `col2`。在加载到 `table10` 之前，`example10.csv` 中第二列和第三列的值可以通过函数转换为 HLL 类型数据。

如果要将 `example10.csv` 中的所有数据加载到 `table10` 中，请运行以下命令：

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
> 在前面的示例中，使用 `column_list` 将 `example10.csv` 的三列命名为 `id`、`temp1` 和 `temp2`。然后，使用函数进行数据转换，如下所示：
>
> - 使用 `hll_hash` 函数将 `example10.csv` 中的 `temp1` 和 `temp2` 的值转换为 HLL 类型数据，并将其映射到 `table10` 的 `col1` 和 `col2` 上。
>
> - 使用 `hll_empty` 函数将指定的默认值填充到 `table10` 的 `col3` 中。

有关 `hll_hash` 和 `hll_empty` 函数的用法，请参见 [hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) 和 [hll_empty](../../sql-functions/aggregate-functions/hll_empty.md)。

#### 从文件路径中提取分区字段值

Broker Load 支持根据目标 StarRocks 表的列定义，解析文件路径中特定分区字段的值。这个 StarRocks 功能类似于 Apache Spark™ 的分区发现功能。

您的 StarRocks 数据库 `test_db` 包含一个名为 `table11` 的表。该表由五列组成，分别为 `col1`、`col2`、`col3`、`city` 和 `utc_date`，按顺序排列。

您的 HDFS 集群中的文件路径 `/user/starrocks/data/input/dir/city=beijing` 包含以下数据文件：

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0000.csv`

- `/user/starrocks/data/input/dir/city=beijing/utc_date=2019-06-26/0001.csv`

这些数据文件由三列组成，依次映射到 `table11` 的 `col1`、`col2` 和 `col3`。

如果要从文件路径 `/user/starrocks/data/input/dir/city=beijing/utc_date=*/*` 加载所有数据文件中的数据到 `table11` 中，并且同时要提取文件路径中包含的分区字段 `city` 和 `utc_date` 的值，并将提取的值加载到 `table11` 的 `city` 和 `utc_date` 中，请运行以下命令：

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

#### 从包含 `%3A` 的文件路径中提取分区字段值

在 HDFS 中，文件路径不能包含冒号（:）。所有冒号（:）都将转换为 `%3A`。

您的 StarRocks 数据库 `test_db` 包含一个名为 `table12` 的表。该表由三列组成，分别为 `data_time`、`col1` 和 `col2`，按顺序排列。表结构如下：

```SQL
data_time DATETIME,
col1        INT,
col2        INT
```

您的 HDFS 集群中的文件路径 `/user/starrocks/data` 包含以下数据文件：

- `/user/starrocks/data/data_time=2020-02-17 00%3A00%3A00/example12.csv`

- `/user/starrocks/data/data_time=2020-02-18 00%3A00%3A00/example12.csv`

如果要从 `example12.csv` 加载所有数据到 `table12` 中，并且同时要从文件路径中提取分区字段 `data_time` 的值，并将提取的值加载到 `table12` 的 `data_time` 中，请运行以下命令：

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
> 在前面的示例中，从分区字段 `data_time` 中提取的值是包含 `%3A` 的字符串，例如 `2020-02-17 00%3A00%3A00`。因此，您需要使用 `str_to_date` 函数将字符串转换为 DATETIME 类型的数据，然后再将其加载到 `table8` 的 `data_time` 中。

#### 设置 `format_type_options`

您的 StarRocks 数据库 `test_db` 包含一个名为 `table13` 的表。该表由三列组成，分别为 `col1`、`col2` 和 `col3`，按顺序排列。

您的数据文件 `example13.csv` 也包含三列，依次映射到 `table13` 的 `col2`、`col1` 和 `col3`。

如果要从 `example13.csv` 加载所有数据到 `table13` 中，并且希望跳过 `example13.csv` 的前两行，删除列分隔符前后的空格，并将 `enclose` 设置为 `\`，`escape` 设置为 `\`，请运行以下命令：

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

本节介绍加载 Parquet 数据时需要注意的一些参数设置。

您的 StarRocks 数据库 `test_db` 包含一个名为 `table13` 的表。该表由三列组成，分别为 `col1`、`col2` 和 `col3`，按顺序排列。

您的数据文件 `example13.parquet` 也包含三列，依次映射到 `table13` 的 `col1`、`col2` 和 `col3`。

如果要从 `example13.parquet` 加载所有数据到 `table13` 中，请运行以下命令：

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
> 默认情况下，当您加载 Parquet 数据时，StarRocks会根据文件名是否包含扩展名 **.parquet** 来确定数据文件格式。如果文件名不包含扩展名 **.parquet**，则必须使用 `FORMAT AS` 来指定数据文件格式为 `Parquet`。

### 加载 ORC 数据

本节描述了加载 ORC 数据时需要注意的一些参数设置。

您的 StarRocks 数据库 `test_db` 包含一张名为 `table14` 的表。该表包括三列，分别为 `col1`、`col2` 和 `col3`。

您的数据文件 `example14.orc` 也包含三列，它们依次映射到 `table14` 的 `col1`、`col2` 和 `col3`。

如果您想要将 `example14.orc` 中的所有数据加载到 `table14` 中，请运行以下命令：

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
> - 默认情况下，当您加载 ORC 数据时，StarRocks会根据文件名是否包含扩展名 **.orc** 来确定数据文件格式。如果文件名不包含扩展名 **.orc**，则必须使用 `FORMAT AS` 来指定数据文件格式为 `ORC`。
>
> - 在 StarRocks v2.3 及更早版本中，如果数据文件包含 ARRAY 类型的列，您必须确保 ORC 数据文件的列与 StarRocks 表中的映射列同名，并且这些列不能在 SET 子句中指定。