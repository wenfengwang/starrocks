---
displayed_sidebar: "Japanese"
---

# データのバックアップとリストア

このトピックでは、StarRocksでデータのバックアップとリストアを行う方法、またはデータを新しいStarRocksクラスタに移行する方法について説明します。

StarRocksでは、データをリモートストレージシステムにスナップショットとしてバックアップし、そのデータを任意のStarRocksクラスタにリストアすることができます。

StarRocksは、次のリモートストレージシステムをサポートしています:

- Apache™ Hadoop® (HDFS) クラスタ
- AWS S3
- Google GCS

## データのバックアップ

StarRocksでは、データベース、テーブル、またはパーティションの単位で完全バックアップをサポートしています。

テーブルに大量のデータを保存している場合は、パーティションごとにデータをバックアップおよびリストアすることをお勧めします。これにより、ジョブの失敗時にリトライのコストを削減できます。定期的に増分データをバックアップする必要がある場合は、テーブルごとに特定の時間間隔で[動的パーティション](../table_design/dynamic_partitioning.md)の計画を立て、毎回新しいパーティションのみをバックアップできます。

### リポジトリの作成

データをバックアップする前に、リモートストレージシステムにデータスナップショットを保存するためのリポジトリを作成する必要があります。StarRocksクラスタに複数のリポジトリを作成できます。詳しい手順については、[CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md)を参照してください。

- HDFSにリポジトリを作成

次の例では、HDFSクラスタに名前が`test_repo`のリポジトリを作成します。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "hdfs://<hdfs_host>:<hdfs_port>/repo_dir/backup"
PROPERTIES(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

- AWS S3にリポジトリを作成

  AWS S3を利用したIAMユーザーベースの資格情報 (アクセスキーとシークレットキー)、インスタンスプロファイル、または仮定される役割を資格情報の方法として選択できます。

  - 次の例では、IAMユーザーベースの資格情報を資格情報の方法として使用して、AWS S3バケット`bucket_s3`に名前が`test_repo`のリポジトリを作成します。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.access_key" = "XXXXXXXXXXXXXXXXX",
      "aws.s3.secret_key" = "yyyyyyyyyyyyyyyyyyyyyyyy",
      "aws.s3.endpoint" = "s3.us-east-1.amazonaws.com"
  );
  ```

  - 次の例では、インスタンスプロファイルを資格情報の方法として使用して、AWS S3バケット`bucket_s3`に名前が`test_repo`のリポジトリを作成します。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 次の例では、仮定される役割を資格情報の方法として使用して、AWS S3バケット`bucket_s3`に名前が`test_repo`のリポジトリを作成します。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::xxxxxxxxxx:role/yyyyyyyy",
      "aws.s3.region" = "us-east-1"
  );
  ```

> **注意**
>
> StarRocksは、AWS S3内でのリポジトリの作成をS3Aプロトコルにのみ対応しています。したがって、AWS S3内でのリポジトリを作成する際には、`ON LOCATION`で渡すS3 URI内の`s3://`を`s3a://`に置換する必要があります。

- Google GCSにリポジトリを作成

次の例では、Google GCSバケット`bucket_gcs`に名前が`test_repo`のリポジトリを作成します。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "s3a://bucket_gcs/backup"
PROPERTIES(
    "fs.s3a.access.key" = "xxxxxxxxxxxxxxxxxxxx",
    "fs.s3a.secret.key" = "yyyyyyyyyyyyyyyyyyyy",
    "fs.s3a.endpoint" = "storage.googleapis.com"
);
```

> **注意**
>
> StarRocksは、Google GCS内でのリポジトリの作成をS3Aプロトコルにのみ対応しています。したがって、Google GCS内でのリポジトリを作成する際には、`ON LOCATION`で渡すGCS URI内のプレフィックスを`s3a://`に置換する必要があります。

リポジトリが作成されたら、[SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md)を使用してリポジトリを確認できます。データのリストア後、[DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md)を使用してStarRocksでリポジトリを削除できます。ただし、リモートストレージシステムにバックアップされたデータスナップショットはStarRocks経由では削除できません。リモートストレージシステムで手動で削除する必要があります。

### データスナップショットのバックアップ

リポジトリが作成された後、データスナップショットを作成し、リモートリポジトリにバックアップする必要があります。詳しい手順については、[BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md)を参照してください。

次の例では、データベース`sr_hub`のテーブル`sr_member`に対するデータスナップショット`sr_member_backup`を作成し、リポジトリ`test_repo`にバックアップします。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

BACKUPは非同期操作です。[SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md)を使用してBACKUPジョブのステータスを確認したり、[CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md)を使用してBACKUPジョブをキャンセルしたりできます。

## データのリストアまたは移行

リモートストレージシステムにバックアップされたデータスナップショットを現在のStarRocksクラスタまたは他のStarRocksクラスタにリストアまたは移行することができます。

### (オプション) 新しいクラスタでリポジトリを作成

