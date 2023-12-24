---
displayed_sidebar: English
---

# 身份验证 AWS 资源

StarRocks 支持使用三种身份验证方法与 AWS 资源集成：基于实例配置文件的身份验证、假定角色的身份验证和 IAM 用户的身份验证。本主题描述如何使用这些身份验证方法配置 AWS 凭证。

## 身份验证方法

### 基于实例配置文件的身份验证

基于实例配置文件的身份验证方法允许您的 StarRocks 集群继承运行集群的 EC2 实例的实例配置文件中指定的权限。从理论上讲，任何可以登录集群的集群用户都可以根据您配置的 AWS IAM 策略对您的 AWS 资源执行允许的操作。此使用案例的典型场景是，您不需要在集群中的多个集群用户之间进行任何 AWS 资源访问控制。此身份验证方法意味着不需要在同一群集中进行隔离。

但是，这种身份验证方法仍然可以看作是集群级别的安全访问控制解决方案，因为谁可以登录集群，谁就由集群管理员控制。

### 假定角色的身份验证

与基于实例配置文件的身份验证不同，假定角色的身份验证方法支持假定 AWS IAM 角色以访问您的 AWS 资源。有关更多信息，请参阅 [假定角色](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)。

### IAM 用户的身份验证

IAM 用户的身份验证方法支持使用 IAM 用户凭证来访问您的 AWS 资源。有关更多信息，请参阅 [IAM 用户](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)。

## 准备工作

首先，找到运行 StarRocks 集群的 EC2 实例关联的 IAM 角色（本文下文中该角色称为 EC2 实例角色），并获取该角色的 ARN。您将需要 EC2 实例角色进行基于实例配置文件的身份验证，并且需要 EC2 实例角色及其 ARN 进行假定角色的身份验证。

接下来，根据您要访问的 AWS 资源类型和 StarRocks 中的具体操作场景创建 IAM 策略。AWS IAM 中的策略声明对特定 AWS 资源的一组权限。创建策略后，您需要将其附加到 IAM 角色或用户。因此，将为 IAM 角色或用户分配策略中声明的权限，以访问指定的 AWS 资源。

> **注意**
>
> 要进行这些准备工作，您必须有权限登录 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home) 并编辑 IAM 用户和角色。

有关您需要访问特定 AWS 资源的 IAM 策略，请参阅以下部分：

