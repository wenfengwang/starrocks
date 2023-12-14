---
displayed_sidebar: "Chinese"
---

# Dataphin

Dataphin是阿里巴巴集团OneData数据治理方法的云输出产品。它提供了数据集成、构建、管理和利用的一站式解决方案，贯穿大数据的整个生命周期，旨在帮助企业显著提高数据治理水平，构建高可靠质量、便捷消费、安全经济生产的企业级数据中台。Dataphin提供了各种计算平台支持和可扩展的开放能力，以满足各行业企业的平台技术架构和特定需求。

有几种方法可以将Dataphin与StarRocks集成：

- 作为数据集成的源头或目的地数据源。数据可以从StarRocks读取并推送到其他数据源，或者可以从其他数据源拉取并写入StarRocks。

- 作为flink SQL和datastream开发的源头表（无界扫描）、维表（有界扫描）或结果表（流式下沉和批量下沉）。

- 作为数据仓库或数据集市。StarRocks可以注册为计算数据源，可用于SQL脚本开发、调度、数据质量检测、安全标识和其他数据研究和治理任务。

## 数据集成

您可以创建StarRocks数据源并在线下集成任务中将其用作源数据库或目的地数据库。具体步骤如下：

### 创建一个StarRocks数据源

#### 基本信息

![创建StarRocks数据源-1](../../assets/Dataphin/create_sr_datasource_1.png)

- **名称**：必填。输入数据源名称。只能包含中文字符、字母、数字、下划线(_)和连字符(-)。长度不能超过64个字符。

- **数据源编码**：可选。配置数据源编码后，可以使用`数据源编码.表`或`数据源编码.模式.表`格式引用数据源中的Flink SQL。如果要在对应环境下实现自动访问数据源，可使用`${数据源编码}.表`或`${数据源编码}.模式.表`格式访问。

  > **注意**
  >
  > 目前仅支持MySQL、Hologres和MaxCompute数据源。

- **支持场景**：数据源可以应用的场景。

- **描述**：可选。您可以输入数据源的简要描述。最多允许128个字符。

- **环境**：如果业务数据源区分生产数据源和开发数据源，请选择**Prod and Dev**。如果业务数据源不区分生产和开发数据源，请选择**Prod**。

- **标签**：您可以选择标签来标记数据源。

#### 配置信息

![创建StarRocks数据源-2](../../assets/Dataphin/create_sr_datasource_2.png)

- **JDBC URL**：必填。格式为`jdbc:mysql://<主机>:<端口>/<数据库名>`。`主机`是StarRocks集群中FE（前端）主机的IP地址，`端口`是FE的查询端口，`数据库名`是数据库名称。

- **Load URL**：必填。格式为`fe_ip:http_port;fe_ip:http_port`。`fe_ip`是FE（前端）的主机，`http_port`是FE的端口。

- **用户名**：必填。数据库的用户名。

- **密码**：必填。数据库的密码。

#### 高级设置

![创建StarRocks数据源-3](../../assets/Dataphin/create_sr_datasource_3.png)

- **连接超时时间**：数据库的连接超时时间（以毫秒为单位）。默认值为900000毫秒（15分钟）。

- **套接字超时时间**：数据库的套接字超时时间（以毫秒为单位）。默认值为1800000毫秒（30分钟）。

### 从StarRocks数据源读取数据并将数据写入其他数据源

#### 将StarRocks输入组件拖动至离线集成任务画布中

![从StarRocks读取数据-1](../../assets/Dataphin/read_from_sr_datasource_1.png)

#### StarRocks输入组件配置

![从StarRocks读取数据-2](../../assets/Dataphin/read_from_sr_datasource_2.png)

- **步骤名称**：根据当前组件的场景和位置输入合适的名称。

- **数据源**：选择在Dataphin上创建的StarRocks数据源或项目。需要有数据源的读取权限。如果没有满足需求的数据源，您可以添加数据源或申请相关权限。

- **源表**：选择与输入相同表结构的单个表或多个表。

- **表格**：从下拉列表中选择StarRocks数据源中的表。

- **分裂键**：与并发配置一起使用。您可以使用源数据表中的列作为分裂键。建议使用主键或有索引的列作为分裂键。

