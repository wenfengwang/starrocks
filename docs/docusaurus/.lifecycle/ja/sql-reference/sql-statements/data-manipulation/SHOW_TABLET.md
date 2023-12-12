---
displayed_sidebar: "Japanese"
---

# SHOW TABLET（タブレットの表示）

## 説明

タブレットに関する情報を表示します。

> **注**
>
> v3.0以降では、この操作にはSYSTEMレベルのOPERATE権限とTABLEレベルのSELECT権限が必要です。v2.5以前では、この操作にはADMIN_PRIV権限が必要です。

## 構文

### テーブルまたはパーティションのタブレットの情報をクエリする

```sql
SHOW TABLET
FROM[<db_name>.]<table_name>
[PARTITION(<partition_name>, ...]
[
WHERE[version =<version_number>]
[[AND]backendid=<backend_id>]
[[AND] STATE="NORMAL"|"ALTER"|"CLONE"|"DECOMMISSION"]
]
[ORDER BY<field_name>[ASC | DESC]]
[LIMIT[<offset>,]<limit>]
```

| **パラメータ**   | **必須** | **説明**   |
| -------------- | -------- | --------------- |
| db_name        | いいえ      | データベース名。このパラメータを指定しない場合、デフォルトで現在のデータベースが使用されます。          |
| table_name     | はい      | タブレット情報をクエリしたいテーブルの名前。このパラメータは必ず指定する必要があります。そうでない場合はエラーが返されます。    |
| partition_name | いいえ      | クエリしたいタブレット情報のパーティションの名前。          |
| version_number | いいえ      | データバージョン番号。    |
| backend_id     | いいえ      | タブレットのレプリカが存在するBEのID。            |
| STATE          | いいえ      | タブレットレプリカのステータス。<ul><li>`NORMAL`: レプリカが正常です。</li><li>`ALTER`: レプリカでロールアップまたはスキーマ変更が実行されています。</li><li>`CLONE`: レプリカがクローンされています（このステータスのレプリカは使用できません）。</li><li>`DECOMMISSION`: レプリカが廃止中です。</li></ul> |
| field_name     | いいえ      | 結果をソートするフィールド。`SHOW TABLET FROM <table_name>`で返されたすべてのフィールドがソート可能です。<ul><li>昇順で結果を表示する場合は、`ORDER BY field_name ASC`を使用します。</li><li>降順で結果を表示する場合は、`ORDER BY field_name DESC`を使用します。</li></ul> |
| offset         | いいえ      | 結果からスキップするタブレットの数。たとえば、`OFFSET 5`は最初の5つのタブレットをスキップすることを意味します。デフォルト値: 0。    |
| limit          | いいえ      | 返されるタブレットの数。たとえば、`LIMIT 10`は10個のタブレットのみを返します。このパラメータが指定されていない場合、フィルタ条件を満たすすべてのタブレットが返されます。 |

### 特定のタブレットの情報をクエリする

`SHOW TABLET FROM <table_name>`を使用してすべてのタブレットIDを取得した後、単一のタブレットの情報をクエリできます。

```sql
SHOW TABLET <tablet_id>
```

| **パラメータ**   | **必須** | **説明**             |
| --------- | -------- | --------------- |
| tablet_id | はい      | タブレットID |

## 戻り値フィールドの説明

### テーブルまたはパーティションのタブレットの情報をクエリする

```plain
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
| TabletId | ReplicaId | BackendId | SchemaHash | Version | VersionHash | LstSuccessVersion | LstSuccessVersionHash  | LstFailedVersion | LstFailedVersionHash | LstFailedTime  | DataSize | RowCount | State  | LstConsistencyCheckTime | CheckVersion | CheckVersionHash | VersionCount | PathHash | MetaUrl  | CompactionStatus  |
+----------+-----------+-----------+------------+---------+-------------+-------------------+-----------------------+------------------+----------------------+---------------+----------+----------+--------+-------------------------+--------------+------------------+--------------+----------+----------+-------------------+
```

| **フィールド**           | **説明**        |
| ----------------------- | ----------------- |
| TabletId                | テーブルID。     |
| ReplicaId               | レプリカID。       |
| BackendId               | レプリカが存在するBEのID。   |
| SchemaHash              | スキーマハッシュ（ランダムに生成されたもの）。     |
| Version                 | データバージョン番号。           |
| VersionHash             | データバージョン番号のハッシュ。          |
| LstSuccessVersion       | 最後に正常にロードされたバージョン。        |
| LstSuccessVersionHash   | 最後に正常にロードされたバージョンのハッシュ。    |
| LstFailedVersion        | 最後のロード失敗のバージョン。 `-1`は、失敗したバージョンがないことを示します。 |
| LstFailedVersionHash    | 最後の失敗バージョンのハッシュ。     |
| LstFailedTime           | 最後のロード失敗の時間。 `NULL`は、ロードの失敗がないことを示します。  |
| DataSize                | タブレットのデータサイズ。   |
| RowCount                | タブレットのデータ行数。     |
| State                   | タブレットのレプリカのステータス。   |
| LstConsistencyCheckTime | 最後の整合性チェックの時間。 `NULL`は、整合性チェックが実行されていないことを示します。 |
| CheckVersion            | 整合性チェックが実行されたデータバージョン。 `-1`は、チェックされたバージョンがないことを示します。   |
| CheckVersionHash        | 整合性チェックが実行されたバージョンのハッシュ。   |
| VersionCount            | データバージョンの合計数。         |
| PathHash                | タブレットが格納されているディレクトリのハッシュ。 |
| MetaUrl                 | より多くのメタ情報をクエリするために使用されるURL。 |
| CompactionStatus        | データバージョンのコンパクションステータスをクエリするために使用されるURL。 |

### 特定のタブレットの情報をクエリする

```plain
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
| DbName | TableName | PartitionName | IndexName | DbId | TableId | PartitionId | IndexId | IsSync | DetailCmd |
+--------+-----------+---------------+-----------+------+---------+-------------+---------+--------+-----------+
```

| **フィールド**      | **説明**   |
| ------------- | ------------------- |
| DbName        | タブレットが属するデータベースの名前。   |
| TableName     | タブレットが属するテーブルの名前。       |
| PartitionName | タブレットが属するパーティションの名前。  |
| IndexName     | インデックスの名前。             |
| DbId          | データベースID。                  |
| TableId       | テーブルID。                    |
| PartitionId   | パーティションID。                |
| IndexId       | インデックスID。                  |
| IsSync        | タブレットのデータがテーブルメタと一致しているかどうか。`true`は、データが一致し、タブレットが正常であることを示します。 `false`は、タブレットにデータが欠落していることを示します。 |
| DetailCmd     | より多くの情報をクエリするために使用されるURL。 |

## 例

データベース`example_db`にテーブル`test_show_tablet`を作成します。

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

- 例1: 指定されたテーブルのすべてのタブレットの情報をクエリします。次の例は、返される情報から1つのタブレットの情報を抜粋しています。

    ```plain
        mysql> show tablet from example_db.test_show_tablet\G
        *************************** 1. row ***************************
                TabletId: 9588955
```yaml
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

```yaml
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
              State: 正常
LstConsistencyCheckTime: NULL
      CheckVersion: -1
  CheckVersionHash: 0
      VersionCount: 1
          PathHash: 0
           MetaUrl: http://172.26.92.141:8038/api/meta/header/9588955
    CompactionStatus: http://172.26.92.141:8038/api/compaction/show?tablet_id=9588955
```