- [从 AWS S3 批量加载数据](../reference/aws_iam_policies.md#batch-load-data-from-aws-s3)
- [读/写 AWS S3](../reference/aws_iam_policies.md#readwrite-aws-s3)
- [与 AWS Glue 集成](../reference/aws_iam_policies.md#integrate-with-aws-glue)

### 基于实例配置文件的身份验证准备

将用于访问所需 AWS 资源的 IAM 策略附加到 EC2 实例角色。请参阅 [IAM 策略](../reference/aws_iam_policies.md)。

### 假定角色的身份验证准备

#### 创建 IAM 角色并附加策略

根据您要访问的 AWS 资源，创建一个或多个 IAM 角色。请参阅 [创建 IAM 角色](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)。然后，将用于访问所需 AWS 资源的 IAM 策略附加到您创建的 IAM 角色。

例如，您希望 StarRocks 集群能够访问 AWS S3 和 AWS Glue。在这种情况下，您可以选择创建一个 IAM 角色（例如，`s3_assumed_role`），并将用于访问 AWS S3 的策略和用于访问 AWS Glue 的策略都附加到该角色。或者，您可以选择创建两个不同的 IAM 角色（例如，`s3_assumed_role` 和 `glue_assumed_role`），并将这些策略分别附加到两个不同的角色（即，将用于访问 AWS S3 的策略附加到 `s3_assumed_role`，将用于访问 AWS Glue 的策略附加到 `glue_assumed_role`）。

您创建的 IAM 角色将由 StarRocks 集群的 EC2 实例角色代入，以访问指定的 AWS 资源。

本部分假定您只创建了一个假定角色 `s3_assumed_role`，并已将用于访问 AWS S3 的策略和用于访问 AWS Glue 的策略添加到该角色中。

#### 配置信任关系

按如下方式配置假定角色：

1. 登录 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)。
2. 在左侧导航栏，选择 **访问管理** > **角色**。
3. 找到假定角色（`s3_assumed_role`）并单击其名称。
4. 在角色的详细信息页面上，单击 **信任关系** 选项卡，然后在 **信任关系** 选项卡上单击 **编辑信任策略**。
5. 在 **编辑信任策略** 页面上，删除现有的 JSON 策略文档，然后粘贴以下 IAM 策略，在其中您必须替换 `<cluster_EC2_iam_role_ARN>` 为您上面获取的 EC2 实例角色的 ARN。然后，单击 **更新策略**。

   ```JSON
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Principal": {
                   "AWS": "<cluster_EC2_iam_role_ARN>"
               },
               "Action": "sts:AssumeRole"
           }
       ]
   ```
如果您为访问不同的 AWS 资源创建了不同的假定角色，您需要重复上述步骤来配置其他假定角色。例如，您已分别创建 `s3_assumed_role` 和 `glue_assumed_role` 用于访问 AWS S3 和 AWS Glue。在这种情况下，您需要重复上述步骤来配置 `glue_assumed_role`。

按如下方式配置您的 EC2 实例角色：

1. 登录 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)。
2. 在左侧导航栏，选择 **访问管理** > **角色**。
3. 找到 EC2 实例角色，然后单击其名称。
4. 在角色详细信息页面的 **权限策略** 部分，单击 **添加权限**，然后选择 **创建内联策略**。
5. 在 **指定权限** 步骤中，单击 **JSON** 选项卡，删除现有 JSON 策略文档，然后粘贴以下 IAM 策略，在其中您必须替换 `<s3_assumed_role_ARN>` 为代入角色 `s3_assumed_role` 的 ARN。然后，单击 **查看策略**。

   ```JSON
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": ["sts:AssumeRole"],
               "Resource": [
                   "<s3_assumed_role_ARN>"
               ]
           }
       ]
   ```
如果您为访问不同的 AWS 资源创建了不同的假定角色，您需要在上述 IAM 策略的 **Resource** 元素中填充所有这些假定角色的 ARN，并用逗号（,）分隔它们。例如，您已分别创建 `s3_assumed_role` 和 `glue_assumed_role` 用于访问 AWS S3 和 AWS Glue。在这种情况下，您需要使用以下格式在 **Resource** 元素中填充 `s3_assumed_role` 和 `glue_assumed_role` 的 ARN："`<s3_assumed_role_ARN>","<glue_assumed_role_ARN>`。

6. 在 **查看策略** 步骤中，输入策略名称，然后单击 **创建策略**。

### IAM 用户的身份验证准备

创建一个 IAM 用户。请参阅 [在您的 AWS 账户中创建 IAM 用户](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)。

然后，将用于访问所需 AWS 资源的 IAM 策略附加到您创建的 IAM 用户。

## 身份验证方法的比较

下图提供了 StarRocks 中基于实例配置文件的身份验证、假定角色的身份验证和 IAM 用户的身份验证机制之间差异的高级解释。

![身份验证方法之间的比较](../assets/authenticate_s3_credential_methods.png)

## 与 AWS 资源建立连接

### 访问 AWS S3 的身份验证参数

在 StarRocks需要与 AWS S3 集成的各种场景下，例如创建外部目录或文件外部表，或从 AWS S3 中提取、备份或恢复数据时，请按如下方式配置访问 AWS S3 的身份验证参数：

- 对于基于实例配置文件的身份验证，请将 `aws.s3.use_instance_profile` 设置为 `true`。
- 对于假定角色的身份验证，请将 `aws.s3.use_instance_profile` 设置为 `true`，并配置 `aws.s3.iam_role_arn` 为您用于访问 AWS S3 的假定角色的 ARN（例如，您在上面创建的 `s3_assumed_role` 的 ARN）。
- 对于 IAM 用户的身份验证，请将 `aws.s3.use_instance_profile` 设置为 `false`，并配置 `aws.s3.access_key` 和 `aws.s3.secret_key` 为您的 AWS IAM 用户的访问密钥和私有密钥。

以下表格描述了参数。

| 参数                   | 必填 | 描述                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值： `true` 和 `false`。默认值： `false`。 |
| aws.s3.iam_role_arn         | 否       | 具有对您的 AWS S3 存储桶具有权限的 IAM 角色的 ARN。如果您使用基于假定角色的身份验证方法访问 AWS S3，则必须指定此参数。  |
| aws.s3.access_key           | 否       | 您的 IAM 用户的访问密钥。如果您使用基于IAM用户的身份验证方法访问AWS S3，则必须指定此参数。 |
| aws.s3.secret_key           | 否       | 您的IAM用户的私有密钥。如果您使用基于IAM用户的身份验证方法访问AWS S3，则必须指定此参数。 |

### 访问 AWS Glue 的身份验证参数

在StarRocks需要与AWS Glue集成的各种场景中，例如创建外部目录时，配置访问AWS Glue的身份验证参数如下：

- 对于基于实例配置文件的身份验证，请将 `aws.glue.use_instance_profile` 设置为 `true`。
- 对于基于假定角色的身份验证，请将 `aws.glue.use_instance_profile` 设置为 `true` 并配置 `aws.glue.iam_role_arn` 作为用于访问AWS Glue的假定角色的ARN（例如，您在上面创建的假定角色`glue_assumed_role`的ARN）。
- 对于基于IAM用户的身份验证，请将 `aws.glue.use_instance_profile` 设置为 `false` 并配置 `aws.glue.access_key` 和 `aws.glue.secret_key` 作为您的AWS IAM用户的访问密钥和私有密钥。

以下表格描述了参数。

| 参数                     | 必填 | 描述                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| aws.glue.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证。有效值： `true` 和 `false`。默认值： `false`。 |
| aws.glue.iam_role_arn         | 否       | 具有对您的AWS Glue数据目录具有权限的IAM角色的ARN。如果您使用基于假定角色的身份验证方法访问AWS Glue，则必须指定此参数。 |
| aws.glue.access_key           | 否       | 您的AWS IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法访问AWS Glue，则必须指定此参数。 |
| aws.glue.secret_key           | 否       | 您的AWS IAM用户的私有密钥。如果您使用基于IAM用户的身份验证方法访问AWS Glue，则必须指定此参数。 |

## 集成示例

### 外部目录

在StarRocks集群中创建外部目录意味着构建与目标数据湖系统的集成，目标数据湖系统由两个关键组件组成：

- 用于存储表文件的文件存储，例如AWS S3
- 用于存储表文件的元数据和位置的元存储，例如Hive元存储或AWS Glue

StarRocks支持以下类型的目录：

- [Hive目录](../data_source/catalog/hive_catalog.md)
- [Iceberg目录](../data_source/catalog/iceberg_catalog.md)
- [Hudi目录](../data_source/catalog/hudi_catalog.md)
- [Delta Lake目录](../data_source/catalog/deltalake_catalog.md)

以下示例创建一个名为`hive_catalog_hms`或`hive_catalog_glue`的Hive目录，具体取决于您使用的元存储类型，用于查询来自Hive集群的数据。有关详细的语法和参数，请参见[Hive目录](../data_source/catalog/hive_catalog.md)。

#### 基于实例配置文件的身份验证

- 如果您在Hive集群中使用Hive元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2"
  );
  ```

#### 假定的基于角色的身份验证

- 如果您在Hive集群中使用Hive元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/glue_assumed_role",
      "aws.glue.region" = "us-west-2"
  );
  ```

