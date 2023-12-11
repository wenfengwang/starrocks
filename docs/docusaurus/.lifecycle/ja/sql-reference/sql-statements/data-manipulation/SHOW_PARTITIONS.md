---
displayed_sidebar: "Japanese"
---

# パーティションの表示

## 説明

一般的なパーティションと[一時的なパーティション](../../../table_design/Temporary_partition.md)を含むパーティション情報を表示します。

## 構文

```sql
[db_name.]table_name [WHERE] [ORDER BY] [LIMIT] から [一時的な] パーティションを表示
```

> 注
>
> この構文はStarRocksテーブル(`"ENGINE" = "OLAP"`)のみをサポートしています。
> v3.0以降、この操作には指定されたテーブルのSELECT権限が必要です。v2.5およびそれ以前のバージョンでは、この操作には指定されたテーブルのSELECT__PRIV権限が必要です。

## 戻り値の説明

```plaintext
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
| PartitionId | PartitionName | VisibleVersion | VisibleVersionTime  | VisibleVersionHash | State  | PartitionKey | Range | DistributionKey    | Buckets | ReplicationNum | StorageMedium | CooldownTime        | LastConsistencyCheckTime | DataSize | IsInMemory | RowCount |
+-------------+---------------+----------------+---------------------+--------------------+--------+--------------+-------+--------------------+---------+----------------+---------------+---------------------+--------------------------+----------+------------+----------+
```

| **フィールド**          | **説明**                                   |
| ------------------------ | ------------------------------------------ |
| PartitionId              | パーティションのID。                            |
| PartitionName            | パーティションの名前。                            |
| VisibleVersion           | 最後の正常なロードトランザクションのバージョン番号。各正常なロードトランザクションでバージョン番号は1ずつ増加します。 |
| VisibleVersionTime       | 最後の正常なロードトランザクションのタイムスタンプ。          |
| VisibleVersionHash       | 最後の正常なロードトランザクションのバージョン番号のハッシュ値。     |
| State                    | パーティションの状態。固定値: `Normal`。                   |
| PartitionKey             | 1つまたは複数のパーティション列で構成されるパーティションキー。       |
| Range                    | パーティションの範囲、右半開区間です。                        |
| DistributionKey          | ハッシュバケットのバケットキー。                          |
| Buckets                  | パーティションのバケット数。                          |
| ReplicationNum           | パーティション内の各タブレットのレプリカ数。                   |
| StorageMedium            | パーティション内のデータを保存するためのストレージメディア。値`HHD`はハードディスクドライブを示し、値`SSD`はソリッドステートドライブを示します。 |
| CooldownTime             | パーティション内のデータのクールダウン時間。初期のストレージメディアがSSDの場合、このパラメータで指定された時間後に、ストレージメディアがSSDからHDDに切り替わります。形式: "yyyy-MM-dd HH:mm:ss"。 |
| LastConsistencyCheckTime | 最終整合性チェックの時間。`NULL`は整合性チェックが行われなかったことを示します。  |
| DataSize                 | パーティション内のデータのサイズ。                      |
| IsInMemory               | パーティション内のすべてのデータがメモリに格納されているかどうか。       |
| RowCount                 | パーティションのデータ行数。                        |

## 例

1. 指定されたデータベース`test`の指定されたテーブル`site_access`からすべての通常のパーティションの情報を表示します。

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

2. 指定されたデータベース`test`の指定されたテーブル`site_access`からすべての一時的なパーティションの情報を表示します。

    ```sql
    SHOW TEMPORARY PARTITIONS FROM test.site_access;
    ```

3. 指定されたデータベース`test`の指定されたテーブル`site_access`の指定されたパーティション`p1`の情報を表示します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access WHERE PartitionName = "p1";
    ```

4. 指定されたデータベース`test`の指定されたテーブル`site_access`の最新のパーティション情報を表示します。

    ```sql
    -- 通常のパーティション
    SHOW PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    -- 一時的なパーティション
    SHOW TEMPORARY PARTITIONS FROM test.site_access ORDER BY PartitionId DESC LIMIT 1;
    ```