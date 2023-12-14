---
displayed_sidebar: "Chinese"
---

# Dataphin

Dataphin是阿里巴巴集团OneData数据治理方法论内部实践的云化输出，一站式提供数据采、建、管、用全生命周期的大数据能力，以助力企业显著提升数据治理水平，构建质量可靠、消费便捷、生产安全经济的企业级数据中台。Dataphin提供多种计算平台支持及可拓展的开放能力，以适应各行业企业的平台技术架构和特定诉求。

Dataphin与StarRocks集成分为以下几种场景：

- 作为数据集成的来源或目标数据源，支持从StarRocks中读取数据到其他数据源，或从其他数据源写入数据到StarRocks。

- 作为实时研发的来源表、维表或结果表。

- 作为数据仓库或数据集市，注册StarRocks为Dataphin计算源，可进行SQL研发及调度、数据质量检测、安全识别等数据研发及治理工作。

## 数据集成

在Dataphin中，支持创建StarRocks数据源，并且在离线集成任务中使用StarRocks数据源作为来源数据库或目标数据库。具体使用步骤如下：

### 创建StarRocks数据源

#### 基本信息

![创建StarRocks数据源-基本信息](../../assets/Dataphin/create_sr_datasource_1.png)

- **数据源名称**：必填。输入数据源的名称，只能包含中文、字母、数字、下划线（_）或中划线（-），长度不能超过64个字符。

- **数据源编码**：选填。配置数据源编码后，可在Flink SQL任务中通过`数据源编码.table`或`数据源编码.schema.table`的格式引用数据源中的表。如果需要根据所处环境自动访问对应环境的数据源，请通过`${数据源编码}.table`或`${数据源编码}.schema.table`的格式访问。注意：目前仅支持MySQL、Hologres、MaxCompute数据源。

- **支持应用场景**：支持StarRocks数据源的应用场景。

- **数据源描述**：选填。输入对数据源简单的描述，长度不得超过128个字符。

- **数据源配置**：必填。如果业务数据源区分生产数据源和开发数据源，则选择**“生产+开发”数据源**。如果业务数据源不区分生产数据源和开发数据源，则选择**“生产”数据源**。

- **标签**：选填。可选择标签给数据源进行分类打标。

#### 配置信息

![创建StarRocks数据源-配置信息](../../assets/Dataphin/create_sr_datasource_2.png)

- **JDBC URL**：必填。格式为`jdbc:mysql://<host>:<port>/<dbname>`，其中`host`为StarRocks集群的FE（Front End）主机IP地址，`port`为FE的查询端口，`dbname`为数据库名称。

- **Load URL**：必填。格式为`fe_ip:http_port;fe_ip:http_port`，其中`fe_ip`为FE的Host，`http_port`为FE的HTTP端口。

- **用户名**：必填。数据库的用户名。

- **密码**：必填。数据库的密码。

#### 高级设置

![创建StarRocks数据源-高级设置](../../assets/Dataphin/create_sr_datasource_3.png)

- **connectTimeout**：数据库的`connectTimeout`时长（单位ms），默认900000毫秒（15分钟）。

- **socketTimeout**：数据库的`socketTimeout`时长（单位ms），默认1800000毫秒（30分钟）。

### 从StarRocks数据源读取数据写入其他数据源

#### 在离线集成任务画布中拖入StarRocks输入组件

![从StarRocks读取-1](../../assets/Dataphin/read_from_sr_datasource_1.png)

#### 配置StarRocks输入组件配置

![从StarRocks读取-2](../../assets/Dataphin/read_from_sr_datasource_2.png)

- **步骤名称**：根据当前组件的使用场景及定位，输入合适的名称。

- **数据源**：可选Dataphin中创建的StarRocks数据源或是项目。需要配置人员具备同步读权限的数据源。如有不满足，可通过添加数据源或申请相关权限获取。

- **来源表信息**：根据实际场景需要，选择单张表或多张具有相同表结构的表，作为输入。

- **表**：下拉可以选择StarRocks数据源中的表。

- **切分键**：配合并发度配置使用。您可以将源数据表中某一列作为切分键，该字段类型必须是整型数字，建议使用主键或有索引的列作为切分键。

- **批量条数**：批量抽取数据的条数。

- **过滤信息**：过滤信息非必填项。

两种情况下会填写相关信息：

