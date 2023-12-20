---
displayed_sidebar: English
---

# DESC STORAGE VOLUME

## 描述

描述存储卷。此功能从 v3.1 版本开始支持。

> **注意**
> 只有对特定存储卷具有 USAGE 权限的用户才能执行此操作。

## 语法

```SQL
DESC[RIBE] STORAGE VOLUME <storage_volume_name>
```

## 参数

|**参数**|**说明**|
|---|---|
|storage_volume_name|要描述的存储卷的名称。|

## 返回值

|**返回**|**说明**|
|---|---|
|Name|存储卷的名称。|
|Type|远程存储系统的类型。有效值：`S3` 和 `AZBLOB`。|
|IsDefault|存储卷是否为默认存储卷。|
|Location|远程存储系统的位置。|
|Params|用于访问远程存储系统的凭证信息。|
|Enabled|存储卷是否启用。|
|Comment|存储卷的备注。|

## 示例

示例 1：描述存储卷 `my_s3_volume`。

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

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)