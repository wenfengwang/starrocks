---
displayed_sidebar: English
---

# SHOW TABLET

## 説明

タブレットに関連する情報を表示します。

> **注記**
>
> v3.0以降、この操作にはSYSTEMレベルのOPERATE権限とTABLEレベルのSELECT権限が必要です。v2.5以前では、ADMIN_PRIV権限が必要です。

## 構文

### テーブルまたはパーティションのタブレット情報を照会

```sql
SHOW TABLET
FROM [<db_name>.]<table_name>
[PARTITION(<partition_name>, ...)]
[
WHERE [version = <version_number>] 
    [[AND] backend_id = <backend_id>] 
    [[AND] STATE = "NORMAL"|"ALTER"|"CLONE"|"DECOMMISSION"]
]
[ORDER BY <field_name> [ASC | DESC]]
[LIMIT [<offset>,]<limit>]
```

| **パラメータ**       | **必須** | **説明**                                                     |
| -------------- | -------- | ------------------------------------------------------------ |
| db_name        | いいえ       | データベース名。指定しない場合、デフォルトで現在のデータベースが使用されます。                     |
| table_name     | はい       | タブレット情報を照会するテーブルの名前。このパラメータは必須です。指定しない場合、エラーが返されます。                                                     |
| partition_name | いいえ       | タブレット情報を照会するパーティションの名前。                                                     |
| version_number | いいえ       | データのバージョン番号。                                                   |
| backend_id     | いいえ       | タブレットのレプリカがあるBEのID。                                 |
| STATE          | いいえ       | タブレットレプリカの状態。<ul><li>`NORMAL`: レプリカは正常です。</li><li>`ALTER`: レプリカにロールアップやスキーマ変更が行われています。</li><li>`CLONE`: レプリカがクローン作成中です。（この状態のレプリカは使用できません）</li><li>`DECOMMISSION`: レプリカが使用停止中です。</li></ul> |
| field_name     | いいえ       | 結果の並び替えに使用するフィールド。`SHOW TABLET FROM <table_name>`で返されるすべてのフィールドは並び替え可能です。<ul><li>昇順で結果を表示する場合は`ORDER BY field_name ASC`を使用します。</li><li>降順で結果を表示する場合は`ORDER BY field_name DESC`を使用します。</li></ul> |
| offset         | いいえ       | 結果からスキップするタブレットの数。例えば、`OFFSET 5`は最初の5つのタブレットをスキップします。デフォルト値は0です。 |
| limit          | いいえ       | 返すタブレットの数。例えば、`LIMIT 10`は10個のタブレットのみを返します。このパラメータが指定されていない場合、フィルタ条件を満たすすべてのタブレットが返されます。 |

### 単一のタブレットの情報を照会

`SHOW TABLET FROM <table_name>`を使用してすべてのタブレットIDを取得した後、単一のタブレットの情報を照会できます。

```sql
SHOW TABLET <tablet_id>
```

| **パラメータ**  | **必須** | **説明**       |
| --------- | -------- | -------------- |
| tablet_id | はい       | タブレットID |

## 戻り値フィールドの説明

### テーブルまたはパーティションのタブレット情報を照会

```plain
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
| TabletId | ReplicaId | BackendId | SchemaHash | Version | VersionHash | LstSuccessVersion | LstSuccessVersionHash | LstFailedVersion | LstFailedVersionHash | LstFailedTime | DataSize | RowCount | State  | LstConsistencyCheckTime | CheckVersion | CheckVersionHash | VersionCount | PathHash | MetaUrl  | CompactionStatus  |
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
```

