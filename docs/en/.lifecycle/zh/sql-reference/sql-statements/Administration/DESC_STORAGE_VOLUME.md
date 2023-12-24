---
displayed_sidebar: English
---

# 存储卷描述

## 描述

描述存储卷。此功能从 v3.1 版本开始支持。

> **注意**
>
> 只有对特定存储卷具有 USAGE 权限的用户才能执行此操作。

## 语法

```SQL
DESC[RIBE] STORAGE VOLUME <storage_volume_name>
```

## 参数

| **参数**            | **描述**                   |
| ------------------- | -------------------------- |
| storage_volume_name | 要描述的存储卷的名称。    |

## 返回值

| **返回** | **描述**                                              |
| ---------- | ------------------------------------------------------------ |
| 名字       | 存储卷的名称。                              |
| 类型       | 远程存储系统的类型。有效值： `S3` 和 `AZBLOB`。 |
| IsDefault（默认）  | 存储卷是否为默认存储卷。    |
| 位置   | 远程存储系统的位置。                   |
| 参数     | 用于访问远程存储系统的凭据信息。 |
| 启用    | 存储卷是否已启用。                       |
| 评论    | 存储卷的注释。                           |

## 例子

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

- [创建存储卷](./CREATE_STORAGE_VOLUME.md)
- [更改存储卷](./ALTER_STORAGE_VOLUME.md)
- [丢弃存储卷](./DROP_STORAGE_VOLUME.md)
- [设置默认存储卷](./SET_DEFAULT_STORAGE_VOLUME.md)
- [显示存储卷](./SHOW_STORAGE_VOLUMES.md)
