---
displayed_sidebar: "Japanese"
---

# DESC STORAGE VOLUME

## 説明

ストレージボリュームについて説明します。この機能は v3.1 からサポートされています。

> **注意**
>
> 特定のストレージボリュームに対する USAGE 権限を持つユーザーだけがこの操作を実行できます。

## 構文

```SQL
DESC[RIBE] STORAGE VOLUME <storage_volume_name>
```

## パラメータ

| **パラメータ**     | **説明**                                 |
| ------------------- | ---------------------------------------- |
| storage_volume_name | 説明するストレージボリュームの名前。   |

## 戻り値

| **戻り値** | **説明**                                                     |
| ---------- | ------------------------------------------------------------ |
| Name       | ストレージボリュームの名前。                                |
| Type       | リモートストレージシステムのタイプ。有効な値: `S3` および `AZBLOB`。 |
| IsDefault  | ストレージボリュームがデフォルトのストレージボリュームかどうか。 |
| Location   | リモートストレージシステムの場所。                         |
| Params     | リモートストレージシステムにアクセスするために使用される資格情報。 |
| Enabled    | ストレージボリュームが有効かどうか。                       |
| Comment    | ストレージボリュームのコメント。                            |

## 例

Example 1: ストレージボリューム `my_s3_volume` を説明する。

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

## 関連する SQL ステートメント

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)