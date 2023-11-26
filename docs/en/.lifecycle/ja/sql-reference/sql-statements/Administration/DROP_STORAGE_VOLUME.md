---
displayed_sidebar: "Japanese"
---

# ストレージボリュームの削除

## 説明

ストレージボリュームを削除します。削除されたストレージボリュームはもはや参照できません。この機能はv3.1からサポートされています。

> **注意**
>
> - 特定のストレージボリュームにDROP権限を持つユーザーのみがこの操作を実行できます。
> - デフォルトのストレージボリュームと組み込みのストレージボリューム`builtin_storage_volume`は削除できません。ストレージボリュームがデフォルトのストレージボリュームであるかどうかを確認するには、[DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)を使用できます。
> - 既存のデータベースやクラウドネイティブテーブルで参照されているストレージボリュームは削除できません。

## 構文

```SQL
DROP STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
```

## パラメータ

| **パラメータ**        | **説明**                                 |
| -------------------- | ---------------------------------------- |
| storage_volume_name  | 削除するストレージボリュームの名前。 |

## 例

例1: ストレージボリューム`my_s3_volume`を削除する。

```Plain
MySQL > DROP STORAGE VOLUME my_s3_volume;
Query OK, 0 rows affected (0.01 sec)
```

## 関連するSQLステートメント

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)
