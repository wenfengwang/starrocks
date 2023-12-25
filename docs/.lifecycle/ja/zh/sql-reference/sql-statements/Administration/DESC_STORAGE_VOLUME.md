---
displayed_sidebar: Chinese
---

# DESC STORAGE VOLUME

## 機能

指定されたストレージボリュームの情報を表示します。この機能はv3.1からサポートされています。

> **注意**
>
> 指定されたストレージボリュームのUSAGE権限を持つユーザーのみがこの操作を実行できます。

## 構文

```SQL
DESC[RIBE] STORAGE VOLUME <storage_volume_name>
```

## パラメータ説明

| **パラメータ**        | **説明**                   |
| ------------------- | -------------------------- |
| storage_volume_name | 確認するストレージボリュームの名前。 |

## 戻り値

| **戻り値**  | **説明**                                                |
| --------- | ------------------------------------------------------- |
| Name      | ストレージボリュームの名前。                            |
| Type      | リモートストレージシステムのタイプ。有効な値：`S3`、`AZBLOB`、`HDFS`。 |
| IsDefault | そのストレージボリュームがデフォルトかどうか。            |
| Location  | リモートストレージシステムの場所。                      |
| Params    | リモートストレージシステムに接続するための認証情報。      |
| Enabled   | そのストレージボリュームが有効化されているかどうか。      |
| Comment   | ストレージボリュームのコメント。                        |

## 例

例1：ストレージボリューム `my_s3_volume` の情報を表示します。

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

## 関連するSQL

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)
