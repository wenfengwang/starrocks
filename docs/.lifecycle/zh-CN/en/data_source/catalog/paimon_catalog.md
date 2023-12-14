---
displayed_sidebar: "Chinese"
---

# 派蒙目录

StarRocks支持从v3.1版本开始的Paimon目录。

Paimon目录是一种外部目录，它使你能够在不进行摄入的情况下从Apache Paimon中查询数据。

此外，你可以通过基于Paimon目录的[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)直接转换和加载Paimon的数据。

为了确保你的Paimon集群可以成功处理SQL工作负载，你的StarRocks集群需要与两个重要组件集成：

- 分布式文件系统 (HDFS) 或类似AWS S3、Microsoft Azure Storage、Google GCS或其他兼容S3的存储系统的对象存储
- 文件系统或Hive Metastore

## 使用注意事项

你只能使用Paimon目录查询数据。不能使用Paimon目录删除、清除或插入Paimon集群中的数据。

## 集成准备

在创建Paimon目录之前，请确保你的StarRocks集群可以与Paimon集群的存储系统和元数据存储集成。

### AWS IAM

如果你的Paimon集群使用AWS S3作为存储，选择适当的身份验证方法并进行必要的准备，以确保你的StarRocks集群能够访问相关的AWS云资源。

推荐以下身份验证方法：

- 实例配置文件 (推荐的)
- 假定角色
- IAM用户

在上述三种身份验证方法中，实例配置文件是最常用的。

更多信息，请参见[AWS IAM身份验证准备](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)。

### HDFS

如果你选择HDFS作为存储，配置你的StarRocks集群如下：

- （可选）设置用于访问你的HDFS集群和Hive Metastore的用户名。默认情况下，StarRocks使用FE和BE进程的用户名来访问你的HDFS集群和Hive Metastore。你也可以通过在每个FE的**fe/conf/hadoop_env.sh**文件和每个BE的**be/conf/hadoop_env.sh**文件的开头添加`export HADOOP_USER_NAME="<user_name>"`来设置用户名。在这些文件中设置用户名后，重新启动每个FE和BE以使参数设置生效。你可以为每个StarRocks集群设置一个用户名。
- 当你查询Paimon数据时，你的StarRocks集群的FE和BE使用HDFS客户端来访问你的HDFS集群。在大多数情况下，你不需要配置你的StarRocks集群来实现这一目的，StarRocks会使用默认配置启动HDFS客户端。在以下情况下，你只需要配置你的StarRocks集群：
  - 为你的HDFS集群启用高可用性 (HA)：将你的HDFS集群的**hdfs-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。
  - 为你的HDFS集群启用视图文件系统 (ViewFs)：将你的HDFS集群的**core-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。

> **注意**
>
> 如果发送查询时返回了主机未知的错误，你必须将你的HDFS集群节点的主机名和IP地址的映射添加到**/etc/hosts**路径。

### Kerberos身份验证

如果启用了Kerberos身份验证用于你的HDFS集群或Hive Metastore，配置你的StarRocks集群如下：

- 在每个FE和每个BE上运行`kinit -kt keytab_path principal`命令以获取来自密钥分发中心 (KDC) 的票证授予票(TGT)。运行此命令时，你必须具有访问你的HDFS集群和Hive Metastore的权限。注意，使用此命令访问KDC是有时间限制的，因此你需要使用cron定期运行此命令。
- 在每个FE的**$FE_HOME/conf/fe.conf**文件和每个BE的**$BE_HOME/conf/be.conf**文件中添加`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf`是**krb5.conf**文件的保存路径。你可以根据自己的需要修改路径。

