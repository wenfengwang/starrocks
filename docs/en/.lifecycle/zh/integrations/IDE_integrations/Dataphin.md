---
displayed_sidebar: English
---

# Dataphin

Dataphin是阿里巴巴集团OneData数据治理方法论的云端输出，它提供了一个一站式解决方案，涵盖了大数据全生命周期的数据集成、构建、管理和利用，旨在帮助企业显著提升数据治理水平，构建高品质、可靠、便于消费、安全且经济的企业级数据中台。Dataphin提供多样化的计算平台支持和可扩展的开放能力，以满足不同行业企业的平台技术架构和特定需求。

将Dataphin与StarRocks集成的方法包括：

- 作为数据集成的源数据源或目标数据源。可以从StarRocks读取数据并推送到其他数据源，或者从其他数据源拉取数据并写入StarRocks。

- 作为Flink SQL和DataStream开发的源表（无界扫描）、维度表（有界扫描）或结果表（流式Sink和批量Sink）。

- 作为数据仓库或数据集市。StarRocks可以注册为计算源，用于SQL脚本开发、调度、数据质量检测、安全识别以及其他数据研究和治理任务。

## 数据集成

您可以创建StarRocks数据源，并在离线集成任务中将其作为源数据库或目标数据库。操作步骤如下：

### 创建StarRocks数据源

#### 基本信息

![Create a StarRocks data source - 1](../../assets/Dataphin/create_sr_datasource_1.png)

- **名称**：必填。输入数据源名称，只能包含汉字、字母、数字、下划线（_）和连字符（-），长度不能超过64个字符。

- **数据源代码**：可选。配置数据源代码后，您可以使用`data_source_code.table`或`data_source_code.schema.table`格式引用数据源中的Flink SQL。若要在相应环境中自动访问数据源，请使用`${data_source_code}.table`或`${data_source_code}.schema.table`格式访问。

    > **注意**
    > 目前仅支持MySQL、Hologres和MaxCompute数据源。

- **支持场景**：数据源可以应用的场景。

- **描述**：可选。您可以输入数据源的简要描述，最多允许128个字符。

- **环境**：如果业务数据源区分生产数据源和开发数据源，选择**Prod and Dev**；如果不区分，则选择**Prod**。

- **标签**：您可以选择标签来标记数据源。

#### 配置信息

![Create a StarRocks data source - 2](../../assets/Dataphin/create_sr_datasource_2.png)

- **JDBC URL**：必填。格式为`jdbc:mysql://<host>:<port>/<dbname>`。`host`是StarRocks集群中FE（前端）主机的IP地址，`port`是FE的查询端口，`dbname`是数据库名称。

- **Load URL**：必填。格式为`fe_ip:http_port;fe_ip:http_port`。`fe_ip`是FE（前端）的主机，`http_port`是FE的端口。

- **用户名**：必填。数据库的用户名。

- **密码**：必填。数据库的密码。

#### 高级设置

![Create a StarRocks data source - 3](../../assets/Dataphin/create_sr_datasource_3.png)

- **connectTimeout**：数据库的连接超时时间（单位为毫秒），默认值为900000毫秒（15分钟）。

- **socketTimeout**：数据库的socket超时时间（单位为毫秒），默认值为1800000毫秒（30分钟）。

### 从StarRocks数据源读取数据并写入其他数据源

#### 将StarRocks输入组件拖动到离线集成任务画布上

![Read data from StarRocks - 1](../../assets/Dataphin/read_from_sr_datasource_1.png)

#### StarRocks输入组件配置

![Read data from StarRocks - 2](../../assets/Dataphin/read_from_sr_datasource_2.png)

- **步骤名称**：根据当前组件的场景和位置输入合适的名称。

- **数据源**：选择在Dataphin上创建的StarRocks数据源或项目。需要数据源的读取权限。如果没有合适的数据源，您可以添加数据源或申请相关权限。

- **源表**：选择作为输入的单个表或多个具有相同表结构的表。

- **表**：从下拉列表中选择StarRocks数据源中的表。

- **分割键**：与并发配置一起使用。您可以使用源数据表中的列作为分割键，建议使用主键或索引列作为分割键。

- **批号**：每批次提取的数据记录数。

