---
displayed_sidebar: Chinese
---

# ストレージボリュームの表示

## 機能

現在のStarRocksクラスター内のストレージボリュームを表示します。この機能はv3.1からサポートされています。

:::tip

この操作には権限は必要ありません。

:::

## 文法

```SQL
SHOW STORAGE VOLUMES [ LIKE '<pattern>' ]
```

## パラメータ説明

| **パラメータ** | **説明**               |
| -------------- | ---------------------- |
| pattern        | ストレージボリュームをマッチングするためのパターン。 |

## 戻り値

| **戻り値**       | **説明**       |
| ---------------- | -------------- |
| Storage Volume   | ストレージボリュームの名前。 |

## 例

例1：現在のStarRocksクラスター内のすべてのストレージボリュームを表示します。

```SQL
MySQL > SHOW STORAGE VOLUMES;
+----------------+
| Storage Volume |
+----------------+
| my_s3_volume   |
+----------------+
1 row in set (0.01 sec)
```

## 関連SQL

- [CREATE STORAGE VOLUME](./CREATE_STORAGE_VOLUME.md)
- [ALTER STORAGE VOLUME](./ALTER_STORAGE_VOLUME.md)
- [DROP STORAGE VOLUME](./DROP_STORAGE_VOLUME.md)
- [SET DEFAULT STORAGE VOLUME](./SET_DEFAULT_STORAGE_VOLUME.md)
- [DESC STORAGE VOLUME](./DESC_STORAGE_VOLUME.md)
