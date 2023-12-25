---
displayed_sidebar: Chinese
---

# バックアップと復元

この文書では、StarRocks のデータのバックアップと復元、または新しい StarRocks クラスターへのデータ移行方法について説明します。

StarRocks は、データをスナップショットファイルとしてリモートストレージシステムにバックアップし、バックアップされたデータを任意の StarRocks クラスターにリモートストレージシステムから復元することをサポートしています。この機能を使用することで、定期的に StarRocks クラスターのデータのスナップショットバックアップを行ったり、異なる StarRocks クラスター間でデータを移行することができます。

StarRocks は、以下の外部ストレージシステムでデータのバックアップをサポートしています：

- Apache™ Hadoop® （HDFS）クラスター
- AWS S3
- Google GCS
- 阿里云 OSS
- 腾讯云 COS

## データのバックアップ

StarRocks は、データベース、テーブル、またはパーティションを単位として全量データのバックアップをサポートしています。

テーブルのデータ量が大きい場合は、パーティションごとにバックアップを実行することをお勧めします。これにより、失敗時のリトライコストを低減できます。定期的なデータバックアップが必要な場合は、テーブル作成時に[動的パーティショニング](../table_design/dynamic_partitioning.md)戦略を策定することをお勧めします。これにより、運用管理プロセスで新たに追加されたパーティションのデータのみを定期的にバックアップすることができます。

### リポジトリの作成

リポジトリは、リモートストレージシステムにバックアップファイルを保存するために使用されます。データをバックアップする前に、リモートストレージシステムのパスを基に StarRocks でリポジトリを作成する必要があります。同じクラスター内で複数のリポジトリを作成することができます。詳細な使用方法は [CREATE REPOSITORY](../sql-reference/sql-statements/data-definition/CREATE_REPOSITORY.md) を参照してください。

- HDFS クラスターでリポジトリを作成する

以下の例では、Apache™ Hadoop® クラスターで `test_repo` というリポジトリを作成しています。

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

  IAM ユーザーの資格情報（Access Key と Secret Key）、Instance Profile、または Assumed Role を使用して Amazon S3 にアクセスするためのセキュリティ資格情報として選択できます。

  - 以下の例では、IAM ユーザーの資格情報をセキュリティ資格情報として使用し、Amazon S3 のバケット `bucket_s3` で `test_repo` というリポジトリを作成しています。

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

  - 以下の例では、Instance Profile をセキュリティ資格情報として使用し、Amazon S3 のバケット `bucket_s3` で `test_repo` というリポジトリを作成しています。

  ```SQL
  CREATE REPOSITORY test_repo
  WITH BROKER
  ON LOCATION "s3a://bucket_s3/backup"
  PROPERTIES(
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-east-1"
  );
  ```

  - 以下の例では、Assumed Role をセキュリティ資格情報として使用し、Amazon S3 のバケット `bucket_s3` で `test_repo` というリポジトリを作成しています。

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

> **説明**
>
> StarRocks は AWS S3 でリポジトリを作成する際に S3A プロトコルのみをサポートしています。したがって、AWS S3 でリポジトリを作成する際には、`ON LOCATION` パラメーターで S3 URI の `s3://` を `s3a://` に置き換える必要があります。

- Google GCS でリポジトリを作成する

以下の例では、Google GCS のバケット `bucket_gcs` で `test_repo` というリポジトリを作成しています。

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

> **説明**
>
> StarRocks は Google GCS でリポジトリを作成する際に S3A プロトコルのみをサポートしています。したがって、Google GCS でリポジトリを作成する際には、`ON LOCATION` パラメーターで GCS URI のプレフィックスを `s3a://` に置き換える必要があります。

- 阿里云 OSS でリポジトリを作成する