| **フィールド**                | **説明**                        |
| ----------------------- | ------------------------------- |
| TabletId                | タブレットID。                   |
| ReplicaId               | レプリカID。                      |
| BackendId               | レプリカが配置されているBEのID。  |
| SchemaHash              | スキーマハッシュ（ランダムに生成される）。        |
| Version                 | データのバージョン番号。                     |
| VersionHash             | データバージョン番号のハッシュ。              |
| LstSuccessVersion       | 最後に成功したロードのバージョン。        |
| LstSuccessVersionHash   | 最後に成功したロードのバージョンのハッシュ。 |
| LstFailedVersion        | 最後に失敗したロードのバージョン。`-1`は失敗したバージョンがないことを示します。 |
| LstFailedVersionHash    | 最後に失敗したバージョンのハッシュ。 |
| LstFailedTime           | 最後に失敗したロードの時刻。`NULL`はロード失敗がないことを示します。       |
| DataSize                | タブレットのデータサイズ。          |
| RowCount                | タブレットのデータ行数。            |
| State                   | タブレットのレプリカ状態。           |
| LstConsistencyCheckTime | 最後の整合性チェックの時刻。`NULL`は整合性チェックが実行されていないことを示します。 |

| CheckVersion            | 整合性チェックが実行されたデータバージョン。 `-1` は、バージョンがチェックされなかったことを示します。    |
| CheckVersionHash        | 整合性チェックが実行されたバージョンのハッシュ。         |
| VersionCount            | データバージョンの総数。                      |
| PathHash                | タブレットが保存されているディレクトリのハッシュ。        |
| MetaUrl                 | より多くのメタ情報を照会するために使用される URL。     |
| CompactionStatus        | データバージョンのコンパクション状態を照会するために使用される URL。    |

### 特定のタブレットの情報を照会する

```Plain
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
| DbName | TableName | PartitionName | IndexName | DbId | TableId | PartitionId | IndexId | IsSync | DetailCmd |
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
```

| **フィールド**      | **説明**                                                     |
| ------------- | ------------------------------------------------------------ |
| DbName        | タブレットが属するデータベースの名前。      |
| TableName     | タブレットが属するテーブルの名前。         |
| PartitionName | タブレットが属するパーティションの名前。     |
| IndexName     | インデックス名。                                            |
| DbId          | データベースID。                                           |
| TableId       | テーブルID。                                              |
| PartitionId   | パーティションID。                                          |
| IndexId       | インデックスID。                                              |
| IsSync        | タブレット上のデータがテーブルメタと一致しているかどうか。 `true` はデータが一致しており、タブレットが正常であることを示します。 `false` はタブレットにデータが欠落していることを示します。 |
| DetailCmd     | 詳細情報を照会するために使用される URL。                    |

## 例

データベース `example_db` に `test_show_tablet` テーブルを作成します。

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
PARTITION p20210109 VALUES [("2021-01-09"), ("2021-01-10"]))
DISTRIBUTED BY HASH(`k1`, `k2`, `k3`);
```

- 例 1: 指定されたテーブル内のすべてのタブレットの情報を照会します。以下の例は、返された情報から1つのタブレットの情報のみを抜粋しています。

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

- 例 2: タブレット 9588955 の情報を照会します。

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

- 例 3: パーティション `p20210103` のタブレットの情報を照会します。

    ```sql
    SHOW TABLET FROM test_show_tablet partition(p20210103);
    ```

- 例 4: 10個のタブレットの情報を返します。

    ```sql
        SHOW TABLET FROM test_show_tablet limit 10;
    ```

- 例 5: オフセット5で10個のタブレットの情報を返します。

    ```sql
    SHOW TABLET FROM test_show_tablet limit 5,10;
    ```

- 例 6: `backendid`、`version`、`state` によってタブレットをフィルタリングします。

    ```sql
        SHOW TABLET FROM test_show_tablet
        WHERE backendid = 10004 and version = 1 and state = "NORMAL";
    ```

- 例 7: `version` によってタブレットをソートします。

    ```sql

        SHOW TABLET FROM table_name where backendid = 10004 order by version;
    ```

- 例 8: インデックス名が `test_show_tablet` であるタブレットの情報を返します。

    ```sql
    SHOW TABLET FROM test_show_tablet where indexname = "test_show_tablet";
    ```

