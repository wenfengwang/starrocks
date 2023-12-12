---
表示されるサイドバー: "日本語"
---

# データのエクスポート

## Alibaba Cloud OSS バックアップおよびリストア

StarRocksは、データをalicloud OSS / AWS S3（またはS3プロトコルに準拠したオブジェクトストレージ）にバックアップすることをサポートしています。DB1クラスターとDB2クラスターという2つのStarRocksクラスターがあると仮定します。DB1のデータをalicloud OSSにバックアップし、必要に応じてDB2にリストアする必要があります。バックアップおよびリカバリの一般的なプロセスは次のとおりです:

### クラウドリポジトリの作成

それぞれDB1およびDB2でSQLを実行します:

```sql
CREATE REPOSITORY `リポジトリ名`
WITH BROKER `ブローカー名`
ON LOCATION "oss://バケット名/パス"
PROPERTIES
(
"fs.oss.accessKeyId" = "xxx",
"fs.oss.accessKeySecret" = "yyy",
"fs.oss.endpoint" = "oss-cn-beijing.aliyuncs.com"
);
```

a. DB1およびDB2の両方を作成する必要があり、作成されたREPOSITORY名は同じである必要があります。リポジトリを表示:

```sql
SHOW REPOSITORIES;
```

b. broker_名は、クラスター内のブローカー名を記入する必要があります。BrokerNameを表示:

```sql
SHOW BROKER;
```

c. fs.oss.endpointの後のパスには、バケット名が含まれる必要はありません。

### バックアップデータテーブル

DB1でクラウドリポジトリへバックアップするテーブルをバックアップします。DB1で以下のSQLを実行します:

```sql
BACKUP SNAPSHOT [db_name].{スナップショット名}
TO `リポジトリ名`
ON (
`table_name` [PARTITION (`p1`, ...)],
...
)
PROPERTIES ("key"="value", ...);
```

```plain text
PROPERTIESで現在サポートされているプロパティは次のとおりです:
"type" = "full": これがフルアップデートを示します（デフォルト）。
"timeout" = "3600": タスクのタイムアウト。デフォルトは1日です。単位は秒です。
```

現時点ではStarRocksはフルデータベースバックアップをサポートしていません。バックアップを実行するテーブルまたはパーティションをON(...)で指定する必要があり、これらのテーブルまたはパーティションは並列にバックアップされます。

進行中のバックアップタスクを表示します（同時に1つのバックアップタスクのみが実行できることに注意してください）:

```sql
SHOW BACKUP FROM db_name;
```

バックアップが完了したら、OSS内のバックアップデータが既に存在するかどうかを確認できます（不要なバックアップはOSSで削除する必要があります）:

```sql
SHOW SNAPSHOT ON OSS repository name; 
```

### データのリストア

DB2でのデータリストアにおいて、DB2でリストアするテーブル構造を作成する必要はありません。リストア操作中に自動的に作成されます。リストアSQLを実行します:

```sql
RESTORE SNAPSHOT [db_name].{スナップショット名}
FROMrepository_name``
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

リストアの進行状況を表示します:

```sql
SHOW RESTORE;
```