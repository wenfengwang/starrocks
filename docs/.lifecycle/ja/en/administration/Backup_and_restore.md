---
displayed_sidebar: English
---

# データのバックアップと復元

このトピックでは、StarRocks でのデータのバックアップおよび復元方法、または新しい StarRocks クラスターへのデータ移行方法について説明します。

StarRocks は、データをスナップショットとしてリモートストレージシステムにバックアップし、任意の StarRocks クラスターにデータを復元することをサポートしています。

StarRocks は、以下のリモートストレージシステムをサポートしています：

- Apache™ Hadoop® (HDFS) クラスター
- AWS S3
- Google GCS

## データのバックアップ

StarRocks は、データベース、テーブル、またはパーティションの粒度レベルでの FULL バックアップをサポートしています。

テーブルに大量のデータを格納している場合、ジョブの失敗時のリトライコストを削減するために、パーティションごとにデータをバックアップおよび復元することを推奨します。定期的に増分データをバックアップする必要がある場合は、テーブルに対して[動的パーティショニング](../table_design/dynamic_partitioning.md)計画（例えば、一定の時間間隔で）を立て、新しいパーティションのみを毎回バックアップすることができます。

### リポジトリの作成

データをバックアップする前に、リモートストレージシステムにデータスナップショットを保存するためのリポジトリを作成する必要があります。StarRocks クラスターでは複数のリポジトリを作成できます。詳細な手順については、[CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md) を参照してください。

- HDFS でリポジトリを作成する

次の例では、HDFS クラスターに `test_repo` という名前のリポジトリを作成します。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "hdfs://<hdfs_host>:<hdfs_port>/repo_dir/backup"
PROPERTIES(
    "username" = "<hdfs_username>",
    "password" = "<hdfs_password>"
);
```

- AWS S3 でリポジトリを作成する

  AWS S3 へのアクセスには、IAM ユーザーベースの認証情報（アクセスキーとシークレットキー）、インスタンスプロファイル、またはアサインされたロールを認証方法として選択できます。

  - 次の例では、IAM ユーザーベースの認証情報を使用して AWS S3 バケット `bucket_s3` に `test_repo` という名前のリポジトリを作成します。

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

  - 次の例では、インスタンスプロファイルを使用して AWS S3 バケット `bucket_s3` に `test_repo` という名前のリポジトリを作成します。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 次の例では、アサインされたロールを使用して AWS S3 バケット `bucket_s3` に `test_repo` という名前のリポジトリを作成します。

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
> StarRocks は S3A プロトコルに基づいてのみ AWS S3 でリポジトリを作成することをサポートしています。したがって、AWS S3 でリポジトリを作成する際には、`ON LOCATION` で指定する S3 URI の `s3://` を `s3a://` に置き換える必要があります。

- Google GCS でリポジトリを作成する

次の例では、Google GCS バケット `bucket_gcs` に `test_repo` という名前のリポジトリを作成します。

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
> StarRocks は S3A プロトコルに基づいてのみ Google GCS でリポジトリを作成することをサポートしています。そのため、Google GCS でリポジトリを作成する際には、`ON LOCATION` で指定する GCS URI のプレフィックスを `s3a://` に置き換える必要があります。

リポジトリが作成されたら、[SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md) でリポジトリを確認できます。データを復元した後、[DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md) を使用して StarRocks のリポジトリを削除できます。ただし、リモートストレージシステムにバックアップされたデータスナップショットは StarRocks を通じて削除することはできません。それらはリモートストレージシステムで手動で削除する必要があります。

### データスナップショットのバックアップ

リポジトリが作成されたら、データスナップショットを作成し、リモートリポジトリにバックアップする必要があります。詳細な手順については、[BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md) を参照してください。

次の例では、データベース `sr_hub` のテーブル `sr_member` のデータスナップショット `sr_member_backup` を作成し、リポジトリ `test_repo` にバックアップします。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

BACKUP は非同期操作です。[SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md) を使用して BACKUP ジョブのステータスを確認するか、[CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md) を使用して BACKUP ジョブをキャンセルできます。

## データの復元または移行

リモートストレージシステムにバックアップされたデータスナップショットを現在の StarRocks クラスターまたは他の StarRocks クラスターに復元して、データを復元または移行することができます。

### （オプション）新しいクラスターにリポジトリを作成する

