---
displayed_sidebar: "Chinese"
---

# 验证 AWS 资源

StarRocks 支持使用三种验证方法与 AWS 资源集成功能：基于实例配置文件的验证、基于假定角色的验证和基于 IAM 用户的验证。本主题描述了如何通过使用这些验证方法配置 AWS 凭据。

## 验证方法

### 基于实例配置文件的验证

基于实例配置文件的验证方法允许您的 StarRocks 集群继承运行集群的 EC2 实例的实例配置文件中指定的权限。从理论上讲，可以登录到集群的任何集群用户都可以根据您配置的 AWS IAM 策略对您的 AWS 资源执行允许的操作。这种用例的典型情况是，您不需要在集群中的多个集群用户之间进行任何 AWS 资源访问控制。这种验证方法意味着在同一集群内不需要隔离。

然而，这种验证方法仍然可以被视为集群级别的安全访问控制解决方案，因为可以登陆到集群的任何人都受到集群管理员的控制。

### 基于假定角色的验证

与基于实例配置文件的验证不同，基于假定角色的验证方法支持假定 AWS IAM 角色以访问您的 AWS 资源。有关更多信息，请参见[假定角色](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-sharing-logs-assume-role.html)。

<!--具体来说，您可以将多个目录绑定到特定的假定角色，以便这些目录可以连接到特定的 AWS 资源（例如，特定的 S3 存储桶）。这意味着您可以通过更改当前 SQL 会话的目录来访问不同的数据源。此外，如果集群管理员为不同的目录授予不同的用户权限，这将实现一个访问控制解决方案，就像允许同一集群内的不同用户访问不同数据源一样。-->

### 基于 IAM 用户的验证

基于 IAM 用户的验证方法支持使用 IAM 用户凭据访问您的 AWS 资源。有关更多信息，请参见[IAM 用户](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html)。

<!--您可以将多个目录分别绑定到特定 IAM 用户的凭证（访问密钥和安全密钥），以便相同集群内的不同用户可以访问不同的数据源。-->

## 准备工作

首先，找到您的 StarRocks 集群所在的 EC2 实例关联的 IAM 角色（本主题后文称该角色为 EC2 实例角色），并获取角色的 ARN。您需要 EC2 实例角色来进行基于实例配置文件的验证，并需要 EC2 实例角色和其 ARN 来进行基于假定角色的验证。

接下来，基于想要访问的 AWS 资源类型以及 StarRocks 中的特定操作场景创建一个 IAM 策略。AWS IAM 中的策略声明了对特定 AWS 资源的一组权限。创建策略后，您需要将其附加到一个 IAM 角色或用户。因此，IAM 角色或用户被分配在策略中声明的权限，以访问指定的 AWS 资源。

> **注意**
>
> 要进行这些准备工作，您必须具有登录 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home) 以及编辑 IAM 用户和角色的权限。

有关您需要访问的 AWS 资源的 IAM 策略，请参见以下各节：

