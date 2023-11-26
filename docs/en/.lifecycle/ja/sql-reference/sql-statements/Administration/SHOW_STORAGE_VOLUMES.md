---
displayed_sidebar: "Japanese"
---

# ストレージボリュームの表示

## 説明

StarRocksクラスターのストレージボリュームを表示します。この機能はv3.1からサポートされています。

## 構文

```SQL
SHOW STORAGE VOLUMES [ LIKE '<pattern>' ]
```

## パラメータ

| **パラメータ** | **説明**                                   |
| ------------- | ------------------------------------------ |
| pattern       | ストレージボリュームにマッチするパターン。 |

## 戻り値

| **戻り値**       | **説明**                 |
| -------------- | ----------------------- |
| Storage Volume | ストレージボリュームの名前。 |

## 例

例1: StarRocksクラスターのすべてのストレージボリュームを表示します。

```Plain
MySQL > SHOW STORAGE VOLUMES;
+----------------+
| Storage Volume |
+----------------+
| my_s3_volume   |
+----------------+
1 row in set (0.01 sec)
```

## 関連するSQL文

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
