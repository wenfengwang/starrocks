---
displayed_sidebar: English
---

# デフォルトのストレージボリュームを設定

## 説明

外部データソース用のストレージボリュームを作成した後、そのストレージボリュームをStarRocksクラスタのデフォルトのストレージボリュームとして設定できます。この機能はv3.1からサポートされています。

> **注意**
>
> - 特定のストレージボリュームに対するUSAGE権限を持つユーザーのみがこの操作を実行できます。
> - デフォルトのストレージボリュームは削除または無効化することはできません。
> - StarRocksはシステム統計情報をデフォルトのストレージボリュームに保存しますので、共有データを持つStarRocksクラスタにはデフォルトのストレージボリュームを設定する必要があります。

## 構文

```SQL
SET <storage_volume_name> AS DEFAULT STORAGE VOLUME
```

## パラメータ

| **パラメータ**       | **説明**                                              |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | デフォルトのストレージボリュームとして設定するストレージボリュームの名前です。 |

## 例

例 1: ストレージボリューム `my_s3_volume` をデフォルトのストレージボリュームとして設定します。

```SQL
MySQL > SET my_s3_volume AS DEFAULT STORAGE VOLUME;
Query OK, 0 rows affected (0.01 sec)
```

## 関連するSQLステートメント

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
- [SHOW STORAGE VOLUMES](./SHOW_STORAGE_VOLUMES.md)
