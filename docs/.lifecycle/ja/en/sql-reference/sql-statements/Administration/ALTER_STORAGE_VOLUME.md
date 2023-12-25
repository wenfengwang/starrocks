---
displayed_sidebar: English
---

# ストレージボリュームの変更

## 説明

ストレージボリュームの認証プロパティ、コメント、またはステータス (`enabled`) を変更します。ストレージボリュームのプロパティの詳細については、[CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)を参照してください。この機能はv3.1からサポートされています。

> **注意**
>
> - 特定のストレージボリュームに対するALTER権限を持つユーザーのみがこの操作を実行できます。
> - 既存のストレージボリュームの`TYPE`、`LOCATIONS`、およびその他のパス関連プロパティは変更できません。資格情報関連のプロパティのみ変更可能です。パス関連の設定項目を変更した場合、変更前に作成したデータベースとテーブルは読み取り専用となり、データのロードができなくなります。
> - `enabled`が`false`の場合、該当するストレージボリュームは参照できません。

## 構文

```SQL
ALTER STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
{ COMMENT '<comment_string>'
| SET ("key" = "value"[,...]) }
```

## パラメーター

| **パラメーター**       | **説明**                                  |
| ------------------- | ---------------------------------------- |
| storage_volume_name | 変更するストレージボリュームの名前です。 |
| COMMENT             | ストレージボリュームのコメントです。       |

変更または追加可能なプロパティの詳細については、[CREATE STORAGE VOLUME - PROPERTIES](./CREATE_STORAGE_VOLUME.md#properties)を参照してください。

## 例

例 1: ストレージボリューム`my_s3_volume`を無効にする。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET ("enabled" = "false");
クエリ OK, 0行が影響しました (0.01 秒)
```

例 2: ストレージボリューム`my_s3_volume`の資格情報を変更する。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET (
    ->     "aws.s3.use_instance_profile" = "true"
    -> );
クエリ OK, 0行が影響しました (0.00 秒)
```

## 関連するSQLステートメント

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)
