---
displayed_sidebar: "Japanese"
---

# SHOW PARTITIONS

## 説明

一般的なパーティションと[一時的なパーティション](../../../table_design/Temporary_partition.md)を含むパーティション情報を表示します。

## 構文

```sql
SHOW [TEMPORARY] PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

> 注意
>
> この構文は、StarRocksテーブル（`"ENGINE" = "OLAP"`）のみをサポートしています。
> v3.0以降、この操作には指定したテーブルに対するSELECT権限が必要です。v2.5以前のバージョンでは、この操作には指定したテーブルに対するSELECT__PRIV権限が必要です。

## 戻り値のフィールドの説明

```plaintext
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
```

| **フィールド**           | **説明**                                                     |
| ------------------------ | ------------------------------------------------------------ |
| PartitionId              | パーティションのIDです。                                |
| PartitionName            | パーティションの名前です。                                   |
| VisibleVersion           | 最後の成功したロードトランザクションのバージョン番号です。バージョン番号は、成功したロードトランザクションごとに1ずつ増加します。 |
| VisibleVersionTime       | 最後の成功したロードトランザクションのタイムスタンプです。       |
| VisibleVersionHash       | 最後の成功したロードトランザクションのバージョン番号のハッシュ値です。 |
| State                    | パーティションのステータスです。固定値: `Normal`。           |
| PartitionKey             | 1つ以上のパーティション列で構成されるパーティションキーです。 |
| Range                    | パーティションの範囲です。右半開区間です。 |
| DistributionKey          | ハッシュバケットのバケットキーです。                            |
| Buckets                  | パーティションのバケット数です。                     |
| ReplicationNum           | パーティション内の各タブレットのレプリカ数です。        |
| StorageMedium            | パーティション内のデータを格納するストレージメディアです。値`HHD`はハードディスクドライブを示し、値`SSD`はソリッドステートドライブを示します。 |
| CooldownTime             | パーティション内のデータのクールダウン時間です。初期のストレージメディアがSSDの場合、このパラメータで指定された時間後にストレージメディアがSSDからHDDに切り替わります。形式: "yyyy-MM-dd HH:mm:ss"。 |
| LastConsistencyCheckTime | 最後の整合性チェックの時間です。`NULL`は整合性チェックが実行されていないことを示します。 |
| DataSize                 | パーティション内のデータのサイズです。                          |
| IsInMemory               | パーティション内のすべてのデータがメモリに格納されているかどうかです。          |
| RowCount                 | パーティションのデータ行数です。                    |

## 例

1. 指定したデータベース`test`の指定したテーブル`site_access`のすべての通常のパーティションの情報を表示します。

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

2. 指定したデータベース`test`の指定したテーブル`site_access`のすべての一時的なパーティションの情報を表示します。

    ```sql
    SHOW TEMPORARY PARTITIONS FROM test.site_access;
    ```

3. 指定したデータベース`test`の指定したテーブル`site_access`の指定したパーティション`p1`の情報を表示します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    ```

4. 指定したデータベース`test`の指定したテーブル`site_access`の最新のパーティション情報を表示します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    ```
