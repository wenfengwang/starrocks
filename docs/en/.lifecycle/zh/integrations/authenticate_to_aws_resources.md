---
displayed_sidebar: English
---

# 对 AWS 资源进行身份验证

StarRocks 支持使用三种身份验证方法与 AWS 资源集成：基于实例配置文件的身份验证、基于假定角色的身份验证和基于 IAM 用户的身份验证。本主题介绍如何使用这些身份验证方法配置 AWS 凭证。

## 身份验证方法

### 基于实例配置文件的身份验证

基于实例配置文件的身份验证方法允许您的 StarRocks 集群继承运行集群的 EC2 实例的实例配置文件中指定的权限。理论上，任何可以登录集群的集群用户都可以根据您配置的 AWS IAM 策略对您的 AWS 资源执行允许的操作。此用例的典型场景是您不需要在集群中的多个集群用户之间进行任何 AWS 资源访问控制。这种身份验证方法意味着同一集群内不需要隔离。

然而，这种身份验证方法仍然可以被视为一种集群级别的安全访问控制解决方案，因为谁可以登录到集群由集群管理员控制。

### 基于假定角色的身份验证

与基于实例配置文件的身份验证不同，基于假定角色的身份验证方法支持假定 AWS IAM 角色以获取对您的 AWS 资源的访问权限。有关更多信息，请参阅[假定一个角色](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)。

<!--具体来说，您可以将多个目录分别绑定到特定的假定角色，这样这些目录就可以连接到特定的 AWS 资源（例如，特定的 S3 存储桶）。这意味着您可以通过更改当前 SQL 会话的目录来访问不同的数据源。此外，如果集群管理员向不同用户授予不同目录的权限，这将实现一种访问控制解决方案，就像允许同一集群中的不同用户访问不同数据源一样。-->

### 基于 IAM 用户的身份验证

基于 IAM 用户的身份验证方法支持使用 IAM 用户凭证来访问您的 AWS 资源。有关更多信息，请参阅 [IAM 用户](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)。

<!--您可以将多个目录分别绑定到特定 IAM 用户的凭证（访问密钥和秘密密钥），以便同一集群中的不同用户可以访问不同的数据源。-->

## 准备工作

首先，找到与运行 StarRocks 集群的 EC2 实例关联的 IAM 角色（以下在本主题中称为 EC2 实例角色），并获取该角色的 ARN。您将需要 EC2 实例角色来进行基于实例配置文件的身份验证，并且需要 EC2 实例角色及其 ARN 来进行基于假定角色的身份验证。

下一步，根据您想要访问的 AWS 资源类型以及在 StarRocks 中的具体操作场景创建 IAM 策略。AWS IAM 中的策略声明了对特定 AWS 资源的一组权限。创建策略后，您需要将其附加到 IAM 角色或用户。因此，IAM 角色或用户被赋予了策略中声明的权限，以访问指定的 AWS 资源。

> **注意**
> 要进行这些准备工作，您必须有权限登录到 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home) 并编辑 IAM 用户和角色。

对于您需要访问特定 AWS 资源的 IAM 策略，请参阅以下部分：