- 固定的某一部分数据。
- 带参数过滤，比如对于需要每天增量追加或全量覆盖获取数据的情况，往往会填入带有表内日期字段限制为Dataphin的系统时间参数，比如StarRocks库中的一张交易表，交易创建日期=`${bizdate}`。

- **输出字段**：针对所选的表信息，获取表的字段作为输出字段。可进行字段重命名、移除或再次添加、移动字段的顺序。一般情况下，重命名是为了下游的数据可读性或输出时候的字段方便映射；移除是因为从应用场景角度考虑不需要相关字段，因此在输入步骤及早对不需要的字段进行剔除；移动字段顺序是为了下游有多个输入数据进行合并或输出的时候，对名称不一致情况下可以采用同行映射的方式高效进行数据合并或映射输出。

### 选择目标数据源作为输出组件并配置

![选择目标数据源作为输出组件](../../assets/Dataphin/write_to_sr_datasource_1.png)

#### 选择目标数据源作为输出组件，并配置输出组件

![从StarRocks读取-3](../../assets/Dataphin/read_from_sr_datasource_3.png)

### 从其他数据源读取数据写入到StarRocks数据源

#### 在离线集成任务中配置输入组件，配置StarRocks输出组件作为下游

![写入到StarRocks-1](../../assets/Dataphin/write_to_sr_datasource_1.png)

#### 配置StarRocks输出组件

![写入到StarRocks-2](../../assets/Dataphin/write_to_sr_datasource_2.png)

- **步骤名称**：根据当前组件的使用场景及定位，输入合适的名称。

- **数据源**：可选Dataphin中创建的StarRocks数据源或是项目。需要配置人员具备同步写权限的数据源。如有不满足，可通过添加数据源或申请相关权限获取。

- **表**：下拉可选择StarRocks数据源中的表，作为数据写入的目标。

- **一键生成目标表**：如果在 StarRocks 数据源中尚未创建目标表，则可以自动获取上游读取的字段名称、类型和备注，并生成建表语句。单击该按钮即可一键生成目标表。

- **CSV导入列分隔符**：在使用 Stream Load 进行 CSV 格式导入时，可以配置 CSV 导入列分隔符，默认为`\t`。如果要使用默认值，请不要在此处明确指定。如果数据本身包含`\t`，则需要自定义使用其他字符作为分隔符。

- **CSV导入行分隔符**：在使用 Stream Load 进行 CSV 格式导入时，可以配置 CSV 导入行分隔符，默认为`\n`。如果要使用默认值，请不要在此处明确指定。如果数据本身包含`\n`，则需要自定义使用其他字符作为分隔符。

- **解析方案**：非必填项。用于指示数据输出前和输出完成后的一些特殊处理。准备语句将在数据写入 StarRocks 数据源之前执行，结束语句将在写入完成后执行。

- **字段映射**：根据上游的输入和目标表的字段，可以手动选择字段映射或批量根据同行或同名映射。

## 实时研发

### 简介

广泛应用于企业实时计算场景中。它可以用于实时业务监控和分析、实时用户行为分析、广告实时竞价系统、实时风控和反欺诈，以及实时监控和预警等应用场景。通过实时分析和查询数据，企业可以快速了解业务情况、优化决策、提供更好的服务和保护企业利益。

### StarRocks 连接器

Flink 连接器内部的结果表是通过缓存并批量通过 Stream Load 导入实现，源表是通过批量读取数据实现。StarRocks 连接器支持的信息如下。

| **类别**                   | **详情**         |
| -------------------------- | ---------------- |
| 支持类型                   | 来源表、维表、结果表 |
| 运行模式                   | 流模式和批模式   |
| 数据格式                   | JSON 和 CSV      |
| 特有监控指标               | 暂无             |
| API 种类                  | DataStream 和 SQL |
| 是否支持更新或删除结果表数据  | 是               |

### 如何使用

Dataphin 支持将 StarRocks 数据源作为实时计算的读写目标端，并支持创建 StarRocks 元表并用于实时计算任务。以下是操作步骤示例：

#### 创建 StarRocks 元表

1. 进入 **Dataphin** > **研发** > **开发** > **表管理**。

