---
displayed_sidebar: Chinese
---

# SHOW TABLET

## 機能

Tablet 関連情報を表示します。

> **注意**
>
> バージョン 3.0 から、この操作には SYSTEM レベルの OPERATE 権限と、対応するテーブルの SELECT 権限が必要です。バージョン 2.5 以前では、ADMIN_PRIV 権限が必要でした。

## 文法

### 特定のテーブルまたはパーティション内のすべての Tablet の情報を表示

WHERE 句を指定して、条件に合致する Tablet をフィルタリングすることもできます。

```sql
SHOW TABLET
FROM [<db_name>.]<table_name>
[PARTITION(<partition_name>, ...)]
[
WHERE
    [version = <version_number>] 
    [[AND] backendid = <backend_id>] 
    [[AND] STATE = "NORMAL"|"ALTER"|"CLONE"|"DECOMMISSION"]
]
[ORDER BY <field_name> [ASC | DESC]]
[LIMIT [<offset>,]<limit>]
```

| **パラメータ** | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | いいえ   | データベース名。指定しない場合は、現在のデータベースがデフォルトで使用されます。 |
| table_name     | はい     | テーブル名。指定は必須です。指定がない場合はエラーになります。 |
| partition_name | いいえ   | パーティション名。                                           |
| version_number | いいえ   | データのバージョン番号。                                     |
| backend_id     | いいえ   | Tablet のレプリカが存在する BE の ID。                       |
| STATE          | いいえ   | Tablet のレプリカの状態。<ul><li>`NORMAL`：レプリカは正常な状態です。</li><li>`ALTER`：レプリカは Rollup または schema change を行っています。</li><li>`CLONE`：レプリカは clone 状態です（clone が完了していないレプリカは使用できません）。</li><li>`DECOMMISSION`：レプリカは DECOMMISSION 状態です（オフライン）。</li></ul> |
| field_name     | いいえ   | 指定されたフィールドに基づいて結果を昇順または降順でソートします。`SHOW TABLET FROM <table_name>` で返されるフィールドはすべてソートフィールドとして使用できます。<ul><li>昇順にするには `ORDER BY field_name ASC` を指定します。</li><li>降順にするには `ORDER BY field_name DESC` を指定します。</li></ul> |
| offset         | いいえ   | 結果からスキップされる Tablet の数です。デフォルトは 0 です。例えば `OFFSET 5` は最初の 5 つの Tablet をスキップし、残りの結果を返します。 |
| limit          | いいえ   | 表示する Tablet の数を指定します。例えば `LIMIT 10` は 10 個の Tablet の情報を表示します。このパラメータを指定しない場合、フィルタ条件に合致するすべての Tablet がデフォルトで表示されます。 |

### 単一の Tablet の情報を表示

`SHOW TABLET FROM <table_name>` で取得したすべての Tablet ID の後、特定の Tablet の詳細情報をクエリすることができます。

```sql
SHOW TABLET <tablet_id>
```

| **パラメータ** | **必須** | **説明**       |
| -------------- | -------- | -------------- |
| tablet_id      | はい     | Tablet の ID。 |

## 戻り値の説明

### 特定のテーブルまたはパーティション内のすべての tablet を表示

```plain
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
| TabletId | ReplicaId | BackendId | SchemaHash | Version | VersionHash | LstSuccessVersion | LstSuccessVersionHash | LstFailedVersion | LstFailedVersionHash | LstFailedTime | DataSize | RowCount | State  | LstConsistencyCheckTime | CheckVersion | CheckVersionHash | VersionCount | PathHash | MetaUrl  | CompactionStatus  |
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
```

| **フィールド**          | **説明**                        |
| ----------------------- | ------------------------------- |
| TabletId                | テーブルの ID。                 |
| ReplicaId               | レプリカの ID。                 |
| BackendId               | レプリカが位置する BE の ID。   |
| SchemaHash              | スキーマハッシュ（ランダム生成）。 |
| Version                 | データのバージョン番号。         |
| VersionHash             | データのバージョンハッシュ。     |
| LstSuccessVersion       | 最後の成功したロードのバージョン。 |
| LstSuccessVersionHash   | 最後の成功したロードのバージョンハッシュ。 |
| LstFailedVersion        | 最後の失敗したロードのバージョン。`-1` は失敗したバージョンがないことを意味します。 |
| LstFailedVersionHash    | 最後の失敗したロードのバージョンハッシュ。 |
| LstFailedTime           | 最後の失敗したロードの時間。`NULL` は失敗がなかったことを意味します。 |
| DataSize                | その Tablet のデータサイズ。     |
| RowCount                | その Tablet の行数。             |
| State                   | Tablet のレプリカの状態。       |
| LstConsistencyCheckTime | 最後の一貫性チェックの時間。`NULL` は一貫性チェックが行われていないことを意味します。 |
| CheckVersion            | 一貫性チェックのバージョン。`-1` はチェックされたバージョンがないことを意味します。 |
| CheckVersionHash        | 一貫性チェックのバージョンハッシュ。 |
| VersionCount            | データのバージョン数。           |
| PathHash                | Tablet のストレージディレクトリのハッシュ。 |
| MetaUrl                 | URL 経由で更に多くのメタ情報を照会します。 |
| CompactionStatus        | URL 経由でコンパクションの状態を照会します。 |

### 単一の tablet を表示

```Plain
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
| DbName | TableName | PartitionName | IndexName | DbId | TableId | PartitionId | IndexId | IsSync | DetailCmd |
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
```