- **批次号**：一批中提取的数据记录数。

- **输入过滤**：可选。

  在以下两种情况下，需要填写过滤信息：

  - 如果要过滤某部分数据。
  - 如果需要每天增量追加数据或获取全量数据，需要填写日期，其值设置为Dataphin控制台的系统时间。例如，StarRocks中的交易表及其交易创建日期设置为`${bizdate}`。

- **输出字段**：根据输入表信息列出相关字段。您可以重命名、删除、添加和移动字段。通常，字段在输入阶段被重命名，以增加下游数据的可读性或方便输出期间字段的映射。在应用场景中不需要相关字段时，字段可以在输入阶段被删除。字段的顺序被修改以确保在合并多个输入数据或在下游端输出时，通过将具有不同名称的字段映射至同一行时能够有效地合并数据或映射输出数据。

#### 选择并配置一个输出组件作为目的地数据源

![从StarRocks读取数据-3](../../assets/Dataphin/read_from_sr_datasource_3.png)

### 从其他数据源读取数据并将数据写入StarRocks数据源

#### 在离线集成任务中配置输入组件，并选择并配置StarRocks输出组件作为目的地数据源

![向StarRocks写入数据-1](../../assets/Dataphin/write_to_sr_datasource_1.png)

#### 配置StarRocks输出组件

![向StarRocks写入数据-2](../../assets/Dataphin/write_to_sr_datasource_2.png)

- **步骤名称**：根据当前组件的场景和位置输入合适的名称。

- **数据源**：选择在StarRocks中创建的Dataphin数据源或项目。配置人员具有同步写入权限的数据源。如果没有满足需求的数据源，您可以添加数据源或申请相关权限。

- **表格**：从下拉列表中选择StarRocks数据源中的表。

- **一键生成目标表格**：如果尚未在StarRocks数据源中创建目标表格，您可以自动获取从上游读取的字段的名称、类型和备注，并生成表格创建语句。单击一键生成目标表格。

- **CSV导入列分隔符**：使用StreamLoad CSV导入。您可以配置CSV导入列分隔符。默认值`\t`。不要在此处指定默认值。如果数据本身包含`\t`，必须使用其他字符作为分隔符。

- **CSV导入行分隔符**：使用StreamLoad CSV导入。您可以配置CSV导入行分隔符。默认值：`\n`。不要在此处指定默认值。如果数据本身包含`\n`，必须使用其他字符作为分隔符。

- **解析方案**：可选。数据写入StarRocks数据源之前或之后的一些特殊处理。在数据写入到StarRocks数据源之前执行准备语句，并在数据写入后执行完成语句。

- **字段映射**：您可以手动选择字段进行映射，或者使用基于名称或位置的映射来同时处理多个字段，基于上游输入和目的地表格中的字段。

## 实时开发

### 简要介绍

StarRocks是一个快速可扩展的实时分析数据库。它常用于实时计算，以满足实时数据分析和查询的需求。它广泛用于企业实时计算场景。可用于实时业务监控和分析、实时用户行为分析、实时广告竞价系统、实时风险控制、防欺诈、实时监控和预警等应用场景。通过实时分析和查询数据，企业可以快速了解业务状况，优化决策，提供更好的服务并保护自己的利益。

### StarRocks连接器

StarRocks连接器支持以下信息：

| **类别**                                           | **数据和数字**                       |
| -------------------------------------------------- | ----------------------------------- |

| 支持的类型                                           | 源表、维度表、结果表                |
| 运行模式                                             | 流模式和批处理模式                   |
| 数据格式                                             | JSON 和 CSV                          |
| 特殊指标                                             | 无                                   |
| API 类型                                             | 数据流和 SQL                        |
| 是否支持更新或删除结果表中的数据？                    | 是                                   |

### 如何使用？

Dataphin支持将StarRocks数据源作为实时计算的读写目标。您可以创建StarRocks元表并将其用于实时计算任务：

#### 创建StarRocks元表

1. 进入**Dataphin** > **研发** > **开发** > **表**。