## 创建Paimon目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "paimon",
    CatalogParams,
    StorageCredentialParams
)
```

### 参数

#### catalog_name

Paimon目录的名称。命名规范如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不得超过1023个字符。

#### comment

Paimon目录的描述。该参数是可选的。

#### type

你的数据源类型。将值设置为`paimon`。

#### CatalogParams

关于StarRocks如何访问你的Paimon集群元数据的一组参数。

以下表格描述了你需要在`CatalogParams`中配置的参数。

| 参数                      | 是否必需 | 描述                                                   |
| ------------------------- | -------- | ------------------------------------------------------ |
| paimon.catalog.type       | 是       | 用于你的Paimon集群的元数据存储类型。将该参数设置为`filesystem`或`hive`。 |
| paimon.catalog.warehouse  | 是       | 你的Paimon数据的仓库存储路径。                             |
| hive.metastore.uris       | 否       | 你的Hive Metastore的URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。如果为你的Hive Metastore启用了高可用性(HA)，你可以指定多个元数据存储URI，并用逗号(,)分隔，例如，`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

> **注意**
>
> 如果你使用Hive Metastore，你必须在查询Paimon数据之前，将你的Hive Metastore节点的主机名和IP地址的映射添加到**/etc/hosts**路径。否则，当你启动查询时，StarRocks可能无法访问你的Hive Metastore。

#### StorageCredentialParams

关于StarRocks如何与你的存储系统集成的一组参数。此参数集是可选的。

如果你使用HDFS作为存储，你不需要配置`StorageCredentialParams`。

如果你使用AWS S3、其他兼容S3的存储系统、Microsoft Azure Storage或Google GCS作为存储，你必须配置`StorageCredentialParams`。

##### AWS S3

如果选择AWS S3作为你的Paimon集群的存储，执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于假定角色的身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于IAM用户的身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下表格描述了你需要在`StorageCredentialParams`中配置的参数。

| 参数                      | 是否必需 | 描述                                                       |
| ------------------------- | -------- | ---------------------------------------------------------- |
| aws.s3.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值：`true`和`false`。默认值：`false`。 |
| aws.s3.iam_role_arn       | 否       | 对你的AWS S3存储桶具有权限的IAM角色的ARN。如果使用基于假定角色的身份验证方法访问AWS S3，你必须指定此参数。 |
| aws.s3.region             | 是       | 你的AWS S3存储桶所在的地区。示例：`us-west-1`。              |
| aws.s3.access_key         | 否       | 你的IAM用户的访问密钥。如果使用基于IAM用户的身份验证方法访问AWS S3，你必须指定此参数。 |
| aws.s3.secret_key         | 否       | 你的IAM用户的秘密密钥。如果使用基于IAM用户的身份验证方法访问AWS S3，你必须指定此参数。 |

