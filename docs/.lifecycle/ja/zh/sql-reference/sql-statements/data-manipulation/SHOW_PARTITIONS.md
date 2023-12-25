---
displayed_sidebar: Chinese
---

# SHOW PARTITIONS

## 機能

このステートメントは、通常のパーティションまたは一時的なパーティション情報を表示するために使用されます。

## 構文

```sql
SHOW [TEMPORARY] PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

> 説明
>
> * PartitionId、PartitionName、State、Buckets、ReplicationNum、LastConsistencyCheckTime などの列でフィルタリングをサポートします。
> * この構文は StarRocks テーブル（つまり `"ENGINE" = "OLAP"`）のみをサポートしています。
> * バージョン 3.0 以降、この操作には対応するテーブルの SELECT 権限が必要です。バージョン 3.0 以前では、対応するデータベースとテーブルの SELECT_PRIV 権限が必要でした。

## 戻り値の説明

```SQL
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
```

| **フィールド**                     | **説明**                                                         |
| ------------------------ | ------------------------------------------------------------ |
| PartitionId              | パーティション ID。                                                    |
| PartitionName            | パーティション名。                                                     |
| VisibleVersion           | 最後の成功したインポートのバージョン番号。インポートが成功するたびに、バージョン番号が 1 増加します。   |
| VisibleVersionTime       | 最後の成功したインポートの時間。                                   |
| VisibleVersionHash       | 最後の成功したインポートのバージョンハッシュ。                         |
| State                    | パーティションの状態。常に `Normal` です。                                |
| PartitionKey             | パーティションキーは、一つまたは複数のパーティション列で構成されます。                                                     |
| Range                    | Range パーティションの範囲で、左閉じ右開きの区間です。                           |
| DistributionKey          | パーティション内のデータがハッシュバケットに分配される際のキー。                           |
| Buckets                  | パーティション内のバケット数。                                           |
| ReplicationNum           | パーティション内の各 Tablet のレプリカ数。                                |
| StorageMedium            | データストレージ媒体。`HDD` は機械式ハードドライブ、`SSD` はソリッドステートドライブを意味します。           |
| CooldownTime             | データのクールダウン時間。最初にデータストレージ媒体が SSD だった場合、この時点以降に HDD に切り替わります。形式："yyyy-MM-dd HH:mm:ss"。|
| LastConsistencyCheckTime | 最後の一貫性チェックの時間。`NULL` は一貫性チェックが行われていないことを意味します。    |
| DataSize                 | パーティション内のデータサイズ。                                             |
| IsInMemory               | パーティションのデータが全てメモリ内に格納されているかどうか。                             |
| RowCount                 | パーティションのデータ行数。                                             |
| MaxCS                    | パーティションの最大 Compaction Score。計算とストレージが分離されたクラスターに限ります。                   |

## 例

1. 指定されたデータベース（例：`test`）の指定されたテーブル（例：`site_access`）のすべての正式なパーティション情報を照会します：

    ```SQL
    MySQL > show partitions from test.site_access\G
    *************************** 1. row ***************************
                PartitionId: 20990
            PartitionName: p2019 
            VisibleVersion: 1
        VisibleVersionTime: 2023-08-08 15:45:13
        VisibleVersionHash: 0
                    State: NORMAL
                PartitionKey: datekey
                    Range: [types: [DATE]; keys: [2019-01-01]; ..types: [DATE]; keys: [2020-01-01]; )
            DistributionKey: site_id
                    Buckets: 6
            ReplicationNum: 3
            StorageMedium: HDD
                CooldownTime: 9999-12-31 23:59:59
    LastConsistencyCheckTime: NULL
                    DataSize:  4KB   
                IsInMemory: false
                    RowCount: 3 
    1 row in set (0.00 sec)
    ```

2. 指定されたデータベース（例：`test`）の指定されたテーブル（例：`site_access`）のすべての一時的なパーティション情報を照会します：

    ```sql
    SHOW TEMPORARY PARTITIONS FROM test.site_access;
    ```

3. 指定されたデータベース（例：`test`）の指定されたテーブル（例：`site_access`）の特定のパーティション（例：`p1`）の情報を照会します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    ```

4. 指定されたデータベース（例：`test`）の指定されたテーブル（例：`site_access`）の最新のパーティション情報を照会します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    ```