別の StarRocks クラスターにデータを移行するには、新しいクラスターに同じ**リポジトリ名**と**場所**を持つリポジトリを作成する必要があります。そうしないと、以前にバックアップしたデータスナップショットを表示できません。詳細については、[リポジトリの作成](#create-a-repository) を参照してください。

### スナップショットの確認

データを復元する前に、[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md) を使用して指定されたリポジトリ内のスナップショットを確認できます。

次の例では、リポジトリ `test_repo` のスナップショット情報を確認します。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| Snapshot         | Timestamp               | Status |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 row in set (1.16 sec)
```

### スナップショットを使用したデータの復元

[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) ステートメントを使用して、リモートストレージシステム内のデータスナップショットを現在の StarRocks クラスターまたは他の StarRocks クラスターに復元できます。

次の例では、テーブル`sr_member`上のデータスナップショット`sr_member_backup`を`test_repo`から復元します。復元されるのは1つのデータレプリカのみです。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

RESTOREは非同期操作です。[SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md)を使用してRESTOREジョブのステータスを確認するか、[CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md)を使用してRESTOREジョブをキャンセルできます。

## BACKUPまたはRESTOREジョブの設定

BACKUPまたはRESTOREジョブのパフォーマンスを最適化するには、BE設定ファイル**be.conf**の以下の設定項目を変更します：

| 設定項目                | 説明                                                                                   |
| ----------------------- | -------------------------------------------------------------------------------------- |
| upload_worker_count     | BEノード上のBACKUPジョブのアップロードタスク用スレッドの最大数。デフォルト：`1`。この設定項目の値を増やして、アップロードタスクの並行性を高めます。 |
| download_worker_count   | BEノード上のRESTOREジョブのダウンロードタスク用スレッドの最大数。デフォルト：`1`。この設定項目の値を増やして、ダウンロードタスクの並行性を高めます。 |
| max_download_speed_kbps | BEノードのダウンロード速度の上限。デフォルト：`50000`。単位：KB/s。通常、RESTOREジョブのダウンロードタスクの速度はデフォルト値を超えません。この設定によってRESTOREジョブのパフォーマンスが制限されている場合は、帯域幅に応じて増加させることができます。|

## マテリアライズドビューのBACKUPとRESTORE

テーブルのBACKUPまたはRESTOREジョブ中、StarRocksは自動的にその[同期マテリアライズドビュー](../using_starrocks/Materialized_view-single_table.md)をバックアップまたは復元します。

v3.2.0から、StarRocksはデータベースをバックアップおよび復元する際に、その中にある[非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md)もバックアップおよび復元することをサポートしています。

データベースのBACKUPおよびRESTORE中、StarRocksは以下のように操作します：

- **BACKUP**

1. データベースを走査して、すべてのテーブルと非同期マテリアライズドビューに関する情報を収集します。
2. BACKUPおよびRESTOREキュー内のテーブルの順序を調整し、マテリアライズドビューのベーステーブルがマテリアライズドビューの前に配置されるようにします：
   - ベーステーブルが現在のデータベースに存在する場合、StarRocksはそのテーブルをキューに追加します。
   - ベーステーブルが現在のデータベースに存在しない場合、StarRocksは警告ログを出力し、プロセスをブロックせずにBACKUP操作を続行します。
3. キューの順序に従ってBACKUPタスクを実行します。

- **RESTORE**

1. BACKUPおよびRESTOREキューの順序に従って、テーブルとマテリアライズドビューを復元します。
2. マテリアライズドビューとそのベーステーブル間の依存関係を再構築し、リフレッシュタスクのスケジュールを再提出します。

RESTOREプロセス中に発生したエラーはプロセスをブロックしません。

RESTORE後、[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md)を使用してマテリアライズドビューのステータスを確認できます。

- マテリアライズドビューがアクティブな場合、直接使用できます。
- マテリアライズドビューが非アクティブな場合、その原因はベーステーブルが復元されていない可能性があります。すべてのベーステーブルが復元された後、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md)を使用してマテリアライズドビューを再アクティブ化できます。

## 使用上の注意点

- ADMIN権限を持つユーザーのみがデータをバックアップまたは復元できます。
- 各データベースでは、一度に1つのBACKUPまたはRESTOREジョブのみが許可されます。それ以外の場合、StarRocksはエラーを返します。
- BACKUPおよびRESTOREジョブはStarRocksクラスタの多くのリソースを占有するため、クラスタの負荷が低い時にデータをバックアップおよび復元することをお勧めします。
- StarRocksはデータバックアップ用のデータ圧縮アルゴリズムの指定をサポートしていません。
- データはスナップショットとしてバックアップされるため、スナップショット生成時にロードされたデータはスナップショットに含まれません。そのため、スナップショット生成後およびRESTOREジョブ完了前に旧クラスタにデータをロードした場合、復元先クラスタにもデータをロードする必要があります。データ移行完了後、一定期間は両クラスタに並行してデータをロードし、データとサービスの正確性を確認した後、アプリケーションを新クラスタに移行することを推奨します。
- RESTOREジョブが完了するまで、復元対象のテーブルを操作することはできません。
- Primary Keyテーブルは、v2.5より前のStarRocksクラスタには復元できません。
- 復元する前に新クラスタで復元対象のテーブルを作成する必要はありません。RESTOREジョブが自動的にテーブルを作成します。
- 復元対象のテーブルと名前が重複する既存のテーブルがある場合、StarRocksはまず既存のテーブルのスキーマが復元対象のテーブルのスキーマと一致するかどうかをチェックします。スキーマが一致する場合、StarRocksは既存のテーブルをスナップショットのデータで上書きします。スキーマが一致しない場合、RESTOREジョブは失敗します。キーワード`AS`を使用して復元対象のテーブルの名前を変更するか、データを復元する前に既存のテーブルを削除できます。
- RESTOREジョブが既存のデータベース、テーブル、またはパーティションを上書きする場合、ジョブがCOMMITフェーズに入った後は上書きされたデータを復元することはできません。この時点でRESTOREジョブが失敗またはキャンセルされた場合、データは破損しアクセス不可能になる可能性があります。この場合、RESTORE操作を再度実行し、ジョブが完了するのを待つしかありません。そのため、現在のデータがもはや使用されないことが確実でない限り、上書きによるデータの復元は行わないことを推奨します。上書き操作では、まずスナップショットと既存のデータベース、テーブル、またはパーティションとの間のメタデータの整合性がチェックされます。不整合が検出された場合、RESTORE操作は実行できません。
- 現在、StarRocksは論理ビューのバックアップと復元をサポートしていません。
- 現在、StarRocksはユーザーアカウント、権限、リソースグループに関連する設定データのバックアップ及び復元をサポートしていません。
- 現在、StarRocksはテーブル間のColocate Join関係のバックアップ及び復元をサポートしていません。
