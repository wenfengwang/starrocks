---
displayed_sidebar: Chinese
---

# デフォルトのストレージボリュームを設定

## 機能

指定されたストレージボリュームをデフォルトのストレージボリュームとして設定します。リモートストレージシステムにストレージボリュームを作成した後、そのストレージボリュームをクラスターのデフォルトのストレージボリュームとして設定できます。この機能はv3.1からサポートされています。

> **注意**
>
> - 指定されたストレージボリュームのUSAGE権限を持つユーザーのみがこの操作を実行できます。
> - デフォルトのストレージボリュームは削除または無効化することはできません。
> - StarRocksのストレージと計算の分離されたクラスターには、デフォルトのストレージボリュームを設定する必要があります。なぜならStarRocksはシステム統計情報をデフォルトのストレージボリュームに保存するからです。

## 语法

```SQL
SET <storage_volume_name> AS DEFAULT STORAGE VOLUME
```

## パラメータ説明

| **パラメータ**      | **説明**               |
| ------------------- | ---------------------- |
| storage_volume_name | 設定するストレージボリュームの名前。 |

## 例

例1：ストレージボリューム `my_s3_volume` をデフォルトのストレージボリュームとして設定します。

```SQL
MySQL > SET my_s3_volume AS DEFAULT STORAGE VOLUME;
Query OK, 0 rows affected (0.01 sec)
```

## 関連するSQL

- [ストレージボリュームの作成](./CREATE_STORAGE_VOLUME.md)
- [ストレージボリュームの変更](./ALTER_STORAGE_VOLUME.md)
- [ストレージボリュームの削除](./DROP_STORAGE_VOLUME.md)
- [ストレージボリュームの説明](./DESC_STORAGE_VOLUME.md)
- [ストレージボリュームの表示](./SHOW_STORAGE_VOLUMES.md)
