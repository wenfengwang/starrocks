---
displayed_sidebar: "Japanese"
---

# デフォルトのストレージボリュームを設定する

## 説明

ストレージボリュームをデフォルトのストレージボリュームとして設定します。外部データソースのためのストレージボリュームを作成した後、それをStarRocksクラスターのデフォルトのストレージボリュームとして設定することができます。この機能はv3.1からサポートされています。

> **注意**
>
> - 特定のストレージボリュームにUSAGE権限を持つユーザーのみがこの操作を実行できます。
> - デフォルトのストレージボリュームは削除または無効化することはできません。
> - 共有データのStarRocksクラスターでは、デフォルトのストレージボリュームを設定する必要があります。なぜなら、StarRocksはシステム統計情報をデフォルトのストレージボリュームに保存するからです。

## 構文

```SQL
SET <storage_volume_name> AS DEFAULT STORAGE VOLUME
```

## パラメータ

| **パラメータ**       | **説明**                                                     |
| ------------------- | ------------------------------------------------------------ |
| storage_volume_name | デフォルトのストレージボリュームとして設定するストレージボリュームの名前。 |

## 例

例1: ストレージボリューム `my_s3_volume` をデフォルトのストレージボリュームとして設定する。

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