2. 点击**创建**选择实时计算表。

   ![创建StarRocks元表 - 步骤 1](../../assets/Dataphin/create_sr_metatable_1.png)

   - **表类型**：选择**元表**。

   - **元表名称**：输入元表的名称。名称不可更改。

   - **数据源**：选择StarRocks数据源。

   - **目录**：选择要创建表的目录。

   - **描述**：可选。

   ![创建StarRocks元表 - 步骤 2](../../assets/Dataphin/create_sr_metatable_2.png)

3. 创建元表后，您可以编辑元表，包括修改数据源、源表、元表字段，以及配置元表参数。

   ![编辑StarRocks元表](../../assets/Dataphin/edit_sr_metatable_1.png)

4. 提交元表。

#### 创建Flink SQL任务将数据实时从Kafka写入StarRocks

1. 进入**Dataphin** > **研发** > **开发** > **计算任务**。

2. 点击**创建Flink SQL任务**。

   ![创建Flink SQL任务 - 步骤 2](../../assets/Dataphin/create_flink_task_step2.png)

3. 编辑Flink SQL代码并预编译。Kafka元表作为输入表，StarRocks元表作为输出表。

   ![创建Flink SQL任务 - 步骤 3 - 1](../../assets/Dataphin/create_flink_task_step3-1.png)
   ![创建Flink SQL任务 - 步骤 3 - 2](../../assets/Dataphin/create_flink_task_step3-2.png)

4. 预编译成功后，可以进行调试并提交代码。

5. 在开发环境中进行测试，可通过打印日志和编写测试表来进行。测试表可以在元表 > 属性 > 调试测试配置中设置。

   ![创建Flink SQL任务 - 步骤 5 - 1](../../assets/Dataphin/create_flink_task_step5-1.png)
   ![创建Flink SQL任务 - 步骤 5 - 2](../../assets/Dataphin/create_flink_task_step5-2.png)

6. 在开发环境中的任务正常运行后，您可以将任务和元表发布到生产环境。

   ![创建Flink SQL任务 - 步骤 6](../../assets/Dataphin/create_flink_task_step6.png)

7. 在生产环境中启动任务以实时将数据从Kafka写入StarRocks。您可以查看运行分析中每个指标的状态和日志，以了解任务运行状态，或者为任务配置监控警报。

   ![创建Flink SQL任务 - 步骤 7 - 1](../../assets/Dataphin/create_flink_task_step7-1.png)
   ![创建Flink SQL任务 - 步骤 7 - 2](../../assets/Dataphin/create_flink_task_step7-2.png)

## 数据仓库或数据集市

### 先决条件

- StarRocks版本为3.0.6或更高。

- 已安装Dataphin，版本为3.12或更高。

- 必须启用统计收集。StarRocks安装后，默认启用收集。更多信息，请参阅[Gather statistics for CBO](../../using_starrocks/Cost_based_optimizer.md)。

- 支持StarRocks内部目录（默认目录），不支持外部目录。

### 连接配置

#### 元数据仓库设置

Dataphin可以基于元数据呈现和显示信息，包括表的使用信息和元数据更改。您可以使用StarRocks处理和计算元数据。因此，在使用之前，您需要初始化元数据计算引擎（元数据仓库）。步骤如下：

1. 使用管理员帐户登录到Dataphin元数据仓库租户。

2. 进入Administration > System > Metadata Warehouse Configuration

   a. 点击启动

   b. 选择StarRocks

   c. 配置参数。通过测试连接后，点击下一步。

   d. 完成元数据仓库初始化

   ![元数据仓库设置](../../assets/Dataphin/metadata_warehouse_settings_1.png)

参数说明如下：

- **JDBC URL**：JDBC连接字符串，分为两部分：

  - 第一部分：格式为`jdbc:mysql://<Host>:<Port>/`。`Host`是StarRocks集群中FE主机的IP地址。`Port`是FE的查询端口。默认值：`9030`。

  - 第二部分：格式为`database?key1=value1&key2=value2`，其中`database`是用于元数据计算的StarRocks数据库的名称，这是必需的。'?'后的参数是可选的。

- **Load URL**：格式为`fe_ip:http_port;fe_ip:http_port`。`fe_ip`是FE（前端）的主机，`http_port`是FE的端口。

