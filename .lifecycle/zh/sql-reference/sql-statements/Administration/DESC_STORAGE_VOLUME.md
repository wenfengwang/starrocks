---
displayed_sidebar: English
---

# 描述存储卷

## 概述

本功能用于描述存储卷，自 v3.1 版本起支持。

> **注意**
> 只有被授权 **USAGE** 权限的用户才能对特定存储卷进行此操作。

## 语法

```SQL
DESC[RIBE] STORAGE VOLUME <storage_volume_name>
```

## 参数

|参数|说明|
|---|---|
|storage_volume_name|要描述的存储卷的名称。|

## 返回值

|返回|说明|
|---|---|
|名称|存储卷的名称。|
|类型|远程存储系统的类型。有效值：S3 和 AZBLOB。|
|IsDefault|存储卷是否为默认存储卷。|
|位置|远程存储系统的位置。|
|Params|用于访问远程存储系统的凭据信息。|
|Enabled|是否启用存储卷。|
|评论|对存储卷的评论。|

## 示例

示例 1：描述名为 my_s3_volume 的存储卷。

```Plain
MySQL > DESCRIBE STORAGE VOLUME my_s3_volume\G
*************************** 1. row ***************************
     Name: my_s3_volume
     Type: S3
IsDefault: false
 Location: s3://defaultbucket/test/
   Params: {"aws.s3.access_key":"xxxxxxxxxx","aws.s3.secret_key":"yyyyyyyyyy","aws.s3.endpoint":"https://s3.us-west-2.amazonaws.com","aws.s3.region":"us-west-2","aws.s3.use_instance_profile":"true","aws.s3.use_aws_sdk_default_behavior":"false"}
  Enabled: false
  Comment: 
1 row in set (0.00 sec)
```

## 相关 SQL 语句

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [修改存储卷](./ALTER_STORAGE_VOLUME.md)
- [删除存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [显示存储卷列表](./SHOW_STORAGE_VOLUMES.md)