有关如何选择用于访问AWS S3的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[用于访问AWS S3的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容S3的存储系统

如果选择兼容S3的存储系统（例如MinIO）作为Paimon集群的存储，需按照以下方式配置“StorageCredentialParams”以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了需要在“StorageCredentialParams”中配置的参数。

| 参数                         | 必填     | 描述                                        |
| -------------------------- | -------- | -------------------------------------------- |
| aws.s3.enable_ssl          | 是       | 指定是否启用SSL连接。 <br />有效值：`true`和`false`。默认值：`true`。 |
| aws.s3.enable_path_style_access | 是       | 指定是否启用路径样式访问。<br />有效值：`true`和`false`。默认值：`false`。对于MinIO，必须将值设置为`true`。<br />路径样式URL使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果在美国西部（俄勒冈）区域中创建了一个名为`DOC-EXAMPLE-BUCKET1`的存储桶，并希望访问该存储桶中的`alice.jpg`对象，则可以使用以下路径样式URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint            | 是       | 用于连接到S3兼容存储系统的终端。            |
| aws.s3.access_key          | 是       | 您IAM用户的访问密钥。                           |
| aws.s3.secret_key          | 是       | 您IAM用户的秘密密钥。                           |

##### Microsoft Azure存储

###### Azure Blob存储

如果选择Blob存储作为Paimon集群的存储，需执行以下操作之一：

- 选择共享密钥身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                         | 必填     | 描述                                  |
  | -------------------------- | -------- | ------------------------------------ |
  | azure.blob.storage_account | 是       | 您Blob存储账户的用户名。                    |
  | azure.blob.shared_key      | 是       | 您Blob存储账户的共享密钥。                    |

- 选择SAS令牌身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                   | 必填     | 描述                                                  |
  | --------------------- | -------- | ---------------------------------------------------- |
  | azure.blob.account_name   | 是       | 您Blob存储账户的用户名。                                  |
  | azure.blob.container_name | 是       | 存储您数据的Blob容器名称。                                  |
  | azure.blob.sas_token      | 是       | 用于访问您Blob存储账户的SAS令牌。                                |

###### Azure数据湖存储Gen1

如果选择数据湖存储Gen1作为Paimon集群的存储，需执行以下操作之一：

- 选择托管服务标识身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                                | 必填     | 描述                                  |
  | ---------------------------------- | -------- | ------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是       | 指定是否启用托管服务标识身份验证方法。将值设置为`true`。 |

- 选择服务主体身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                     | 必填     | 描述                                  |
  | --------------------- | -------- | ------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是       | 服务主体的客户（应用程序）ID。                  |
  | azure.adls1.oauth2_credential | 是       | 创建的新客户（应用程序）密钥的值。               |
  | azure.adls1.oauth2_endpoint   | 是       | 服务主体或应用程序的OAuth 2.0令牌终结点（v1）。              |

###### Azure数据湖存储Gen2

如果选择数据湖存储Gen2作为Paimon集群的存储，需执行以下操作之一：

- 选择托管标识身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                                | 必填     | 描述                                  |
  | ---------------------------------- | -------- | ------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | 是       | 指定是否启用托管标识身份验证方法。将值设置为`true`。 |
  | azure.adls2.oauth2_tenant_id        | 是       | 要访问其数据的租户ID。                      |
  | azure.adls2.oauth2_client_id        | 是       | 托管托管标识的客户（应用程序）ID。              |

- 选择共享密钥身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                   | 必填     | 描述                                      |
  | --------------------- | -------- | ---------------------------------------- |
  | azure.adls2.storage_account | 是       | 您数据湖存储Gen2存储账户的用户名。                |
  | azure.adls2.shared_key      | 是       | 您数据湖存储Gen2存储账户的共享密钥。                |

- 选择服务主体身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                          | 必填     | 描述                                  |
  | -------------------------- | -------- | ------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是       | 服务主体的客户（应用程序）ID。                  |
  | azure.adls2.oauth2_client_secret   | 是       | 创建的新客户（应用程序）密钥的值。               |
  | azure.adls2.oauth2_client_endpoint | 是       | 服务主体或应用程序的OAuth 2.0令牌终结点（v1）。          |

##### Google GCS

如果选择Google GCS作为Paimon集群的存储，需执行以下操作之一：

- 选择基于VM的身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                                  | 默认值 | 值示例                               | 描述                                |
  | ----------------------------------- | ----- | ----------------------------------- | ----------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | FALSE | TRUE                                | 指定是否直接使用绑定到您计算引擎的服务账户。 |

- 选择基于服务账户的身份验证方法时，请如下配置“StorageCredentialParams”：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下表格描述了需要在“StorageCredentialParams”中配置的参数。

  | 参数                              | 默认值 | 值示例                            | 描述                                |
  | ------------------------------- | ----- | --------------------------------- | ----------------------------------- |
### 选项

| 参数                                        | 默认值  | 值示例                                         | 描述                                                         |
| ----------------------------------------- | ------- | --------------------------------------------- | ------------------------------------------------------------ |

| gcp.gcs.use_compute_engine_service_account | FALSE   | TRUE                                          | 指定是否直接使用绑定到您的Compute Engine的服务帐号。                |

| gcp.gcs.impersonation_service_account      | ""      | "hello"                                       | 您想要冒充的服务帐号。                                                |

- 如果您选择基于身份冒充的认证方法，请如下配置`StorageCredentialParams`：

  - 使VM实例冒充服务帐号：


    ```SQL

    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```


    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  - 使一个服务帐号（暂时命名为元服务帐号）冒充另一个服务帐号（暂时命名为数据服务帐号）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",

    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"

    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

### 示例


以下示例将创建名为`paimon_catalog_fs`的Paimon目录，其元数据存储类型`paimon.catalog.type`设置为`filesystem`，以从您的Paimon集群查询数据。

#### AWS S3

- 如果您选择基于实例配置文件的身份验证方法，请执行类似以下命令：

  ```SQL

  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```


- 如果您选择基于假定角色的身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您选择基于IAM用户的身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

#### S3兼容存储系统

以MinIO为例。执行类似以下命令：

```SQL
CREATE EXTERNAL CATALOG paimon_catalog_fs
PROPERTIES
(
    "type" = "paimon",
    "paimon.catalog.type" = "filesystem",
    "paimon.catalog.warehouse" = "<paimon_warehouse_path>",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### Microsoft Azure存储

##### Azure Blob存储

- 如果您选择共享密钥身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<blob_paimon_warehouse_path>",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果您选择SAS令牌身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<blob_paimon_warehouse_path>",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure数据湖存储Gen1

- 如果您选择托管服务标识身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls1_paimon_warehouse_path>",
      "azure.adls1.use_managed_service_identity" = "true"
  );
  ```

- 如果您选择服务主体身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls1_paimon_warehouse_path>",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure数据湖存储Gen2

- 如果您选择托管身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 如果您选择共享密钥身份验证方法，请执行类似以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
```sql
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"
  );
  ```

- If you choose the Service Principal authentication method, run a command like below:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- If you choose the VM-based authentication method, run a command like below:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
      "gcp.gcs.use_compute_engine_service_account" = "true"
  );
  ```

- If you choose the service account-based authentication method, run a command like below:

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  );
  ```