2. 单击**新建**，选择**实时计算表**。

   ![创建 StarRocks 元表 - 1](../../assets/Dataphin/create_sr_metatable_1.png)

   - **表类型**：选择**元表**。

   - **元表名称**：填写元表名称。

   - **数据源**：选择一个 StarRocks 类型的数据源。

   - **来源表**：选择一张物理表作为来源表。

   - **选择目录**：选择要新建表的目录。

   - **描述**：选填。输入原表的简单描述。

   ![创建 StarRocks 元表 - 2](../../assets/Dataphin/create_sr_metatable_2.png)

3. 创建好元表后，可以对元表进行编辑，包括修改数据源、来源表、元表字段、配置元表参数等。

   ![编辑 StarRocks 元表](../../assets/Dataphin/edit_sr_metatable_1.png)

4. 提交元表。

#### 创建 Flink SQL 任务实现将 Kafka 中的数据实时写入到 StarRocks 中

1. 进入 **Dataphin** > **研发** > **开发** > **计算任务**。

2. 单击**新建 Flink SQL 任务**。

   ![创建 Flink SQL 任务 - 1](../../assets/Dataphin/create_flink_task_1.png)

3. 编辑 Fink SQL 代码并进行预编译，这里使用了 Kafka 元表作为输入表，StarRocks 元表作为输出表。

   ![创建 Flink SQL 任务 - 2](../../assets/Dataphin/create_flink_task_2.png)

   ![创建 Flink SQL 任务 - 3](../../assets/Dataphin/create_flink_task_3.png)

4. 预编译成功后，可以对代码进行调试、提交。

5. 在开发环境进行测试，可以通过打印日志和写测试表两种方式进行，其中测试表可以在**元表** > **属性** > **调试测试配置**中进行设置。

   ![创建 Flink SQL 任务 - 4](../../assets/Dataphin/create_flink_task_4.png)

   ![创建 Flink SQL 任务 - 5](../../assets/Dataphin/create_flink_task_5.png)

6. 开发环境任务正常运行后，可以将任务及用到的元表一起发布到生产环境。

   ![创建 Flink SQL 任务 - 6](../../assets/Dataphin/create_flink_task_6.png)

7. 在生产环境启动任务，实现将 Kafka 中的数据实时写入到 StarRocks 中。可以通过查看运行分析中各指标的情况和日志了解任务运行情况，也可以为任务配置监控告警。

## 数据仓库或数据集市

### 前提条件

- StarRocks 版本为 3.0.6 及以上。

- 已安装 Dataphin，且 Dataphin 版本为 3.12 及以上。

