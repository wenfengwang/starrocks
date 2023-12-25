---
displayed_sidebar: English
---

# SHOW PARTITIONS

## 説明

共通パーティションおよび[一時パーティション](../../../table_design/Temporary_partition.md)を含むパーティション情報を表示します。

## 構文

```sql
SHOW [TEMPORARY] PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

> 注意
>
> この構文はStarRocksテーブル（`"ENGINE" = "OLAP"`）のみをサポートします。
> v3.0以降、この操作には指定されたテーブルに対するSELECT権限が必要です。v2.5以前のバージョンでは、この操作には指定されたテーブルに対するSELECT_PRIV権限が必要です。

## 戻り値フィールドの説明

```plaintext
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
```

| **フィールド**                | **説明**                                              |
| ------------------------ | ------------------------------------------------------------ |
| PartitionId              | パーティションのIDです。                                |
| PartitionName            | パーティションの名前です。                                   |
| VisibleVersion           | 最後に成功したロードトランザクションのバージョン番号です。バージョン番号は、成功したロードトランザクションごとに1ずつ増加します。 |
| VisibleVersionTime       | 最後に成功したロードトランザクションのタイムスタンプです。       |
| VisibleVersionHash       | 最後に成功したロードトランザクションのバージョン番号のハッシュ値です。 |
| State                    | パーティションの状態です。固定値：`Normal`。           |
| PartitionKey             | 一つ以上のパーティション列から構成されるパーティションキーです。 |
| Range                    | パーティションの範囲で、右半開区間です。 |
| DistributionKey          | ハッシュバケットのバケットキーです。                            |
| Buckets                  | パーティションのバケット数です。                     |
| ReplicationNum           | パーティション内のタブレットごとのレプリカ数です。        |
| StorageMedium            | パーティションのデータを格納するストレージメディアです。`HDD`はハードディスクドライブ、`SSD`はソリッドステートドライブを指します。 |
| CooldownTime             | パーティション内のデータのクールダウン時間です。初期ストレージメディアがSSDの場合、このパラメータによって指定された時間が経過すると、ストレージメディアはSSDからHDDに切り替わります。形式："yyyy-MM-dd HH:mm:ss"。 |
| LastConsistencyCheckTime | 最後の整合性チェックの時刻です。`NULL`は整合性チェックが実行されていないことを示します。 |
| DataSize                 | パーティション内のデータサイズです。                          |
| IsInMemory               | パーティション内の全てのデータがメモリに格納されているかどうかです。          |
| RowCount                 | パーティションのデータ行数です。                    |
| MaxCS                    | パーティションの最大コンパクションスコアです。共有データクラスタ専用です。                    |

## 例

1. 指定されたデータベース`test`の指定されたテーブル`site_access`のすべての通常パーティションの情報を表示します。

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

2. 指定されたデータベース`test`の指定されたテーブル`site_access`のすべての一時パーティションの情報を表示します。

    ```sql
    SHOW TEMPORARY PARTITIONS FROM test.site_access;
    ```

3. 指定されたデータベース`test`の指定されたテーブル`site_access`の特定のパーティション`p1`の情報を表示します。

    ```sql
    -- 通常パーティション
    SHOW PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    -- 一時パーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    ```

4. 指定されたデータベース`test`の指定されたテーブル`site_access`の最新のパーティション情報を表示します。

    ```sql
    -- 通常パーティション
    SHOW PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    -- 一時パーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    ```
