---
displayed_sidebar: "Japanese"
---

# データのバックアップとリストア

このトピックでは、StarRocksでデータをバックアップおよびリストアする方法、またはデータを新しいStarRocksクラスタに移行する方法について説明します。

StarRocksは、データをスナップショットとしてリモートストレージシステムにバックアップし、そのデータを任意のStarRocksクラスタにリストアすることができます。

StarRocksは、次のリモートストレージシステムをサポートしています。

- Apache™ Hadoop®（HDFS）クラスタ
- AWS S3
- Google GCS

## データのバックアップ

StarRocksは、データベース、テーブル、またはパーティションの単位でのFULLバックアップをサポートしています。

テーブルに大量のデータが格納されている場合、ジョブの失敗時のリトライのコストを削減するために、パーティションごとにデータをバックアップおよびリストアすることをおすすめします。定期的に増分データをバックアップする必要がある場合は、テーブルの[動的パーティショニング](../table_design/dynamic_partitioning.md)プラン（一定の時間間隔など）を立てて、毎回新しいパーティションのみをバックアップすることができます。

### リポジトリの作成

データをバックアップする前に、リモートストレージシステムにデータスナップショットを保存するためのリポジトリを作成する必要があります。StarRocksクラスタには複数のリポジトリを作成することができます。詳細な手順については、[CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md)を参照してください。

- HDFSでリポジトリを作成する