- [从 AWS S3 批量加载数据](../reference/aws_iam_policies.md#batch-load-data-from-aws-s3)
- [读/写 AWS S3](../reference/aws_iam_policies.md#readwrite-aws-s3)
- [与 AWS Glue 集成](../reference/aws_iam_policies.md#integrate-with-aws-glue)

### 准备基于实例配置文件的身份验证

将用于访问所需 AWS 资源的 [IAM 策略](../reference/aws_iam_policies.md) 附加到 EC2 实例角色。

### 准备基于假定角色的身份验证

#### 创建 IAM 角色并附加策略

根据您要访问的 AWS 资源，创建一个或多个 IAM 角色。参见[创建 IAM 角色](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)。然后，将用于访问所需 AWS 资源的 [IAM 策略](../reference/aws_iam_policies.md) 附加到您创建的 IAM 角色。

例如，您希望 StarRocks 集群能够访问 AWS S3 和 AWS Glue。在这种情况下，您可以选择创建一个 IAM 角色（例如 `s3_assumed_role`），并将访问 AWS S3 和访问 AWS Glue 的策略都附加到该角色。或者，您也可以选择创建两个不同的 IAM 角色（例如 `s3_assumed_role` 和 `glue_assumed_role`），并分别将这些策略附加到两个不同的角色（即将访问 AWS S3 的策略附加到 `s3_assumed_role`，将访问 AWS Glue 的策略附加到 `glue_assumed_role`）。

您创建的 IAM 角色将被 StarRocks 集群的 EC2 实例角色承担，以访问指定的 AWS 资源。

本节假设您只创建了一个假定角色 `s3_assumed_role`，并且已将访问 AWS S3 和 AWS Glue 的策略都添加到该角色。

#### 配置信任关系

按以下方式配置您的假定角色：

1. 登录到 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)。
2. 在左侧导航面板中，选择 **权限管理** > **角色**。
3. 找到假定角色（`s3_assumed_role`）并点击其名称。
4. 在角色的详细信息页面上，点击 **信任关系** 选项卡，在 **信任关系** 选项卡上点击 **编辑信任策略**。
5. 在 **编辑信任策略** 页面上，删除现有的 JSON 策略文档，并粘贴以下 IAM 策略，您必须将 `<cluster_EC2_iam_role_ARN>` 替换为您之前获取的 EC2 实例角色的 ARN。然后，点击 **更新策略**。

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
   }
   ```

如果您为访问不同的 AWS 资源创建了不同的假定角色，您需要重复上述步骤来配置您的其他假定角色。例如，您已创建了 `s3_assumed_role` 和 `glue_assumed_role` 来分别访问 AWS S3 和 AWS Glue。在这种情况下，您需要重复上述步骤来配置 `glue_assumed_role`。

按以下方式配置您的 EC2 实例角色：

1. 登录到 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)。
2. 在左侧导航面板中，选择 **权限管理** > **角色**。
3. 找到 EC2 实例角色并点击其名称。
4. 在角色的详细信息页面的 **权限策略** 部分，点击 **添加权限** 并选择 **创建内联策略**。
5. 在 **指定权限** 步骤中，点击 **JSON** 选项卡，删除现有的 JSON 策略文档，并粘贴以下 IAM 策略，您必须将 `<s3_assumed_role_ARN>` 替换为假定角色 `s3_assumed_role` 的 ARN。然后，点击 **审查策略**。

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
   }
   ```
   如果您为访问不同的 AWS 资源创建了不同的假定角色，您需要在上述 IAM 策略的 **Resource** 元素中填写所有这些假定角色的 ARN，并用逗号（,）分隔。例如，您已创建了 `s3_assumed_role` 和 `glue_assumed_role` 来分别访问 AWS S3 和 AWS Glue。在这种情况下，您需要使用以下格式在 **Resource** 元素中填写 `s3_assumed_role` 的 ARN 和 `glue_assumed_role` 的 ARN：`"<s3_assumed_role_ARN>","<glue_assumed_role_ARN>"`。

6. 在 **审查策略** 步骤中，输入策略名称并点击 **创建策略**。

### 准备基于 IAM 用户的身份验证

创建一个 IAM 用户。参见[在您的 AWS 账户中创建 IAM 用户](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)。

然后，将用于访问所需 AWS 资源的 [IAM 策略](../reference/aws_iam_policies.md) 附加到您创建的 IAM 用户。

## 身份验证方法比较

下图提供了 StarRocks 中基于实例配置文件的身份验证、基于假定角色的身份验证和基于 IAM 用户的身份验证之间机制差异的高级说明。

![身份验证方法比较](../assets/authenticate_s3_credential_methods.png)

## 与 AWS 资源建立连接

### 访问 AWS S3 的身份验证参数

在 StarRocks 需要与 AWS S3 集成的各种场景中，例如，当您创建外部目录或文件外部表时，或者当您从 AWS S3 中提取、备份或恢复数据时，请按以下方式配置用于访问 AWS S3 的身份验证参数：

- 对于基于实例配置文件的身份验证，请将 `aws.s3.use_instance_profile` 设置为 `true`。
- 对于基于假定角色的身份验证，请将 `aws.s3.use_instance_profile` 设置为 `true` 并将 `aws.s3.iam_role_arn` 配置为您用于访问 AWS S3 的假定角色的 ARN（例如，您创建的假定角色 `s3_assumed_role` 的 ARN）。
- 对于基于 IAM 用户的身份验证，请将 `aws.s3.use_instance_profile` 设置为 `false` 并将 `aws.s3.access_key` 和 `aws.s3.secret_key` 配置为您的 AWS IAM 用户的访问密钥和秘密密钥。

以下表格描述了参数。

| 参数 | 必填 | 说明 |
```
|---|---|---|
|aws.s3.use_instance_profile|是|指定是否启用基于实例配置文件的认证方法和基于假设角色的认证方法。有效值：`true` 和 `false`。默认值：`false`。|
|aws.s3.iam_role_arn|否|拥有对您的 AWS S3 存储桶权限的 IAM 角色的 ARN。如果您使用基于假设角色的认证方法来访问 AWS S3，则必须指定此参数。|
|aws.s3.access_key|否|您的 IAM 用户的访问密钥。如果您使用基于 IAM 用户的认证方法来访问 AWS S3，则必须指定此参数。|
|aws.s3.secret_key|否|您的 IAM 用户的密钥。如果您使用基于 IAM 用户的认证方法来访问 AWS S3，则必须指定此参数。|

### 访问 AWS Glue 的认证参数

在 StarRocks 需要与 AWS Glue 集成的各种场景中，例如创建外部目录时，请按以下方式配置访问 AWS Glue 的认证参数：

- 对于基于实例配置文件的认证，请将 `aws.glue.use_instance_profile` 设置为 `true`。
- 对于基于假设角色的认证，请将 `aws.glue.use_instance_profile` 设置为 `true` 并配置 `aws.glue.iam_role_arn` 为您用于访问 AWS Glue 的假设角色的 ARN（例如，您之前创建的假设角色 `glue_assumed_role` 的 ARN）。
- 对于基于 IAM 用户的认证，请将 `aws.glue.use_instance_profile` 设置为 `false` 并配置 `aws.glue.access_key` 和 `aws.glue.secret_key` 为您的 AWS IAM 用户的访问密钥和密钥。

以下表格描述了这些参数。

|参数|必填|描述|
|---|---|---|
|aws.glue.use_instance_profile|是|指定是否启用基于实例配置文件的认证方法和基于假设角色的认证。有效值：`true` 和 `false`。默认值：`false`。|
|aws.glue.iam_role_arn|否|拥有对您的 AWS Glue 数据目录权限的 IAM 角色的 ARN。如果您使用基于假设角色的认证方法来访问 AWS Glue，则必须指定此参数。|
|aws.glue.access_key|否|您的 AWS IAM 用户的访问密钥。如果您使用基于 IAM 用户的认证方法来访问 AWS Glue，则必须指定此参数。|
|aws.glue.secret_key|否|您的 AWS IAM 用户的密钥。如果您使用基于 IAM 用户的认证方法来访问 AWS Glue，则必须指定此参数。|

## 集成示例

### 外部目录

在 StarRocks 集群中创建外部目录意味着构建与目标数据湖系统的集成，该系统由两个关键组件组成：

- 类似 AWS S3 的文件存储用于存储表文件
- 元存储（如 Hive Metastore 或 AWS Glue）用于存储表文件的元数据和位置

StarRocks 支持以下类型的目录：

- [Hive 目录](../data_source/catalog/hive_catalog.md)
- [Iceberg 目录](../data_source/catalog/iceberg_catalog.md)
- [Hudi 目录](../data_source/catalog/hudi_catalog.md)
- [Delta Lake 目录](../data_source/catalog/deltalake_catalog.md)

以下示例创建一个名为 `hive_catalog_hms` 或 `hive_catalog_glue` 的 Hive 目录，具体取决于您使用的元存储类型，以便从您的 Hive 集群查询数据。有关详细语法和参数，请参见 [Hive 目录](../data_source/catalog/hive_catalog.md)。

#### 基于实例配置文件的认证

- 如果您在 Hive 集群中使用 Hive Metastore，请运行如下命令：

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

- 如果您在 Amazon EMR Hive 集群中使用 AWS Glue，请运行如下命令：

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

#### 基于假设角色的认证

- 如果您在 Hive 集群中使用 Hive Metastore，请运行如下命令：

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

- 如果您在 Amazon EMR Hive 集群中使用 AWS Glue，请运行如下命令：

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

#### 基于 IAM 用户的认证

- 如果您在 Hive 集群中使用 Hive Metastore，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
  );
  ```

- 如果您在 Amazon EMR Hive 集群中使用 AWS Glue，请运行如下命令：

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

文件外部表必须在名为 `default_catalog` 的内部目录中创建。

以下示例在现有数据库 `test_s3_db` 上创建一个名为 `file_table` 的文件外部表。有关详细语法和参数，请参阅 [文件外部表](../data_source/file_external_table.md)。

#### 基于实例配置文件的认证

运行如下命令：

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

#### 基于假设角色的认证

运行如下命令：

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

#### 基于 IAM 用户的认证

运行如下命令：

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

### 数据摄取

您可以使用 LOAD LABEL 从 AWS S3 加载数据。

以下示例将数据从存储在 `s3a://test-bucket/test_brokerload_ingestion` 路径中的所有 Parquet 数据文件加载到现有数据库 `test_s3_db` 中的 `test_ingestion_2` 表。有关详细语法和参数，请参见 [BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 基于实例配置文件的认证

运行如下命令：

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

#### 基于假设角色的认证

运行如下命令：

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

#### 基于 IAM 用户的认证

运行如下命令：

```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(
    DATA INFILE("s3a://test-bucket/test_brokerload_ingestion/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-1"
)
PROPERTIES
(
    "timeout" = "1200"
);
```
```SQL
LOAD LABEL test_s3_db.test_credential_instanceprofile_7
(
    DATA INFILE("s3a://test-bucket/test_brokerload_ingestion/*")
    INTO TABLE test_ingestion_2
    FORMAT AS "parquet"
)
WITH BROKER
(
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "us-west-1"
)
PROPERTIES
(
    "timeout" = "1200"
);
```