- **输入过滤**：可选。

  在以下两种情况下，您需要填写过滤信息：

  - 如果您想过滤掉某部分数据。
  - 如果您需要每天增量追加数据或获取全量数据，则需要填写日期，其值设置为Dataphin控制台的系统时间。例如，在StarRocks中的交易表，其交易创建日期设置为`${bizdate}`。

- **输出字段**：根据输入表信息列出相关字段。您可以重新命名、删除、添加和移动字段。通常，字段被重命名是为了提高下游数据的可读性或便于输出时字段的映射。在输入阶段，可以删除不需要的字段。更改字段顺序是为了确保在下游合并或输出多个输入数据时，可以通过将不同名称的字段映射到同一行来有效合并数据或映射输出数据。

#### 选择并配置输出组件作为目标数据源

![Read data from StarRocks - 3](../../assets/Dataphin/read_from_sr_datasource_3.png)

### 从其他数据源读取数据并写入StarRocks数据源

#### 在离线集成任务中配置输入组件，并选择并配置StarRocks输出组件作为目标数据源

![Write data to StarRocks - 1](../../assets/Dataphin/write_to_sr_datasource_1.png)

#### 配置StarRocks输出组件

![Write data to StarRocks - 2](../../assets/Dataphin/write_to_sr_datasource_2.png)

- **步骤名称**：根据当前组件的场景和位置输入合适的名称。

- **数据源**：选择在StarRocks中创建的Dataphin数据源或项目。配置人员应具有数据源的同步写入权限。如果数据源不符合要求，您可以添加数据源或申请相关权限。

- **表**：从下拉列表中选择StarRocks数据源中的表。

- **一键生成目标表**：如果您尚未在StarRocks数据源中创建目标表，可以自动获取从上游读取的字段名称、类型和备注，并生成建表语句。点击即可一键生成目标表。

- **CSV导入列分隔符**：使用StreamLoad CSV导入。您可以配置CSV导入列分隔符，默认值为`\t`。如果数据本身包含`\t`，则必须使用其他字符作为分隔符。

- **CSV导入行分隔符**：使用StreamLoad CSV导入。您可以配置CSV导入行分隔符，默认值为`\n`。如果数据本身包含`\n`，则必须使用其他字符作为分隔符。

- **解析方案**：可选。是数据写入前或写入后的特殊处理。准备语句在数据写入StarRocks数据源之前执行，完成语句在数据写入后执行。

- **字段映射**：您可以手动选择字段进行映射，也可以基于名称或位置一次性处理多个字段，根据上游输入的字段和目标表中的字段进行处理。

## 实时开发

### 简介

StarRocks是一个快速且可扩展的实时分析数据库。它常用于实时计算中的数据读写，以满足实时数据分析和查询的需求。广泛应用于企业实时计算场景，如实时业务监控分析、实时用户行为分析、实时广告竞价系统、实时风险控制、反欺诈、实时监控预警等。通过实时分析和查询数据，企业可以快速了解业务状况，优化决策，提供更好的服务，保护自身利益。

### StarRocks连接器

StarRocks连接器支持以下信息：

|**类别**|**事实与数据**|
|---|---|
|支持的类型|源表、维度表、结果表|
|运行模式|流模式和批模式|
|数据格式|JSON和CSV|
|特殊指标|无|
|API类型|DataStream和SQL|
|支持在结果表中更新或删除数据吗？|是|

### 如何使用？