- [从 AWS S3 批量加载数据](../reference/aws_iam_policies.md#batch-load-data-from-aws-s3)
- [读/写 AWS S3](../reference/aws_iam_policies.md#readwrite-aws-s3)
- [与 AWS Glue 集成](../reference/aws_iam_policies.md#integrate-with-aws-glue)

### 基于实例配置文件的验证的准备工作

将用于访问所需 AWS 资源的 [IAM 策略](../reference/aws_iam_policies.md) 附加到 EC2 实例角色。

### 基于假定角色的验证的准备工作

#### 创建 IAM 角色并将策略附加到它们

根据您要访问的 AWS 资源数量创建一个或多个 IAM 角色。请参见[创建 IAM 角色](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create.html)。然后，将用于访问所需 AWS 资源的 [IAM 策略](../reference/aws_iam_policies.md) 附加到您创建的 IAM 角色。

例如，您希望您的 StarRocks 集群访问 AWS S3 和 AWS Glue。在这种情况下，您可以选择创建一个 IAM 角色（例如 `s3_assumed_role`），并将访问 AWS S3 的策略和访问 AWS Glue 的策略都附加到该角色。或者，您可以选择创建两个不同的 IAM 角色（例如 `s3_assumed_role` 和 `glue_assumed_role`），并分别将这些策略附加到两个不同的角色上（也就是，将用于访问 AWS S3 的策略附加到 `s3_assumed_role`，将用于访问 AWS Glue 的策略附加到 `glue_assumed_role`）。

您创建的 IAM 角色将由 StarRocks 集群的 EC2 实例角色假定，以访问指定的 AWS 资源。

本节假设您只创建了一个假定角色 `s3_assumed_role`，并已将用于访问 AWS S3 的策略和用于访问 AWS Glue 的策略都加入该角色。

#### 配置信任关系

配置您的假定角色如下：

1. 登录 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)。
2. 在左侧导航窗格中，选择 **访问管理** > **角色**。
3. 找到假定角色（`s3_assumed_role`）并点击它的名称。
4. 在角色详情页面上，点击 **信任关系** 选项卡，然后在 **信任关系** 选项卡上点击 **编辑信任策略**。
5. 在 **编辑信任策略** 页面上，删除现有的 JSON 策略文档，并粘贴以下 IAM 策略，其中您必须用 EC2 实例角色的 ARN 替换 `<cluster_EC2_iam_role_ARN>`。然后，点击 **更新策略**。

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

如果您为访问不同的 AWS 资源创建了不同的假定角色，那么您需要重复上述步骤来配置您的其他假定角色。例如，您已经为访问 AWS S3 和 AWS Glue 分别创建了 `s3_assumed_role` 和 `glue_assumed_role`。在这种情况下，您需要重复上述步骤来配置 `glue_assumed_role`。

配置您的 EC2 实例角色如下：

1. 登录 [AWS IAM 控制台](https://us-east-1.console.aws.amazon.com/iamv2/home#/home)。
2. 在左侧导航窗格中，选择 **访问管理** > **角色**。
3. 找到 EC2 实例角色并点击它的名称。
4. 在角色详情页面的 **权限策略** 部分，点击 **添加权限** 并选择 **创建嵌入式策略**。
5. 在 **指定权限** 步骤中，点击 **JSON** 选项卡，删除现有的 JSON 策略文档，并粘贴以下 IAM 策略，其中您必须用 `s3_assumed_role` 的 ARN 替换 `<s3_assumed_role_ARN>`。然后，点击 **审核策略**。

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

   如果您为访问不同的 AWS 资源创建了不同的假定角色，那么您需要在上述 IAM 策略的 **资源** 元素中填写所有这些假定角色的 ARN，并用逗号（，）分隔。例如，您已经为访问 AWS S3 和 AWS Glue 分别创建了`s3_assumed_role`和`glue_assumed_role`。在这种情况下，您需要在上述 IAM 策略的 **资源** 元素中用如下格式填写`"<s3_assumed_role_ARN>","<glue_assumed_role_ARN>"`。

6. 在 **审核策略** 步骤中，输入策略名称并点击 **创建策略**。

### 基于 IAM 用户的验证的准备工作

创建一个 IAM 用户。请参见[在您的 AWS 账户中创建 IAM 用户](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)。

然后，将用于访问所需 AWS 资源的 [IAM 策略](../reference/aws_iam_policies.md) 附加到您创建的 IAM 用户。

## 验证方法之间的比较

下图提供了对 StarRocks 中基于实例配置文件的验证、基于假定角色的验证和基于 IAM 用户的验证机制之间差异的高级解释。

![验证方法之间的比较](../assets/authenticate_s3_credential_methods.png)

## 与 AWS 资源建立连接

### 访问 AWS S3 的验证参数

在各種需要StarRocks與AWS S3集成的場景中，例如當您創建外部目錄或文件外部表時，或當您從AWS S3進行數據載入、備份或還原時，配置訪問AWS S3的身份驗證參數如下：

- 對於基於實例配置文件的身份驗證，將`aws.s3.use_instance_profile`設置為`true`。
- 對於基於假定角色的身份驗證，將`aws.s3.use_instance_profile`設置為`true`，並配置`aws.s3.iam_role_arn`作為您用於訪問AWS S3的假定角色的ARN（例如，您在上方創建的假定角色`s3_assumed_role`的ARN）。
- 對於基於IAM用戶的身份驗證，將`aws.s3.use_instance_profile`設置為`false`，並配置`aws.s3.access_key`和`aws.s3.secret_key`作為您的AWS IAM用戶的訪問密鑰和秘密密鑰。

下表描述了這些參數。

| 參數                        | 必填     | 描述                                                      |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是       | 指定是否啟用基於實例配置文件的身份驗證方法和基於假定角色的身份驗證方法。有效值：`true`和`false`。默認值：`false`。 |
| aws.s3.iam_role_arn         | 否       | 具有對AWS S3存儲桶特權的IAM角色的ARN。如果您使用基於假定角色的身份驗證方法訪問AWS S3，則必須指定此參數。  |
| aws.s3.access_key           | 否       | 您的IAM用戶的訪問金鑰。如果您使用基於IAM用戶的身份驗證方法訪問AWS S3，則必須指定此參數。 |
| aws.s3.secret_key           | 否       | 您的IAM用戶的秘密金鑰。如果您使用基於IAM用戶的身份驗證方法訪問AWS S3，則必須指定此參數。 |

### 訪問AWS Glue的身份驗證參數

在各種需要StarRocks與AWS Glue集成的場景中，例如當您創建外部目錄時，配置訪問AWS Glue的身份驗證參數如下：

- 對於基於實例配置文件的身份驗證，將`aws.glue.use_instance_profile`設置為`true`。
- 對於基於假定角色的身份驗證，將`aws.glue.use_instance_profile`設置為`true`，並配置`aws.glue.iam_role_arn`作為您用於訪問AWS Glue的假定角色的ARN（例如，您在上方創建的假定角色`glue_assumed_role`的ARN）。
- 對於基於IAM用戶的身份驗證，將`aws.glue.use_instance_profile`設置為`false`，並配置`aws.glue.access_key`和`aws.glue.secret_key`作為您的AWS IAM用戶的訪問金鑰和秘密金鑰。

下表描述了這些參數。

| 參數                        | 必填     | 描述                                                       |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| aws.glue.use_instance_profile | 是       | 指定是否啟用基於實例配置文件的身份驗證方法和基於假定角色的身份驗證。有效值：`true`和`false`。默認值：`false`。  |
| aws.glue.iam_role_arn         | 否       | 具有對AWS Glue數據目錄特權的IAM角色的ARN。如果您使用基於假定角色的身份驗證方法訪問AWS Glue，則必須指定此參數。  |
| aws.glue.access_key           | 否       | 您的AWS IAM用戶的訪問金鑰。如果您使用基於IAM用戶的身份驗證方法訪問AWS Glue，則必須指定此參數。  |
| aws.glue.secret_key           | 否       | 您的AWS IAM用戶的秘密金鑰。如果您使用基於IAM用戶的身份驗證方法訪問AWS Glue，則必須指定此參數。  |

## 集成示例

### 外部目錄

在StarRocks集群中創建外部目錄意味著與目標數據湖系統集成，該系統由兩個關鍵組件組成：

- 用於存儲表文件的文件存儲，如AWS S3
- 用於存儲表文件的元存儲，如Hive元存儲或AWS Glue

StarRocks支持以下類型的目錄：

- [Hive目錄](../data_source/catalog/hive_catalog.md)
- [Iceberg目錄](../data_source/catalog/iceberg_catalog.md)
- [Hudi目錄](../data_source/catalog/hudi_catalog.md)
- [Delta Lake目錄](../data_source/catalog/deltalake_catalog.md)

以下示例創建一個名為`hive_catalog_hms`或`hive_catalog_glue`的Hive目錄，具體取決於您使用的元存儲類型，以從Hive集群查詢數據。有關詳細的語法和參數，請參閱[Hive目錄](../data_source/catalog/hive_catalog.md)。

#### 基於實例配置文件的身份驗證

- 如果您在Hive集群中使用Hive元存儲，請運行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，請運行如下命令：

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

#### 基於假定角色的身份驗證

- 如果您在Hive集群中使用Hive元存儲，請運行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/s3_assumed_role",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，請運行如下命令：

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

#### 基於IAM用戶的身份驗證

- 如果您在Hive集群中使用Hive元存儲，請運行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，請運行如下命令：

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

文件外部表必須在名為`default_catalog`的內部目錄中創建。

以下示例在現有名為`test_s3_db`的數據庫上創建名為`file_table`的文件外部表。有關詳細的語法和參數，請參閱[文件外部表](../data_source/file_external_table.md)。

#### 基於實例配置文件的身份驗證

運行如下命令：

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
``` 
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

#### 假定的基于角色的身份验证

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

您可以使用LOAD LABEL从AWS S3加载数据。

以下示例将存储在`s3a://test-bucket/test_brokerload_ingestion`路径下所有Parquet数据文件的数据加载到名为`test_s3_db`的现有数据库中的名为`test_ingestion_2`的表中。有关详细的语法和参数，请参阅[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)。

#### 基于实例配置文件的身份验证

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


#### 假定的基于角色的身份验证

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

#### 基于IAM用户的身份验证

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