データを別のStarRocksクラスタに移行する場合は、同じ**リポジトリ名**および**ロケーション**を持つリポジトリを新しいクラスタで作成する必要があります。そうしないと、以前にバックアップされたデータスナップショットを表示できなくなります。詳細については、[リポジトリの作成](#リポジトリの作成)を参照してください。

### スナップショットの確認

データをリストアする前に、[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md)を使用して指定したリポジトリ内のスナップショットを確認できます。

次の例では、`test_repo`のスナップショット情報を確認します。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| Snapshot         | Timestamp               | Status |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 row in set (1.16 sec)
```

### スナップショット経由でデータをリストア

リモートストレージシステムにバックアップされたデータスナップショットを現在のStarRocksクラスタまたは他のStarRocksクラスタにリストアするには、[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md)ステートメントを使用できます。

次の例では、`test_repo`内のデータスナップショット`sr_member_backup`をテーブル`sr_member`に対してリストアします。データは1つのレプリカのみをリストアします。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

RESTOREは非同期操作です。[SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md)を使用してRESTOREジョブのステータスを確認したり、[CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md)を使用してRESTOREジョブをキャンセルしたりできます。

## BACKUPまたはRESTOREジョブの設定

BE構成ファイル**be.conf**の次の設定項目を変更することで、BACKUPまたはRESTOREジョブのパフォーマンスを最適化できます:

| 設定項目                 | 説明                                                                             |
| ----------------------- | -------------------------------------------------------------------------------- |
| upload_worker_count     | BEノード上のBACKUPジョブのアップロードタスクの最大スレッド数。デフォルト: `1`。この設定項目の値を増やすと、アップロードタスクの並列処理を増やすことができます。 |
| download_worker_count   | BEノード上のRESTOREジョブのダウンロードタスクの最大スレッド数。デフォルト: `1`。この設定項目の値を増やすと、ダウンロードタスクの並列処理を増やすことができます。 |
| max_download_speed_kbps | BEノード上のダウンロード速度の上限。デフォルト: `50000`。単位: KB/s。通常、RESTOREジョブのダウンロードタスクの速度はデフォルト値を超えることはありません。この設定がRESTOREジョブのパフォーマンスを制限している場合は、帯域幅に応じて値を増やすことができます。|

## マテリアライズドビューのBACKUPおよびRESTORE

テーブルのBACKUPまたはRESTOREジョブ中、StarRocksは自動的にその[Synchronous materialized view](../using_starrocks/Materialized_view-single_table.md)をバックアップまたはリストアします。
```markdown
+ {R}
+ {R}
  + {R}
    + {R}
      - **BACKUP**
        1. Traverse the database to gather information on all tables and asynchronous materialized views.
        2. Adjust the order of tables in the BACKUP and RESTORE queue, ensuring that the base tables of materialized views are positioned before the materialized views:
          - If the base table exists in the current database, StarRocks adds the table to the queue.
          - If the base table does not exist in the current database, StarRocks prints a warning log and proceeds with the BACKUP operation without blocking the process.
        3. Execute the BACKUP task in the order of the queue.
      - **RESTORE**
        1. Restore the tables and materialized views in the order of the BACKUP and RESTORE queue.
        2. Re-build the dependency between materialized views and their base tables, and re-submit the refresh task schedule.
        
After RESTORE, you can check the status of the materialized view using [SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md).

- If the materialized view is active, it can be used directly.
- If the materialized view is inactive, it might be because its base tables are not restored. After all the base tables are restored, you can use [ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) to re-activate the materialized view.

## Usage notes

- Only users with the ADMIN privilege can back up or restore data.
- In each database, only one running BACKUP or RESTORE job is allowed each time. Otherwise, StarRocks returns an error.
- Because BACKUP and RESTORE jobs occupy many resources of your StarRocks cluster, you can back up and restore your data while your StarRocks cluster is not heavily loaded.
- StarRocks does not support specifying data compression algorithm for data backup.
- Because data is backed up as snapshots, the data loaded upon snapshot generation is not included in the snapshot. Therefore, if you load data into the old cluster after the snapshot is generated and before the RESTORE job is completed, you also need to load the data into the cluster that data is restored into. It is recommended that you load data into both clusters in parallel for a period of time after the data migration is complete, and then migrate your application to the new cluster after verifying the correctness of the data and services.
- Before the RESTORE job is completed, you cannot operate the table to be restored.
- Primary Key tables cannot be restored to a StarRocks cluster earlier than v2.5.
- You do not need to create the table to be restored in the new cluster before restoring it. The RESTORE job automatically creates it.
- If there is an existing table that has a duplicated name with the table to be restored, StarRocks first checks whether or not the schema of the existing table matches that of the table to be restored. If the schemas match, StarRocks overwrites the existing table with the data in the snapshot. If the schema does not match, the RESTORE job fails. You can either rename the table to be restored using the keyword `AS`, or delete the existing table before restoring data.
- If the RESTORE job overwrites an existing database, table, or partition, the overwritten data cannot be restored after the job enters the COMMIT phase. If the RESTORE job fails or is canceled at this point, the data may be corrupted and inaccessible. In this case, you can only perform the RESTORE operation again and wait for the job to complete. Therefore, we recommend that you do not restore data by overwriting unless you are sure that the current data is no longer used. The overwrite operation first checks metadata consistency between the snapshot and the existing database, table, or partition. If an inconsistency is detected, the RESTORE operation cannot be performed.
- Currently, StarRocks does not support backing up and restoring logical views.
- Currently, StarRocks does not support backing up and restoring the configuration data related to user accounts, privileges, and resource groups.
- Currently, StarRocks does not support backing up and restoring the Colocate Join relationship among tables.
```