Dataphin支持将StarRocks数据源作为实时计算的读写目标。您可以创建StarRocks元数据表，并将其用于实时计算任务：
```
#### 创建 StarRocks 元表

1. 转到 **Dataphin** > **R & D** > **Develop** > **Tables**。

2. 单击 **创建**，选择实时计算表。

   ![创建 StarRocks 元表 - 1](../../assets/Dataphin/create_sr_metatable_1.png)

-    **表类型**：选择 **元表**。

-    **元表**：输入元表的名称。名称一经创建不可更改。

-    **数据源**：选择 StarRocks 数据源。

-    **目录**：选择要创建表的目录。

-    **描述**：可选。

   ![创建 StarRocks 元表 - 2](../../assets/Dataphin/create_sr_metatable_2.png)

3. 创建元表后，您可以编辑元表，包括修改数据源、源表、元表字段和配置元表参数。

   ![编辑 StarRocks 元表](../../assets/Dataphin/edit_sr_metatable_1.png)

4. 提交元表。

#### 创建 Flink SQL 任务以实时将数据从 Kafka 写入 StarRocks

1. 进入 **Dataphin** > **R & D** > **Develop** > **Computing Tasks**。

2. 单击 **创建 Flink SQL 任务**。

   ![创建 Flink SQL 任务 - 步骤 2](../../assets/Dataphin/create_flink_task_step2.png)

3. 编辑 Flink SQL 代码并进行预编译。Kafka 元表作为输入表，StarRocks 元表作为输出表。

   ![创建 Flink SQL 任务 - 步骤 3 - 1](../../assets/Dataphin/create_flink_task_step3-1.png)
   ![创建 Flink SQL 任务 - 步骤 3 - 2](../../assets/Dataphin/create_flink_task_step3-2.png)

4. 预编译成功后，可以进行调试并提交代码。

5. 开发环境中的测试可以通过打印日志和写入测试表来进行。测试表可以在元表 > 属性 > 调试测试配置中设置。

   ![创建 Flink SQL 任务 - 步骤 5 - 1](../../assets/Dataphin/create_flink_task_step5-1.png)
   ![创建 Flink SQL 任务 - 步骤 5 - 2](../../assets/Dataphin/create_flink_task_step5-2.png)

6. 开发环境中的任务正常运行后，可以将任务和使用的元表发布到生产环境。

   ![创建 Flink SQL 任务 - 步骤 6](../../assets/Dataphin/create_flink_task_step6.png)

7. 在生产环境中启动任务，实时将数据从 Kafka 写入 StarRocks。您可以查看运行分析中各个指标的状态和日志，了解任务运行状态，或为任务配置监控警报。

   ![创建 Flink SQL 任务 - 步骤 7 - 1](../../assets/Dataphin/create_flink_task_step7-1.png)
   ![创建 Flink SQL 任务 - 步骤 7 - 2](../../assets/Dataphin/create_flink_task_step7-2.png)

## 数据仓库或数据集市

### 先决条件

- StarRocks 版本为 3.0.6 或更高。

- 已安装 Dataphin，且 Dataphin 版本为 3.12 或更高。

- 必须启用统计收集。安装 StarRocks 后，默认启用收集。有关详细信息，请参见 [Gather statistics for CBO](../../using_starrocks/Cost_based_optimizer.md)。

- StarRocks 支持内部目录（默认目录），不支持外部目录。

### 连接配置

#### 元数据仓库设置

Dataphin 可以基于元数据展示和显示信息，包括表使用信息和元数据变更。您可以使用 StarRocks 来处理和计算元数据。因此，在使用元数据计算引擎（元数据仓库）之前，需要对其进行初始化。步骤如下：

1. 使用管理员账号登录 Dataphin 元数据仓库租户。

2. 转至 **管理** > **系统** > **元数据仓库配置**。

   a. 单击 **开始**。

   b. 选择 **StarRocks**。

   c. 配置参数。测试连接通过后，点击 **下一步**。

   d. 完成元数据仓库初始化。

   ![元数据仓库设置](../../assets/Dataphin/metadata_warehouse_settings_1.png)

参数说明如下：

- **JDBC URL**：JDBC 连接字符串，分为两部分：

-   第一部分：格式为 `jdbc:mysql://<Host>:<Port>/`。`Host` 是 StarRocks 集群中 FE 主机的 IP 地址。`Port` 是 FE 的查询端口。默认值：`9030`。

-   第二部分：格式为 `database?key1=value1&key2=value2`，其中 `database` 是用于元数据计算的 StarRocks 数据库名称，必填。`?` 后的参数是可选的。

- **Load URL**：格式为 `fe_ip:http_port;fe_ip:http_port`。`fe_ip` 是 FE（前端）的主机，`http_port` 是 FE 的端口。

- **用户名**：用于连接 StarRocks 的用户名。

  用户需要对 JDBC URL 中指定的数据库具有读写权限，并且必须具有以下数据库和表的访问权限：

-   Information Schema 中的所有表

-   `_statistics_.column_statistics`

-   `_statistics_.table_statistic_v1`

- **密码**：StarRocks 连接的密码。

- **元项目**：Dataphin 中用于元数据处理的项目名称。它仅在 Dataphin 系统内使用。建议使用 `dataphin_meta` 作为项目名称。

#### 创建 StarRocks 项目并开始数据开发

要开始数据开发，请按以下步骤操作：

1. 计算设置。

2. 创建 StarRocks 计算源。

3. 创建项目。