- 统计信息采集已开启，StarRocks 安装后采集即默认开启。详情见[CBO 统计信息](https://docs.starrocks.io/zh-cn/latest/using_starrocks/Cost_based_optimizer)。

- 仅支持 StarRocks Internal Catalog，即 `default_catalog`，不支持 External Catalog。

### 连接配置说明

#### 元仓设置

Dataphin 可以基于元数据进行信息的呈现与展示，包括表使用信息、元数据变更等。可支持使用 StarRocks 进行元数据的加工计算。因此在开始使用前，需要对元数据计算引擎（元仓）进行初始化设置。设置的步骤如下：

1. 使用管理员身份用户登录 Dataphin 元仓租户。

2. 进入**管理中心** > **系统设置** > **元仓设置**。

   a. 单击**开始**。

   b. 选择 **StarRocks**。

   c. 进行参数配置，通过测试连接后，单击**下一步**。

   d. 完成元仓初始化。

   ![元数据计算引擎设置](../../assets/Dataphin/metadata_engine_settings_1.png)

   参数说明如下：

   - **JDBC URL**：JDBC 连接串，分为两部分填写，可参考 MySQL JDBC URL 格式 `https://dev.mysql.com/doc/connector-j/8.1/en/connector-j-reference-jdbc-url-format.html`。

     - 第一部分：格式为 `jdbc:mysql://<Host>:<Port>/`，其中 `Host` 为 StarRocks 集群的 FE 主机 IP 地址，`Port` 为 FE 的查询端口，默认为 9030。

     - 第二部分：格式为 `database?key1=value1&key2=value2`，其中 `database` 为 StarRocks 用于元数据计算的数据库的名称，为必填项。`?` 后的参数为选填项。

   - **Load URL**：The format is `fe_ip:http_port;fe_ip:http_port`, where `fe_ip` is the Host of the FE, and `http_port` is the HTTP port of the FE.

   - **Username**: The username for connecting to StarRocks.

     The user needs to have read and write permissions for the database given in the JDBC URL and needs to have access permissions for the following databases and tables:

     - All tables under Information Schema
       
     - _statistics_.column_statistics
       
     - _statistics_.table_statistic_v1

   - **Password**: The password for connecting to StarRocks.

   - **Meta Project**: The project name used by Dataphin for metadata processing, only for internal use by the Dataphin system, and it is recommended to use **dataphin_meta**.

#### Create a StarRocks project and start data development

Starting data development includes the following steps:

1. Computing settings.

2. Creating a StarRocks computing source.

3. Creating a Dataphin project.

4. Creating a StarRocks SQL task.

##### Step One: Computing Settings

Computing settings set the type of computing engine and the address of the cluster for the tenant. The detailed steps are as follows:

1. Log in to Dataphin as a system administrator or super administrator.

2. Go to **Management Center** > **System Settings** > **Computing Settings**.

3. Select **StarRocks** and click **Next**.

4. Fill in the **JDBC URL** and wait for the validation to pass. The format of the JDBC URL is `jdbc:mysql://<Host>:<Port>/`, where `Host` is the FE main host IP address of the StarRocks cluster, and `Port` is the query port of the FE, which is 9030 by default.

##### Step Two: Create StarRocks Computing Source

The computing source is a concept in Dataphin, the main purpose of which is to bind and register the project space in Dataphin with the storage and computing space (i.e., Database) in StarRocks. A computing source needs to be created for each project. The detailed steps are as follows:

1. Log in to Dataphin as a system administrator or super administrator.

2. Go to **Planning** > **Computing Sources**.

3. Click **Add Computing Source** in the top right corner to create a computing source.

   The detailed configuration information is as follows:

   **Basic Information**

   ![Create compute engine - 1](../../assets/Dataphin/create_compute_engine_1.png)

   - **Computing Source Type**: Select **StarRocks**.

   - **Computing Source Name**: It is recommended to use the same name as the project to be created. If it is used for development projects, add the **_dev** suffix.

   - **Computing Source Description**: Optional. Fill in the description information of the computing source.

   **Configuration Information**

   ![Create compute engine - 2](../../assets/Dataphin/create_compute_engine_2.png)

   - **JDBC URL**: The format is `jdbc:mysql://<Host>:<Port>/`, where `Host` is the FE main host IP address of the StarRocks cluster, and `Port` is the query port of the FE, which is 9030 by default.

   - **Load URL**: The format is `fe_ip:http_port;fe_ip:http_port`, where `fe_ip` is the Host of the FE, and `http_port` is the HTTP port of the FE.

   - **Username**: The username for connecting to StarRocks.

   - **Password**: The password for connecting to StarRocks.

   - **Task Resource Group**: Different StarRocks resource groups can be specified for tasks of different priorities. When **Not Specified** is selected, the task resource group to be executed will be determined by the StarRocks engine. When **Specified Resource Group** is selected, tasks of different priorities will be specified to the set resource group by Dataphin. If a resource group is specified in the code of an SQL task or in the materialization configuration of a logical table, the setting in the code and the materialization configuration of the logical table will prevail, and the task resource group configuration of the computing source will be ignored when the task is executed.

   ![Create compute engine - 3](../../assets/Dataphin/create_compute_engine_3.png)

##### Step Three: Create Dataphin Project

After creating the computing source, it can be bound to a Dataphin project. The Dataphin project carries the management of project members, as well as the storage and computing space of StarRocks, and the management and operation of computing tasks.

The detailed steps for creating a Dataphin project are as follows:

1. Log in to Dataphin as a system administrator or super administrator.

2. Go to **Planning** > **Project**.

3. Click **Create Project** in the top right corner to create a project.

4. Fill in the basic information, and in the offline computing source, select the StarRocks computing source created in the previous step.

5. Click **Finish Creation**.

##### Step Four: Create StarRocks SQL Task

After creating the project, you can start creating StarRocks SQL tasks to perform DDL or DML operations on StarRocks.

The detailed steps are as follows:

1. Go to **Development** > **Development**.

2. Click **+** in the top right corner to create a StarRocks SQL task.

   ![Create Dataphin project - 1](../../assets/Dataphin/configure_dataphin_project_1.png)

3. Fill in the name and scheduling type, and complete the creation of the SQL task.

4. Enter SQL in the editor to start DDL and DML operations on StarRock.

   ![Create Dataphin project - 2](../../assets/Dataphin/configure_dataphin_project_2.png)