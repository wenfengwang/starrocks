```md
---
displayed_sidebar: "Japanese"
---

# ストレージボリュームの変更

## 説明

ストレージボリュームの認証情報、コメント、またはステータス（`enabled`）を変更します。ストレージボリュームのプロパティについて詳しくは[ストレージボリュームの作成](./CREATE_STORAGE_VOLUME.md)を参照してください。この機能はv3.1からサポートされています。

> **注意**
>
> - 特定のストレージボリュームに対するALTER権限を持つユーザーのみがこの操作を実行できます。
> - 既存のストレージボリュームの`TYPE`、`LOCATIONS`、およびその他のパス関連のプロパティは変更できません。認証関連のプロパティのみを変更できます。パス関連の構成項目を変更した場合、変更前に作成したデータベースおよびテーブルは読み取り専用になり、データをロードすることはできません。
> - `enabled`が`false`の場合、対応するストレージボリュームは参照できません。

## 構文

```SQL
ALTER STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
{ COMMENT '<comment_string>'
| SET ("key" = "value"[,...]) }
```

## パラメータ

| **パラメータ**       | **説明**                                     |
| ------------------- | ---------------------------------------- |
| storage_volume_name | 変更するストレージボリュームの名前。 |
| COMMENT             | ストレージボリュームのコメント。             |

変更または追加できるプロパティの詳細については、[ストレージボリュームの作成：プロパティ](./CREATE_STORAGE_VOLUME.md#properties)を参照してください。

## 例

例1: ストレージボリューム `my_s3_volume` を無効にする。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET ("enabled" = "false");
Query OK, 0 rows affected (0.01 sec)
```

例2: ストレージボリューム `my_s3_volume` の認証情報を変更する。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET (
    ->     "aws.s3.use_instance_profile" = "true"
    -> );
Query OK, 0 rows affected (0.00 sec)
```

## 関連するSQLステートメント

- [ストレージボリュームの作成](./CREATE_STORAGE_VOLUME.md)
- [ストレージボリュームの削除](./DROP_STORAGE_VOLUME.md)
- [デフォルトのストレージボリュームを設定](./SET_DEFAULT_STORAGE_VOLUME.md)
- [ストレージボリュームの説明](./DESC_STORAGE_VOLUME.md)
- [ストレージボリュームの表示](./SHOW_STORAGE_VOLUMES.md)
```