次の例では、HDFSクラスタに`test_repo`という名前のリポジトリを作成しています。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "hdfs://<hdfs_host>:<hdfs_port>/repo_dir/backup"
PROPERTIES(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

- AWS S3でリポジトリを作成する

  AWS S3へのアクセスのための認証方法として、IAMユーザーベースの資格情報（アクセスキーとシークレットキー）、インスタンスプロファイル、またはアサムドロールを選択することができます。

  - 次の例では、IAMユーザーベースの資格情報を使用して、AWS S3バケット`bucket_s3`に`test_repo`という名前のリポジトリを作成しています。

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

  - 次の例では、インスタンスプロファイルを使用して、AWS S3バケット`bucket_s3`に`test_repo`という名前のリポジトリを作成しています。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 次の例では、アサムドロールを使用して、AWS S3バケット`bucket_s3`に`test_repo`という名前のリポジトリを作成しています。

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
> StarRocksは、S3Aプロトコルに基づいてAWS S3でのリポジトリの作成のみをサポートしています。したがって、AWS S3でリポジトリを作成する場合は、`ON LOCATION`でリポジトリの場所として渡すS3 URIの`s3://`を`s3a://`に置き換える必要があります。

- Google GCSでリポジトリを作成する

次の例では、Google GCSバケット`bucket_gcs`に`test_repo`という名前のリポジトリを作成しています。

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
> StarRocksは、S3Aプロトコルに基づいてGoogle GCSでのリポジトリの作成のみをサポートしています。したがって、Google GCSでリポジトリを作成する場合は、`ON LOCATION`でリポジトリの場所として渡すGCS URIのプレフィックスを`s3a://`に置き換える必要があります。

リポジトリが作成されたら、[SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md)を使用してリポジトリを確認することができます。データをリストアした後、[DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md)を使用してStarRocksでリポジトリを削除することができます。ただし、リモートストレージシステムにバックアップされたデータスナップショットはStarRocksを介して削除することはできません。リモートストレージシステムで手動で削除する必要があります。

### データスナップショットのバックアップ

リポジトリが作成されたら、データスナップショットを作成し、リモートリポジトリにバックアップする必要があります。詳細な手順については、[BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md)を参照してください。

次の例では、データベース`sr_hub`のテーブル`sr_member`に対してデータスナップショット`sr_member_backup`を作成し、リポジトリ`test_repo`にバックアップしています。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

BACKUPは非同期の操作です。[SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md)を使用してBACKUPジョブのステータスを確認したり、[CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md)を使用してBACKUPジョブをキャンセルしたりすることができます。

## データのリストアまたは移行

リモートストレージシステムにバックアップされたデータスナップショットを現在のStarRocksクラスタまたは他のStarRocksクラスタにリストアまたは移行してデータを復元することができます。

### （オプション）新しいクラスタでリポジトリを作成する

データを別のStarRocksクラスタに移行する場合は、新しいクラスタで同じ**リポジトリ名**と**場所**を持つリポジトリを作成する必要があります。そうしないと、以前にバックアップされたデータスナップショットを表示することができません。詳細については、「[リポジトリの作成](#リポジトリの作成)」を参照してください。

### スナップショットの確認

データをリストアする前に、指定したリポジトリ内のスナップショットを[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md)を使用して確認することができます。

次の例では、`test_repo`内のスナップショット情報を確認しています。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| Snapshot         | Timestamp               | Status |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 row in set (1.16 sec)
```

### スナップショットを使用してデータをリストアする

[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md)ステートメントを使用して、リモートストレージシステムにバックアップされたデータスナップショットを現在のStarRocksクラスタまたは他のStarRocksクラスタにリストアすることができます。

次の例では、`test_repo`のデータスナップショット`sr_member_backup`をテーブル`sr_member`にリストアしています。データレプリカは1つだけリストアされます。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

RESTOREは非同期の操作です。[SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md)を使用してRESTOREジョブのステータスを確認したり、[CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md)を使用してRESTOREジョブをキャンセルしたりすることができます。

## BACKUPまたはRESTOREジョブの設定

BACKUPまたはRESTOREジョブのパフォーマンスを最適化するために、BE構成ファイル**be.conf**の次の設定項目を変更することができます。

| 設定項目                 | 説明                                                                             |
| ----------------------- | -------------------------------------------------------------------------------- |
| upload_worker_count     | BEノード上のBACKUPジョブのアップロードタスクのスレッドの最大数。デフォルト：`1`。この設定項目の値を増やすと、アップロードタスクの並行性が向上します。 |
| download_worker_count   | BEノード上のRESTOREジョブのダウンロードタスクのスレッドの最大数。デフォルト：`1`。この設定項目の値を増やすと、ダウンロードタスクの並行性が向上します。 |
| max_download_speed_kbps | BEノード上のダウンロード速度の上限。デフォルト：`50000`。単位：KB/s。通常、RESTOREジョブのダウンロードタスクの速度はデフォルト値を超えません。この設定がRESTOREジョブのパフォーマンスを制限している場合は、帯域幅に応じて値を増やすことができます。|

## 使用上の注意

- データのバックアップまたはリストアを行うには、ADMIN権限を持つユーザーのみが許可されます。
- 各データベースでは、実行中のBACKUPまたはRESTOREジョブは1つだけ許可されます。それ以外の場合、StarRocksはエラーを返します。
- BACKUPおよびRESTOREジョブは、StarRocksクラスタの多くのリソースを占有するため、StarRocksクラスタの負荷が重くない状態でデータをバックアップおよびリストアすることができます。
- StarRocksは、データバックアップのためのデータ圧縮アルゴリズムの指定をサポートしていません。
- スナップショット生成時にロードされたデータはスナップショットに含まれないため、スナップショット生成後かつRESTOREジョブが完了する前に古いクラスタにデータをロードした場合、データをリストアしたクラスタにもデータをロードする必要があります。データ移行が完了した後、データとサービスの正確性を確認した後、一定期間両方のクラスタにデータを並行してロードし、その後アプリケーションを新しいクラスタに移行することをおすすめします。
- RESTOREジョブが完了するまで、リストアするテーブルを操作することはできません。
- プライマリキーテーブルは、v2.5より前のStarRocksクラスタにリストアすることはできません。
- リストアするテーブルを新しいクラスタで事前に作成する必要はありません。RESTOREジョブが自動的にテーブルを作成します。
- リストアするテーブルと同じ名前の既存のテーブルが存在する場合、StarRocksは既存のテーブルのスキーマがリストアするテーブルのスキーマと一致するかどうかを最初にチェックします。スキーマが一致する場合、StarRocksはスナップショットのデータで既存のテーブルを上書きします。スキーマが一致しない場合、RESTOREジョブは失敗します。リストアするテーブルの名前をキーワード`AS`を使用して変更するか、データをリストアする前に既存のテーブルを削除することができます。
- RESTOREジョブが既存のデータベース、テーブル、またはパーティションを上書きする場合、ジョブがCOMMITフェーズに入った後は上書きされたデータをリストアすることはできません。RESTOREジョブがこの時点で失敗またはキャンセルされた場合、データは破損し、アクセスできなくなる可能性があります。この場合、RESTORE操作を再実行してジョブが完了するのを待つしかありません。したがって、現在のデータがもはや使用されていないことが確実でない限り、上書きによるデータのリストアは行わないことをおすすめします。上書き操作は、スナップショットと既存のデータベース、テーブル、またはパーティションのメタデータの整合性を最初にチェックします。整合性の不一致が検出された場合、RESTORE操作は実行できません。
- BACKUPまたはRESTOREジョブ中、StarRocksは自動的に[Synchronous materialized view](../using_starrocks/Materialized_view-single_table.md)をバックアップまたはリストアします。これにより、データのリストア後もクエリの高速化や書き換えが可能です。現在、StarRocksはビューと[Asynchronous materialized views](../using_starrocks/Materialized_view.md)のバックアップをサポートしていません。物理テーブルのみをバックアップすることができますが、これはクエリの高速化やクエリの書き換えには使用できません。
- 現在、StarRocksは、ユーザーアカウント、権限、リソースグループに関連する設定データのバックアップをサポートしていません。
- 現在、StarRocksは、テーブル間のColocate Joinの関係のバックアップとリストアをサポートしていません。