- **Username**：用于连接到StarRocks的用户名。

  用户需要对JDBC URL中指定的数据库有读写权限，并且必须具有以下数据库和表的访问权限：

  - 信息模式中的所有表

  - _statistics_.column_statistics

  - _statistics_.table_statistic_v1

- **Password**：StarRocks的连接密码。

- **Meta Project**：Dataphin中用于元数据处理的项目名称。仅在Dataphin系统内使用。我们建议您将`dataphin_meta`作为项目名称。

#### 创建StarRocks项目并开始数据开发

要开始数据开发，请按照以下步骤进行：

1. 计算设置。

2. 创建StarRocks计算源。

3. 创建一个项目。

4. 创建StarRocks SQL任务。

##### 计算设置

计算设置设置了租户的计算引擎类型和集群地址。具体步骤如下：

1. 以系统管理员或超级管理员身份登录到Dataphin。

2. 进入Administration > System > Computation Configuration。

3. 选择**StarRocks**并点击**下一步**。

4. 输入JDBC URL并进行验证。JDBC URL的格式为`jdbc:mysql://<Host>:<Port>/`。`Host`是StarRocks集群中的FE主机的IP地址。`Port`是FE的查询端口。默认值：`9030`。

##### StarRocks计算源

计算源是Dataphin的一个概念。其主要目的是将Dataphin项目空间与StarRocks存储计算空间（数据库）进行绑定和注册。您必须为每个项目创建一个计算源。具体的配置信息如下：

1. 以系统管理员或超级管理员身份登录到Dataphin。

2. 进入Planning > Engine。

3. 点击右上角的**添加计算引擎**，创建一个计算源。

具体的配置信息如下：

1. **基本信息**

   ![创建计算引擎 - 1](../../assets/Dataphin/create_compute_engine_1.png)

   - **计算引擎类型**：选择**StarRocks**。

   - **计算引擎名称**：建议使用要创建的项目相同的名称。对于开发项目，添加后缀`_dev`。

   - **描述**：可选。输入计算源的描述。

2. **配置信息**

   ![创建计算引擎 - 2](../../assets/Dataphin/create_compute_engine_2.png)

   - **JDBC URL**：格式为`jdbc:mysql://<Host>:<Port>/`。`Host`是StarRocks集群中的FE主机的IP地址。`Port`是FE的查询端口。默认值：`9030`。

   - **Load URL**：格式为`fe_ip:http_port;fe_ip:http_port`。`fe_ip`是FE（前端）的主机，`http_port`是FE的端口。

   - **Username**：用于连接到StarRocks的用户名。

   - **Password**：StarRocks的密码。

   - **任务资源组**：您可以为不同优先级的任务指定不同的StarRocks资源组。当您选择不指定资源组时，StarRocks引擎会确定要执行的资源组。当您选择指定资源组时，Dataphin将不同优先级的任务分配到指定的资源组。如果在SQL任务的代码中或逻辑表的物化配置中指定了资源组，则在执行任务时将忽略计算源任务的资源组配置。

   ![创建计算引擎-3](../../assets/Dataphin/create_compute_engine_3.png)

##### Dataphin项目

创建计算源后，您可以将其绑定到Dataphin项目。Dataphin项目管理项目成员、StarRocks存储和计算空间，并管理和维护计算任务。

要创建Dataphin项目，请按照以下步骤操作：

1. 以系统管理员或超级管理员身份登录Dataphin。

2. 转到**规划** > **项目管理**。

3. 单击右上角的**创建项目**来创建项目。

4. 输入基本信息，并从线下引擎中选择在上一步创建的StarRocks引擎。

5. 单击**创建**。

##### StarRocks SQL

创建项目后，您可以创建一个StarRocks SQL任务，以在StarRocks上执行DDL或DML操作。

具体步骤如下：

1. 转到**研发** > **开发**。

2. 单击右上角的‘+’来创建StarRocks SQL任务。

   ![配置Dataphin项目-1](../../assets/Dataphin/configure_dataphin_project_1.png)

3. 输入名称和调度类型以创建一个SQL任务。

4. 在编辑器中输入SQL，开始在StarRocks上进行DDL和DML操作。

   ![配置Dataphin项目-2](../../assets/Dataphin/configure_dataphin_project_2.png)