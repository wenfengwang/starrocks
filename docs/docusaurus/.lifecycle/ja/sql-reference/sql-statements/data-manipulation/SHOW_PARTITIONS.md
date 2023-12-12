---
displayed_sidebar: "Japanese"
---

# パーティションの表示

## 説明

一般的なパーティションと[一時パーティション](../../../table_design/Temporary_partition.md)を含むパーティション情報を表示します。

## 構文

```sql
SHOW [TEMPORARY] PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

> 注記
>
> この構文はStarRocksテーブル（ `"ENGINE" = "OLAP"`）のみをサポートしています。
> v3.0以降、この操作には指定されたテーブルにおけるSELECT権限が必要です。v2.5およびそれ以前のバージョンでは、この操作には指定されたテーブルにおけるSELECT__PRIV権限が必要です。

## 戻り値のフィールドの説明

```plaintext
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
```

| **フィールド名**        | **説明**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| PartitionId              | パーティションのID。                                           |
| PartitionName            | パーティションの名前。                                         |
| VisibleVersion           | 直近の成功したロードトランザクションのバージョン番号。成功したロードトランザクションごとにバージョン番号は1ずつ増加します。    |
| VisibleVersionTime       | 直近の成功したロードトランザクションのタイムスタンプ。           |
| VisibleVersionHash       | 直近の成功したロードトランザクションのバージョン番号のハッシュ値。         |
| State                    | パーティションの状態。固定値：`Normal`。                           |
| PartitionKey             | 1つ以上のパーティション列から構成されるパーティションキー。               |
| Range                    | パーティションの範囲。右半開区間です。                               |
| DistributionKey          | ハッシュバケットのバケットキー。                                      |
| Buckets                  | パーティションのバケット数。                                          |
| ReplicationNum           | パーティション内の各タブレットのレプリカ数。                            |
| StorageMedium            | パーティション内のデータを格納するストレージメディア。値`HHD`はハードディスクドライブを示し、値`SSD`はソリッドステートドライブを示します。 |
| CooldownTime             | パーティション内のデータのクールダウン時間。初期のストレージメディアがSSDの場合、このパラメータで指定した時間後にストレージメディアがSSDからHDDに切り替わります。形式："yyyy-MM-dd HH:mm:ss"。 |
| LastConsistencyCheckTime | 直近の整合性チェックの時間。`NULL`は整合性チェックが行われていないことを示します。 |
| DataSize                 | パーティション内のデータサイズ。                                           |
| IsInMemory               | パーティション内のすべてのデータがメモリに格納されているかどうか。               |
| RowCount                 | パーティションのデータ行数。                                           |

## 例

1. 指定されたデータベース`test`内の指定されたテーブル`site_access`のすべての通常のパーティション情報を表示します。

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

2. 指定されたデータベース`test`内の指定されたテーブル`site_access`のすべての一時的なパーティション情報を表示します。

    ```sql
    SHOW TEMPORARY PARTITIONS FROM test.site_access;
    ```

3. 指定されたデータベース`test`内の指定されたテーブル`site_access`の指定されたパーティション`p1`の情報を表示します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    ```

4. 指定されたデータベース`test`内の指定されたテーブル`site_access`の最新のパーティション情報を表示します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    ```