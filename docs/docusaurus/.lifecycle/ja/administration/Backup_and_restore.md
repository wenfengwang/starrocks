---
displayed_sidebar: "Japanese"
---

# データのバックアップとリストア

このトピックでは、StarRocksにおいてデータのバックアップとリストアを行い、また新しいStarRocksクラスタにデータを移行する手順について説明します。

StarRocksはデータをリモートストレージシステムにスナップショットとしてバックアップし、それらのデータを任意のStarRocksクラスタにリストアすることができます。

StarRocksは以下のリモートストレージシステムをサポートしています：

- Apache™ Hadoop®（HDFS）クラスタ
- AWS S3
- Google GCS

## データのバックアップ

StarRocksでは、データベース、テーブル、またはパーティションのレベルでのFULLバックアップをサポートしています。

テーブルに多くのデータが格納されている場合は、ジョブの失敗に備えて、パーティションごとにデータのバックアップとリストアをお勧めします。定期的に増分データのバックアップが必要な場合は、[動的パーティショニング](../table_design/dynamic_partitioning.md) プラン（例えば、特定の時間間隔で）をテーブルごとに考え、毎回新しいパーティションのみをバックアップすることができます。

### リポジトリの作成

データのバックアップを行う前に、リモートストレージシステムにデータスナップショットを保存するためのリポジトリを作成する必要があります。StarRocksクラスタには複数のリポジトリを作成することができます。詳細な手順については、[CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md) を参照してください。

- HDFSにリポジトリを作成

以下の例では、HDFSクラスタ内に `test_repo` という名前のリポジトリを作成しています。

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

  AWS S3をアクセスするための認証方法として、IAMユーザーベースの資格情報（Access KeyとSecret Key）、インスタンスプロファイル、またはAssumed Roleを選択することができます。

  - 以下の例では、IAMユーザーベースの資格情報を使用して `bucket_s3` というAWS S3バケット内に `test_repo` という名前のリポジトリを作成しています。

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

  - 以下の例では、インスタンスプロファイルを使用して `bucket_s3` というAWS S3バケット内に `test_repo` という名前のリポジトリを作成しています。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 以下の例では、Assumed Roleを使用して `bucket_s3` というAWS S3バケット内に `test_repo` という名前のリポジトリを作成しています。

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

> **注記**
>
> StarRocksはAWS S3においてS3Aプロトコルにのみリポジトリの作成をサポートしています。したがって、AWS S3にリポジトリを作成する際には、`ON LOCATION` 内でリポジトリの場所として渡すS3 URI内の `s3://` を `s3a://` に置き換える必要があります。

- Google GCSにリポジトリを作成

以下の例では、Google GCSバケット内に `test_repo` という名前のリポジトリを作成しています。

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

> **注記**
>
> StarRocksはGoogle GCSにおいてS3Aプロトコルにのみリポジトリの作成をサポートしています。したがって、Google GCSにリポジトリを作成する際には、`ON LOCATION` 内で渡すGCS URI内の接頭辞を `s3a://` に置き換える必要があります。

リポジトリが作成された後は、[SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md) を使用してリポジトリを確認することができます。データのリストア後は、[DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md) を使用してStarRocks内のリポジトリを削除することができます。ただし、リモートストレージシステムにバックアップされたデータスナップショットはStarRocksを介して削除することはできません。リモートストレージシステム内で手動で削除する必要があります。

### データスナップショットのバックアップ

リポジトリが作成された後は、データスナップショットを作成し、リモートリポジトリにバックアップする必要があります。詳細な手順については、[BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md) を参照してください。

以下の例では、データベース `sr_hub` 内のテーブル `sr_member` のデータスナップショット `sr_member_backup` を作成し、`test_repo` というリポジトリにバックアップしています。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

BACKUPは非同期操作です。BACKUPジョブの状態は[SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md) を使用して確認することができ、また、[CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md) を使用してBACKUPジョブをキャンセルすることができます。

## データのリストアまたは移行

リモートストレージシステムにバックアップされたデータスナップショットを現在のStarRocksクラスタ、または他のStarRocksクラスタにリストアまたは移行することができます。

### （オプション）新しいクラスタでのリポジトリの作成