| **フィールド**    | **説明**                                                     |
| ----------------- | ------------------------------------------------------------ |
| DbName            | Tablet が属するデータベースの名前。                          |
| TableName         | Tablet が属するテーブルの名前。                              |
| PartitionName     | Tablet が属するパーティションの名前。                        |
| IndexName         | インデックスの名前。                                         |
| DbId              | データベースの ID。                                          |
| TableId           | テーブルの ID。                                              |
| PartitionId       | パーティションの ID。                                        |
| IndexId           | インデックスの ID。                                          |
| IsSync            | Tablet 上のデータがテーブルのメタデータと一致しているかどうかをチェックします。`false` はデータに不足があることを意味し、`true` はデータが正常であること、つまり Tablet が正常であることを意味します。 |
| DetailCmd         | URL 経由で更に多くの情報を照会します。                       |

## 例

`example_db` データベースに `test_show_tablet` テーブルを作成します。

```sql
CREATE TABLE `test_show_tablet` (
  `k1` date NULL COMMENT "",
  `k2` datetime NULL COMMENT "",
  `k3` char(20) NULL COMMENT "",
  `k4` varchar(20) NULL COMMENT "",
  `k5` boolean NULL COMMENT "",
  `k6` tinyint(4) NULL COMMENT "",
  `k7` smallint(6) NULL COMMENT "",
  `k8` int(11) NULL COMMENT "",
  `k9` bigint(20) NULL COMMENT "",
  `k10` largeint(40) NULL COMMENT "",
  `k11` float NULL COMMENT "",
  `k12` double NULL COMMENT "",
  `k13` decimal128(27, 9) NULL COMMENT ""
) ENGINE=OLAP 
DUPLICATE KEY(`k1`, `k2`, `k3`, `k4`, `k5`)
COMMENT "OLAP"
PARTITION BY RANGE(`k1`)
(PARTITION p20210101 VALUES [("2021-01-01"), ("2021-01-02")),
PARTITION p20210102 VALUES [("2021-01-02"), ("2021-01-03")),
PARTITION p20210103 VALUES [("2021-01-03"), ("2021-01-04")),
PARTITION p20210104 VALUES [("2021-01-04"), ("2021-01-05")),
PARTITION p20210105 VALUES [("2021-01-05"), ("2021-01-06")),
PARTITION p20210106 VALUES [("2021-01-06"), ("2021-01-07")),
PARTITION p20210107 VALUES [("2021-01-07"), ("2021-01-08")),
PARTITION p20210108 VALUES [("2021-01-08"), ("2021-01-09")),
PARTITION p20210109 VALUES [("2021-01-09"), ("2021-01-10")))
DISTRIBUTED BY HASH(`k1`, `k2`, `k3`);
```

- 指定されたデータベースの指定されたテーブルのすべての Tablet 情報を表示します。以下の例は、tablet 情報の中から一行だけを取り出して説明しています。

    ```plain
    mysql> show tablet from example_db.test_show_tablet\G
    *************************** 1. row ***************************
               TabletId: 9588955
              ReplicaId: 9588956
              BackendId: 10004
             SchemaHash: 0
                Version: 1
            VersionHash: 0
      LstSuccessVersion: 1
  LstSuccessVersionHash: 0
       LstFailedVersion: -1
   LstFailedVersionHash: 0
          LstFailedTime: NULL
               DataSize: 0B
               RowCount: 0
                  State: NORMAL
  LstConsistencyCheckTime: NULL
           CheckVersion: -1
       CheckVersionHash: 0
           VersionCount: 1
               PathHash: 0
                MetaUrl: http://172.26.92.141:8038/api/meta/header/9588955
       CompactionStatus: http://172.26.92.141:8038/api/compaction/show?tablet_id=9588955
    ```

- ID が 9588955 の Tablet の情報を表示します。

    ```plain
    mysql> show tablet 9588955\G
    *************************** 1. row ***************************
       DbName: example_db
    TableName: test_show_tablet
    PartitionName: p20210103
    IndexName: test_show_tablet
         DbId: 11145
      TableId: 9588953
  PartitionId: 9588946
      IndexId: 9588954
       IsSync: true
    DetailCmd: SHOW PROC '/dbs/11145/9588953/partitions/9588946/9588954/9588955';
    ```

- テーブル内のパーティション `p20210103` の Tablet 情報を表示します。

    ```sql
        SHOW TABLET FROM test_show_tablet PARTITION(p20210103);
    ```

- テーブル内の 10 個の Tablet 情報を返します。

    ```sql
        SHOW TABLET FROM test_show_tablet LIMIT 10;
    ```

- オフセット位置 5 から 10 個の Tablet 情報を取得します。

    ```sql
        SHOW TABLET FROM test_show_tablet LIMIT 5,10;
    ```

- `backendid`, `version`, `state` フィールドに基づいてフィルタリングします。

    ```sql
        SHOW TABLET FROM test_show_tablet
        WHERE backendid = 10004 AND version = 1 AND state = "NORMAL";
    ```

- `version` フィールドでソートします。

    ```sql
        SHOW TABLET FROM table_name WHERE backendid = 10004 ORDER BY version;
    ```

- インデックス名が `test_show_tablet` の Tablet 情報を表示します。

    ```sql
        SHOW TABLET FROM test_show_tablet WHERE indexname = "test_show_tablet";
    ```