- If you choose the impersonation-based authentication method:

  - If you make a VM instance impersonate a service account, run a command like below:

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    PROPERTIES
    (
        "type" = "paimon",
        "paimon.catalog.type" = "filesystem",
        "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    );
    ```

  - If you make a service account impersonate another service account, run a command like below:

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    PROPERTIES
    (
        "type" = "paimon",
        "paimon.catalog.type" = "filesystem",
        "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    );
    ```

## View Paimon catalogs

You can use [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) to query all catalogs in the current StarRocks cluster:

```SQL
SHOW CATALOGS;
```

You can also use [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) to query the creation statement of an external catalog. The following example queries the creation statement of a Paimon catalog named `paimon_catalog_fs`:

```SQL
SHOW CREATE CATALOG paimon_catalog_fs;
```

## Drop a Paimon catalog

You can use [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) to drop an external catalog.

The following example drops a Paimon catalog named `paimon_catalog_fs`:

```SQL
DROP Catalog paimon_catalog_fs;
```

## View the schema of a Paimon table

You can use one of the following syntaxes to view the schema of a Paimon table:

- View schema

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>;
  ```

- View schema and location from the CREATE statement

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>;
  ```

## Query a Paimon table

1. Use [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) to view the databases in your Paimon cluster:

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. Use [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) to switch to the destination catalog in the current session:

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   Then, use [USE](../../sql-reference/sql-statements/data-definition/USE.md) to specify the active database in the current session:

   ```SQL
   USE <db_name>;
   ```

   Or, you can use [USE](../../sql-reference/sql-statements/data-definition/USE.md) to directly specify the active database in the destination catalog:

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. Use [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) to query the destination table in the specified database:

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## Load data from Paimon

Suppose you have an OLAP table named `olap_tbl`, you can transform and load data like below:

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;
```