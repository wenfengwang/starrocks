---
displayed_sidebar: "Japanese"
---

# デフォルトのストレージボリュームを設定する

## 説明

ストレージボリュームをデフォルトのストレージボリュームとして設定します。外部データソースのためのストレージボリュームを作成した後、StarRocksクラスタのデフォルトのストレージボリュームとして設定することができます。この機能はv3.1からサポートされています。

> **注意**
>
> - 特定のストレージボリュームに対するUSAGE権限を持つユーザーのみがこの操作を実行できます。
> - デフォルトのストレージボリュームは削除または無効にすることはできません。
> - システム統計情報をStarRocksはデフォルトのストレージボリュームに保存するため、共有データのStarRocksクラスタにはデフォルトのストレージボリュームを設定する必要があります。

## 構文

```SQL
SET <storage_volume_name> AS DEFAULT STORAGE VOLUME
```

## パラメーター

| パラメーター        | 説明                                                         |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | デフォルトのストレージボリュームとして設定するストレージボリュームの名前。 |

## 例

例1：ストレージボリューム`my_s3_volume`をデフォルトのストレージボリュームとして設定します。

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