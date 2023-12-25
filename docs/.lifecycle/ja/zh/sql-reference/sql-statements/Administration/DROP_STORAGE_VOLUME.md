---
displayed_sidebar: Chinese
---

# ストレージボリュームの削除

## 機能

指定されたストレージボリュームを削除します。削除されたストレージボリュームは参照できません。この機能はv3.1からサポートされています。

> **注意**
>
> - 指定されたストレージボリュームのDROP権限を持つユーザーのみがこの操作を実行できます。
> - デフォルトのストレージボリュームおよび内蔵ストレージボリューム `builtin_storage_volume` は削除できません。ストレージボリュームがデフォルトかどうかは [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md) で確認できます。
> - 既存のデータベースやクラウドネイティブテーブルによって参照されているストレージボリュームは削除できません。

## 文法

```SQL
DROP STORAGE VOLUME [ IF EXISTS ] <storage_volume_name>
```

## パラメータ説明

| **パラメータ**      | **説明**               |
| ------------------- | ---------------------- |
| storage_volume_name | 削除するストレージボリュームの名前。 |

## 例

例1: ストレージボリューム `my_s3_volume` を削除します。

```Plain
MySQL > DROP STORAGE VOLUME my_s3_volume;
Query OK, 0 rows affected (0.01 sec)
```

## 関連するSQL

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)
