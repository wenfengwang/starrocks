---
displayed_sidebar: Chinese
---

# ストレージボリュームの変更

## 機能

ストレージボリュームの認証属性、コメント、または状態（`enabled`）を変更します。ストレージボリュームの属性については、[CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)を参照してください。この機能はv3.1からサポートされています。

> **注意**
>
> - 特定のストレージボリュームのALTER権限を持つユーザーのみがこの操作を実行できます。
> - 既存のストレージボリュームの`TYPE`、`LOCATIONS`、およびその他のストレージパス関連のパラメータは変更できません。認証属性のみ変更可能です。ストレージパス関連の設定を変更した場合、それ以前に作成されたデータベースとテーブルは読み取り専用になり、データをインポートすることはできません。
> - `enabled`が`false`のストレージボリュームは参照できません。

## 文法

```SQL
ALTER STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
{ COMMENT '<comment_string>'
| SET ("key" = "value"[,...]) }
```

## パラメータ説明

| **パラメータ**      | **説明**                         |
| ------------------- | -------------------------------- |
| storage_volume_name | 変更または追加する属性のストレージボリュームの名前。 |
| COMMENT             | ストレージボリュームのコメント。 |

変更または追加可能なPROPERTIESの詳細については、[CREATE STORAGE VOLUME - PROPERTIES](./CREATE_STORAGE_VOLUME.md#properties)を参照してください。

## 例

例1：ストレージボリューム`my_s3_volume`を無効にする。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET ("enabled" = "false");
Query OK, 0 rows affected (0.01 sec)
```

例2：ストレージボリューム`my_s3_volume`の認証情報を変更する。

```Plain
MySQL > ALTER STORAGE VOLUME my_s3_volume
    -> SET (
    ->     "aws.s3.use_instance_profile" = "true"
    -> );
Query OK, 0 rows affected (0.00 sec)
```

## 関連SQL

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)