#### IAM 基于用户的身份验证

- 如果您在Hive集群中使用Hive元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "false",
      "aws.glue.access_key" = "<iam_user_access_key>",
      "aws.glue.secret_key" = "<iam_user_secret_key>",
      "aws.glue.region" = "us-west-2"
  );
  ```

### 文件外部表

必须在名为`default_catalog`的内部目录中创建文件外部表。

以下示例在名为`test_s3_db`的现有数据库上创建一个名为`file_table`的文件外部表。有关详细的语法和参数，请参阅[文件外部表](../data_source/file_external_table.md)。

#### 基于实例配置文件的身份验证

运行以下命令：

```SQL
CREATE EXTERNAL TABLE test_s3_db.file_table
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "s3://starrocks-test/",
    "format" = "ORC",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "us-west-2"
);
```

#### 假定的基于角色的身份验证

运行以下命令：

```SQL
CREATE EXTERNAL TABLE test_s3_db.file_table
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "s3://starrocks-test/",
    "format" = "ORC",
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
    "aws.s3.region" = "us-west-2"
);
```

#### IAM 基于用户的身份验证

运行以下命令：

```SQL
CREATE EXTERNAL TABLE test_s3_db.file_table
(
    id varchar(65500),
    attributes map<varchar(100), varchar(2000)>
) 
ENGINE=FILE
PROPERTIES
(
    "path" = "s3://starrocks-test/",
    "format" = "ORC",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-2"
);
```

### 摄入

您可以使用LOAD LABEL从AWS S3加载数据。

以下示例将存储在路径`s3a://test-bucket/test_brokerload_ingestion`中的所有Parquet数据文件中的数据加载到名为`test_ingestion_2`的现有数据库中的表中。有关详细的语法和参数，请参阅[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 基于实例配置文件的身份验证

运行以下命令：

```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(
    DATA INFILE("s3a://test-bucket/test_brokerload_ingestion/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "us-west-1"
)
PROPERTIES
(
    "timeout" = "1200"
);
```

#### 假定的基于角色的身份验证

运行以下命令：

```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(
    DATA INFILE("s3a://test-bucket/test_brokerload_ingestion/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
    "aws.s3.region" = "us-west-1"
)
PROPERTIES
(
    "timeout" = "1200"
);
```

#### IAM 基于用户的身份验证

运行以下命令：

```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(
    数据文件("s3a://test-bucket/test_brokerload_ingestion/*")
    到表 test_ingestion_2
    格式为 "parquet"
)
使用经纪人
(
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-1"
)
属性
(
    "timeout" = "1200"
);