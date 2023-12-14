---
displayed_sidebar: "Chinese"
---

# 使用MinIO共享数据

从`../../assets/commonMarkdown/sharedDataIntro.md` 导入SharedDataIntro
从`../../assets/commonMarkdown/sharedDataCNconf.md` 导入SharedDataCNconf
从`../../assets/commonMarkdown/sharedDataUseIntro.md` 导入SharedDataUseIntro
从`../../assets/commonMarkdown/sharedDataUse.md` 导入SharedDataUse

<SharedDataIntro />

## 架构

![共享数据架构](../../assets/share_data_arch.png)

## 部署共享数据StarRocks集群

共享数据StarRocks集群的部署类似于共享无数据StarRocks集群。唯一的区别是在共享数据集群中需要部署CN而不是BE。本节仅列出了在部署共享数据StarRocks集群时需要在FE和CN的配置文件**fe.conf**和**cn.conf**中添加的额外FE和CN配置项。有关部署StarRocks集群的详细说明，请参阅[手动部署StarRocks](../../deployment/deploy_manually.md)。

### 针对共享数据StarRocks配置FE节点

在启动FE之前，在FE配置文件**fe.conf**中添加以下配置项。

#### run_mode

StarRocks集群的运行模式。有效值：

- `shared_data`
- `shared_nothing`（默认）。

> **注意**
>
> 无法同时为StarRocks集群采用`shared_data`和`shared_nothing`模式。不支持混合部署。
>
> 请勿在集群部署后更改`run_mode`。否则，集群将无法重新启动。不支持从无数据集群转换为共享数据集群，反之亦然。

#### cloud_native_meta_port

云原生元服务RPC端口。

- 默认值：`6090`

#### enable_load_volume_from_conf

是否允许StarRocks使用FE配置文件中指定的与对象存储相关的属性创建默认存储卷。有效值：

- `true`（默认）：如果在创建新的共享数据集群时将此项指定为`true`，StarRocks将使用FE配置文件中与对象存储相关的属性创建内置存储卷`builtin_storage_volume`，并将其设置为默认存储卷。但是，如果未指定对象存储相关的属性，StarRocks将无法启动。
- `false`：如果在创建新的共享数据集群时将此项指定为`false`，StarRocks将直接启动，而不会创建内置存储卷。在创建StarRocks中的任何对象之前，您必须手动创建存储卷并将其设置为默认存储卷。有关详细信息，请参见[创建默认存储卷](#create-default-storage-volume)。

从v3.1.0版本开始支持。

> **注意**
>
> 我们强烈建议您在从v3.0升级现有的共享数据集群时将此项保留为`true`。如果将此项指定为`false`，则升级前创建的数据库和表将变为只读，并且无法向其加载数据。

#### cloud_native_storage_type

您使用的对象存储类型。在共享数据模式下，StarRocks支持将数据存储在Azure Blob（从v3.1.1版本开始支持）和与S3协议兼容的对象存储中（如AWS S3、Google GCP和MinIO）。有效值：

- `S3`（默认）
- `AZBLOB`。

> 注意
>
> 如果将此参数指定为`S3`，则必须添加以`aws_s3`为前缀的参数。
>
> 如果将此参数指定为`AZBLOB`，则必须添加以`azure_blob`为前缀的参数。

#### aws_s3_path

用于存储数据的S3路径。它由您的S3存储桶的名称和其下的子路径（如果有）组成，例如`testbucket/subpath`。

#### aws_s3_endpoint

用于访问您的S3存储桶的端点，例如`https://s3.us-west-2.amazonaws.com`。

#### aws_s3_region

您的S3存储桶所在的区域，例如`us-west-2`。

#### aws_s3_use_aws_sdk_default_behavior

是否使用[AWS SDK默认凭证提供程序链](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html)。有效值：

- `true`
- `false`（默认）。

#### aws_s3_use_instance_profile

是否使用实例配置文件和假定的角色作为访问S3的凭据方法。有效值：

- `true`
- `false`（默认）。

- 如果使用基于IAM用户的凭据（访问密钥和机密密钥）访问S3，必须将此项指定为`false`，并指定`aws_s3_access_key`和`aws_s3_secret_key`。

- 如果使用实例配置文件访问S3，必须将此项指定为`true`。

- 如果使用假定角色访问S3，必须将此项指定为`true`，并指定`aws_s3_iam_role_arn`。

- 如果使用外部AWS账户，还必须指定`aws_s3_external_id`。

#### aws_s3_access_key

用于访问您的S3存储桶的访问密钥ID。

#### aws_s3_secret_key

用于访问您的S3存储桶的机密访问密钥。

#### aws_s3_iam_role_arn

具有您的S3存储桶特权的IAM角色的ARN。

#### aws_s3_external_id

用于跨账户访问您的S3存储桶的外部AWS账户的外部ID。

#### azure_blob_path

用于存储数据的Azure Blob存储路径。它由您存储账户中容器的名称和容器下的子路径（如果有）组成，例如`testcontainer/subpath`。

#### azure_blob_endpoint

您的Azure Blob存储账户的端点，例如`https://test.blob.core.windows.net`。

#### azure_blob_shared_key

用于授权对您的Azure Blob存储发出请求的共享密钥。

#### azure_blob_sas_token

用于授权对您的Azure Blob存储发出请求的共享访问签名（SAS）。

> **注意**
>
> 仅可以在创建共享数据StarRocks集群后修改与凭据相关的配置项。如果更改了原存储路径相关的配置项，您在更改前创建的数据库和表将变为只读，并且无法向其加载数据。

如果要在创建集群后手动创建默认存储卷，只需添加以下配置项：

```Properties
run_mode = shared_data
cloud_native_meta_port = <meta_port>
enable_load_volume_from_conf = false
```

## 针对共享数据StarRocks配置CN节点

<SharedDataCNconf />

## 使用您的共享数据StarRocks集群

<SharedDataUseIntro />

以下示例为MinIO存储桶`defaultbucket`创建了一个名为`def_volume`的存储卷，并使用访问密钥和机密访问密钥凭据启用了存储卷，并将其设置为默认存储卷：

```SQL
CREATE STORAGE VOLUME def_volume
TYPE = S3
LOCATIONS = ("s3://defaultbucket/test/")
PROPERTIES
(
    "enabled" = "true",
    "aws.s3.region" = "us-west-2",
    "aws.s3.endpoint" = "https://hostname.domainname.com:portnumber",
    "aws.s3.access_key" = "xxxxxxxxxx",
    "aws.s3.secret_key" = "yyyyyyyyyy"
);

SET def_volume AS DEFAULT STORAGE VOLUME;
```

<SharedDataUse />