4. 创建 StarRocks SQL 任务。

##### 计算设置

计算设置设定租户的计算引擎类型和集群地址。详细步骤如下：

1. 以系统管理员或超级管理员身份登录 Dataphin。

2. 转至 **管理** > **系统** > **计算配置**。

3. 选择 **StarRocks** 并单击 **下一步**。

4. 输入 JDBC URL 并进行验证。JDBC URL 的格式为 `jdbc:mysql://<Host>:<Port>/`。`Host` 是 StarRocks 集群中 FE 主机的 IP 地址。`Port` 是 FE 的查询端口。默认值：`9030`。

##### StarRocks 计算源

计算源是 Dataphin 的一个概念。其主要目的是将 Dataphin 项目空间与 StarRocks 存储计算空间（数据库）绑定注册。您必须为每个项目创建一个计算源。详细步骤如下：

1. 以系统管理员或超级管理员身份登录 Dataphin。

2. 转到 **规划** > **引擎**。

3. 单击右上角的 **添加计算引擎** 来创建一个计算源。

详细配置信息如下：

1. **基本信息**

   ![创建计算引擎 - 1](../../assets/Dataphin/create_compute_engine_1.png)

-    **计算引擎类型**：选择 **StarRocks**。

-    **计算引擎名称**：建议使用与要创建的项目相同的名称。对于开发项目，添加后缀 `_dev`。

-    **描述**：可选。输入计算源的描述。

2. **配置信息**

   ![创建计算引擎 - 2](../../assets/Dataphin/create_compute_engine_2.png)

-    **JDBC URL**：格式为 `jdbc:mysql://<Host>:<Port>/`。`Host` 是 StarRocks 集群中 FE 主机的 IP 地址。`Port` 是 FE 的查询端口。默认值：`9030`。

-    **Load URL**：格式为 `fe_ip:http_port;fe_ip:http_port`。`fe_ip` 是 FE（前端）的主机，`http_port` 是 FE 的端口。

-    **用户名**：用于连接 StarRocks 的用户名。

-    **密码**：StarRocks 的密码。

-    **任务资源组**：您可以为不同优先级的任务指定不同的 StarRocks 资源组。当您选择不指定资源组时，StarRocks 引擎将决定要执行的资源组。当选择指定资源组时，Dataphin 会根据不同优先级将任务分配到指定的资源组。如果在 SQL 任务的代码中或逻辑表的物化配置中指定了资源组，则在执行任务时，计算源任务的资源组配置将被忽略。

   ![创建计算引擎 - 3](../../assets/Dataphin/create_compute_engine_3.png)

##### Dataphin 项目

创建计算源后，您可以将其绑定到 Dataphin 项目。Dataphin 项目管理项目成员、StarRocks 存储和计算空间，并管理和维护计算任务。

要创建 Dataphin 项目，请执行以下步骤：

1. 以系统管理员或超级管理员身份登录 Dataphin。

2. 转到 **规划** > **项目管理**。

3. 单击右上角的 **创建项目**。

4. 输入基本信息，从离线引擎中选择上一步创建的 StarRocks 引擎。

5. 单击 **创建**。

##### StarRocks SQL

创建项目后，您可以创建 StarRocks SQL 任务来执行 StarRocks 上的 DDL 或 DML 操作。

详细步骤如下：

1. 转到 **R & D** > **Develop**。

2. 单击右上角的 '+' 创建 StarRocks SQL 任务。

   ![配置 Dataphin 项目 - 1](../../assets/Dataphin/configure_dataphin_project_1.png)

3. 输入名称和调度类型来创建 SQL 任务。

4. 在编辑器中输入 SQL，开始对 StarRocks 进行 DDL 和 DML 操作。

   ![配置 Dataphin 项目 - 2](../../assets/Dataphin/configure_dataphin_project_2.png)
创建项目后，您可以创建 StarRocks SQL 任务来对 StarRocks 执行 DDL 或 DML 操作。

详细步骤如下：

1. 转到 **R&D** > **Develop**。

2. 点击右上角的“+”创建 StarRocks SQL 任务。

   ![配置 Dataphin 项目 - 1](../../assets/Dataphin/configure_dataphin_project_1.png)

3. 输入名称和调度类型以创建 SQL 任务。

4. 在编辑器中输入 SQL，开始对 StarRocks 进行 DDL 和 DML 操作。