以下の例では、阿里云 OSS のバケット `bucket_oss` で `test_repo` というリポジトリを作成しています。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "oss://bucket_oss/backup"
PROPERTIES(
    "fs.oss.accessKeyId" = "xxxxxxxxxxxxxxxxxxxxxxxxxx",
    "fs.oss.accessKeySecret" = "yyyyyyyyyyyyyyyyyyyy",
    "fs.oss.endpoint" = "oss-cn-zhangjiakou-internal.aliyuncs.com"
);
```

- 腾讯云 COS でリポジトリを作成する

以下の例では、腾讯云 COS のバケット `bucket_cos` で `test_repo` というリポジトリを作成しています。

```SQL
CREATE REPOSITORY test_repo
WITH BROKER
ON LOCATION "cosn://bucket_cos/backup"
PROPERTIES(
    "fs.cosn.userinfo.secretId" = "xxxxxxxxxxxxxxxxx",
    "fs.cosn.userinfo.secretKey" = "yyyyyyyyyyyyyyyy",
    "fs.cosn.bucket.endpoint_suffix" = "cos.ap-beijing.myqcloud.com"
);
```

リポジトリが作成された後、[SHOW REPOSITORIES](../sql-reference/sql-statements/data-manipulation/SHOW_REPOSITORIES.md) を通じて作成されたリポジトリを確認できます。データ復元が完了した後、[DROP REPOSITORY](../sql-reference/sql-statements/data-definition/DROP_REPOSITORY.md) ステートメントを通じて StarRocks 内のリポジトリを削除できます。ただし、リモートストレージシステムにバックアップされたスナップショットデータは現在 StarRocks を通じて直接削除することはできません。リモートストレージシステムにバックアップされたスナップショットパスは手動で削除する必要があります。

### データスナップショットのバックアップ

リポジトリを作成した後、[BACKUP](../sql-reference/sql-statements/data-definition/BACKUP.md) コマンドを使用してデータスナップショットを作成し、リモートリポジトリにバックアップすることができます。

以下の例では、データベース `sr_hub` のテーブル `sr_member` に対してデータスナップショット `sr_member_backup` を作成し、リポジトリ `test_repo` にバックアップしています。

```SQL
BACKUP SNAPSHOT sr_hub.sr_member_backup
TO test_repo
ON (sr_member);
```

データのバックアップは非同期操作です。[SHOW BACKUP](../sql-reference/sql-statements/data-manipulation/SHOW_BACKUP.md) ステートメントを使用してバックアップジョブの状態を確認するか、[CANCEL BACKUP](../sql-reference/sql-statements/data-definition/CANCEL_BACKUP.md) ステートメントを使用してバックアップジョブをキャンセルすることができます。

## データの復元または移行

リモートリポジトリにバックアップされたデータスナップショットを現在の StarRocks クラスターまたは他のクラスターに復元し、データの復元または移行を完了することができます。


### （オプション）新しいクラスターでリポジトリを作成する

他の StarRocks クラスターにデータを移行する場合は、新しいクラスターで同じ**リポジトリ名**と**アドレス**を使用してリポジトリを作成する必要があります。そうしないと、以前のバックアップデータスナップショットを表示できません。詳細は [リポジトリの作成](#リポジトリの作成) を参照してください。

### データベーススナップショットを表示する

復元または移行を開始する前に、[SHOW SNAPSHOT](../sql-reference/sql-statements/data-manipulation/SHOW_SNAPSHOT.md) を使用して特定のリポジトリに対応するデータスナップショット情報を表示できます。

以下は、リポジトリ `test_repo` のデータスナップショット情報を表示する例です。

```Plain
mysql> SHOW SNAPSHOT ON test_repo;
+------------------+-------------------------+--------+
| Snapshot         | Timestamp               | Status |
+------------------+-------------------------+--------+
| sr_member_backup | 2023-02-07-14-45-53-143 | OK     |
+------------------+-------------------------+--------+
1 row in set (1.16 sec)
```

### データスナップショットを復元する

[RESTORE](../sql-reference/sql-statements/data-definition/RESTORE.md) ステートメントを使用して、リモートリポジトリのデータスナップショットを現在または別の StarRocks クラスターに復元し、データを復元または移行します。

以下の例では、リポジトリ `test_repo` のデータスナップショット `sr_member_backup` をテーブル `sr_member` として復元し、データのコピーを1つだけ復元します。

```SQL
RESTORE SNAPSHOT sr_hub.sr_member_backup
FROM test_repo
ON (sr_member)
PROPERTIES (
    "backup_timestamp"="2023-02-07-14-45-53-143",
    "replication_num" = "1"
);
```

データの復元は非同期操作です。[SHOW RESTORE](../sql-reference/sql-statements/data-manipulation/SHOW_RESTORE.md) ステートメントを使用して復元ジョブの状態を確認するか、[CANCEL RESTORE](../sql-reference/sql-statements/data-definition/CANCEL_RESTORE.md) ステートメントを使用して復元ジョブをキャンセルできます。

## 関連するパラメーターの設定

BE の設定ファイル **be.conf** で以下の設定項目を変更することで、バックアップまたは復元ジョブを加速できます：

| 設定項目                   | 説明                                                                             |
| ----------------------- | -------------------------------------------------------------------------------- |
| upload_worker_count     | BE ノードのアップロードタスクの最大スレッド数で、バックアップジョブに使用されます。デフォルト値：`1`。この設定項目の値を増やすことでアップロードタスクの並列度を増やすことができます。|
| download_worker_count   | BE ノードのダウンロードタスクの最大スレッド数で、復元ジョブに使用されます。デフォルト値：`1`。この設定項目の値を増やすことでダウンロードタスクの並列度を増やすことができます。|
| max_download_speed_kbps | BE ノードのダウンロード速度の上限。デフォルト値：`50000`。単位：KB/s。通常、復元ジョブのダウンロード速度はデフォルト値を超えることはありません。この速度の上限が復元ジョブのパフォーマンスを制限している場合は、帯域幅の状況に応じて適宜増やすことができます。|

## マテリアライズドビューのバックアップと復元

テーブル（Table）データのバックアップまたは復元中に、StarRocks は自動的にその中の [同期マテリアライズドビュー](../using_starrocks/Materialized_view-single_table.md) をバックアップまたは復元します。

v3.2.0 から、StarRocks はデータベース（Database）のバックアップと復元時に、データベース内の [非同期マテリアライズドビュー](../using_starrocks/Materialized_view.md) をバックアップおよび復元することをサポートしています。

データベースのバックアップと復元中に、StarRocks は以下の操作を実行します：

- **バックアップ**

1. データベースをトラバースして、すべてのテーブルと非同期マテリアライズドビューの情報を収集します。
2. バックアップおよび復元キュー内のテーブルの順序を調整し、マテリアライズドビューのベーステーブルがマテリアライズドビューの前に来るようにします：
   - ベーステーブルが現在のデータベース内に存在する場合、StarRocks はテーブルをキューに追加します。
   - ベーステーブルが現在のデータベース内に存在しない場合、StarRocks は警告ログを出力し、プロセスをブロックすることなくバックアップ操作を続行します。
3. キューの順序に従ってバックアップタスクを実行します。

- **復元**

1. バックアップおよび復元キューの順序に従ってテーブルとマテリアライズドビューを復元します。
2. マテリアライズドビューとそのベーステーブル間の依存関係を再構築し、リフレッシュタスクのスケジュールを再提出します。

復元プロセス中に発生したエラーはプロセスをブロックしません。

復元後、[SHOW MATERIALIZED VIEWS](../sql-reference/sql-statements/data-manipulation/SHOW_MATERIALIZED_VIEW.md) を使用してマテリアライズドビューの状態を確認できます。

- マテリアライズドビューが Active 状態であれば、直接使用できます。
- マテリアライズドビューが Inactive 状態である場合は、そのベーステーブルがまだ復元されていない可能性があります。すべてのベーステーブルを復元した後、[ALTER MATERIALIZED VIEW](../sql-reference/sql-statements/data-definition/ALTER_MATERIALIZED_VIEW.md) を使用してマテリアライズドビューを再アクティブ化できます。

## 注意事項

- バックアップと復元機能は ADMIN 権限を持つユーザーのみが実行できます。
- 単一のデータベース内で、同時に実行できるバックアップまたは復元ジョブは1つだけです。そうでない場合、StarRocks はエラーを返します。
- バックアップと復元操作は一定のシステムリソースを消費するため、クラスターのオフピーク時に実行することをお勧めします。
- 現在、StarRocks はデータのバックアップ時に圧縮アルゴリズムを使用することはサポートしていません。
- データバックアップはスナップショットの形で行われるため、現在のデータスナップショットが生成された後にインポートされたデータはバックアップされません。したがって、スナップショットの生成から復元（移行）ジョブの完了までの期間にインポートされたデータは、クラスターに再インポートする必要があります。移行が完了した後、新旧のクラスターに並行して一定期間データをインポートし、データとビジネスの正確性を検証した後、ビジネスを新しいクラスターに移行することをお勧めします。
- 復元ジョブが完了する前に、復元されるテーブルは操作できません。
- Primary Key テーブルは v2.5 以前の StarRocks クラスターに復元することはできません。
- 復元ジョブを開始する前に新しいクラスターで復元されるテーブルを作成する必要はありません。復元ジョブはそのテーブルを自動的に作成します。
- 復元されるテーブルが既存のテーブルと同名の場合、StarRocks はまず既存のテーブルのスキーマを識別します。スキーマが同じであれば、StarRocks は既存のテーブルに上書きします。スキーマが異なる場合、復元ジョブは失敗します。`AS` キーワードを使用して復元されるテーブルの名前を変更するか、既存のテーブルを削除してから復元ジョブを再開始できます。
- 復元ジョブが上書き操作（既存のテーブルまたはパーティションにデータを復元することを指定する）である場合、復元ジョブの COMMIT フェーズから、現在のクラスター上で上書きされるデータは復元できなくなる可能性があります。この時点で復元ジョブが失敗したりキャンセルされたりすると、以前のデータが損傷し、アクセスできなくなる可能性があります。この場合、復元操作を再度実行し、ジョブが完了するのを待つしかありません。したがって、現在のデータが不要でない限り、必要がなければデータを上書きして復元することはお勧めしません。上書き操作は、スナップショットと既存のテーブルまたはパーティションのメタデータが同じであるかどうかをチェックします。これにはスキーマやロールアップなどの情報が含まれます。異なる場合は復元操作を実行できません。
- 現在のところ、StarRocks は論理ビューのバックアップとリストアをサポートしていません。
- 現在のところ、StarRocks はユーザー、権限、およびリソースグループの設定に関連するデータのバックアップとリストアをサポートしていません。
- StarRocks は、テーブル間の Colocate Join 関係のバックアップとリストアをサポートしていません。