データを別のStarRocksクラスタに移行する場合、新しいクラスタで **リポジトリの名前** と **場所** が同じである必要があります。そうでないと、以前にバックアップされたデータスナップショットを表示することができません。詳細については、[リポジトリの作成](#リポジトリの作成) を参照してください。

### スナップショットの確認

データをリストアする前に、特定のリポジトリにおけるスナップショットを[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md) を使用して確認することができます。

以下の例では、`test_repo` におけるスナップショット情報を確認しています。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| Snapshot         | Timestamp               | Status |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 row in set (1.16 sec)
```

### スナップショットを使用したデータのリストア

[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) ステートメントを使用して、リモートストレージシステムにおけるデータスナップショットを現在のStarRocksクラスタ、または他のStarRocksクラスタにリストアすることができます。

以下の例では、`test_repo` の `sr_member_backup` というデータスナップショットを、`sr_member` のテーブルにリストアしています。データのレプリカは1つのみリストアされます。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

RESTOREは非同期操作です。RESTOREジョブの状態は[SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md) を使用して確認することができ、また、[CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md) を使用してRESTOREジョブをキャンセルすることができます。

## バックアップまたはリストアジョブの構成

BE設定ファイル **be.conf** において、以下の設定項目を変更することで、バックアップまたはリストアジョブのパフォーマンスを最適化することができます：

| 設定項目                   | 説明                                                                             |
| ----------------------- | -------------------------------------------------------------------------------- |
| upload_worker_count     | BEノードにおけるバックアップジョブのアップロードタスクの最大スレッド数。デフォルト：`1`。この設定項目の値を増やすことで、アップロードタスクの並列処理を増やすことができます。 |
| download_worker_count   | BEノードにおけるリストアジョブのダウンロードタスクの最大スレッド数。デフォルト：`1`。この設定項目の値を増やすことで、ダウンロードタスクの並列処理を増やすことができます。 |
| max_download_speed_kbps | BEノードにおけるダウンロード速度の上限。デフォルト：`50000`。単位：KB/s。通常、リストアジョブのダウンロードタスクの速度はデフォルト値を超えることはありません。この設定がリストアジョブのパフォーマンスを制限している場合は、帯域幅に合わせて増やすことができます。|

## マテリアライズドビューのバックアップとリストア

テーブルのバックアップまたはリストアジョブ実行中には、StarRocksが自動的にその[同期マテリアライズドビュー](../using_starrocks/Materialized_view-single_table.md)をバックアップまたはリストアします。

v3.2.0から、StarRocksは、バックアップと復元データベースで[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)をサポートしています。

BACKUPおよびRESTORE実行時には、StarRocksは以下のように実行します。

- **BACKUP**

1. データベースをトラバースしてすべてのテーブルと非同期マテリアライズドビューに関する情報を収集します。
2. BACKUPおよびRESTOREキュー内のテーブルの順序を調整し、マテリアライズドビューの基底テーブルがマテリアライズドビューよりも前に位置するようにします。
   - 基底テーブルが現在のデータベースに存在する場合、StarRocksはそのテーブルをキューに追加します。
   - 基底テーブルが現在のデータベースに存在しない場合、StarRocksは警告ログを出力し、プロセスをブロックせずにBACKUP操作を続行します。
3. キューの順序でBACKUPタスクを実行します。

- **RESTORE**

1. BACKUPおよびRESTOREキューの順序でテーブルとマテリアライズドビューを復元します。
2. マテリアライズドビューとその基底テーブル間の依存関係を再構築し、リフレッシュタスクスケジュールを再送信します。

RESTOREプロセス中にエラーが発生してもプロセスはブロックされません。

RESTORE後に、[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用してマテリアライズドビューの状態をチェックできます。

- マテリアライズドビューがアクティブであれば、直接使用できます。
- マテリアライズドビューが非アクティブであれば、基底テーブルが復元されていない可能性があります。すべての基底テーブルが復元された後に、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を使用してマテリアライズドビューを再アクティブ化できます。

## 使用上の注意

- ADMIN権限を持つユーザーのみがデータのバックアップまたは復元を行うことができます。
- 各データベースにつき、同時に実行されるBACKUPまたはRESTOREジョブは1つだけです。それ以外の場合、StarRocksはエラーを返します。
- BACKUPおよびRESTOREジョブは、StarRocksクラスタの多くのリソースを占有するため、StarRocksクラスタの負荷が重くない状態でデータのバックアップおよび復元を行うことができます。
- StarRocksは、データのバックアップのためのデータ圧縮アルゴリズムの指定をサポートしていません。
- データはスナップショットとしてバックアップされるため、スナップショット生成時にロードされたデータは含まれません。したがって、スナップショット生成後、RESTOREジョブが完了する前に古いクラスタにデータをロードした場合、そのデータを復元先クラスタにもロードする必要があります。データ移行が完了した後、一定期間両方のクラスタにデータを並行してロードし、データとサービスの正確性を確認した後に、アプリケーションを新しいクラスタに移行することをお勧めします。
- RESTOREジョブが完了するまで、復元するテーブルを操作することはできません。
- プライマリキーのテーブルはv2.5よりも古いStarRocksクラスタには復元できません。
- 新しいクラスタ内に復元するテーブルを作成する必要はありません。RESTOREジョブが自動的に作成します。
- 復元するテーブルと同じ名前の既存のテーブルがある場合、StarRocksはまず既存のテーブルのスキーマが復元するテーブルのスキーマと一致するかどうかをチェックします。スキーマが一致する場合、StarRocksはスナップショット内のデータで既存のテーブルを上書きします。スキーマが一致しない場合、RESTOREジョブは失敗します。復元するテーブルの名前を`AS`を使用してリネームするか、データを復元する前に既存のテーブルを削除することができます。
- RESTOREジョブが既存のデータベース、テーブル、またはパーティションを上書きする場合、ジョブがCOMMITフェーズに入った後は上書きされたデータを復元することはできません。RESTOREジョブがこの時点で失敗またはキャンセルされた場合、データが破損しアクセスできなくなる可能性があります。この場合、データを再生し、ジョブが完了するのを待つことしかできません。したがって、現在のデータが使用されていないことを確認できる場合を除き、上書きしてデータを復元しないことをお勧めします。上書き操作は、最初にスナップショットと既存のデータベース、テーブル、またはパーティションのメタデータの整合性をチェックします。不整合が検出された場合、RESTORE操作は実行できません。
- 現在、StarRocksは論理ビューのバックアップおよび復元をサポートしていません。
- 現在、StarRocksは、ユーザーアカウント、権限、およびリソースグループに関連する構成データのバックアップおよび復元をサポートしていません。
- 現在、StarRocksはテーブル間のColocate Join関係のバックアップおよび復元をサポートしていません。