---
displayed_sidebar: English
---

# データエクスポート

## Alibaba Cloud OSS バックアップおよびリストア

StarRocks は、alicloud OSS / AWS S3（または S3 プロトコルと互換性のあるオブジェクトストレージ）へのデータバックアップをサポートしています。DB1 クラスターと DB2 クラスターという 2 つの StarRocks クラスターがあるとします。DB1 のデータを alicloud OSS にバックアップし、必要に応じて DB2 にリストアする必要があります。バックアップとリカバリの一般的なプロセスは以下の通りです。

### クラウドリポジトリの作成

DB1 と DB2 でそれぞれ SQL を実行します：

```sql
CREATE REPOSITORY `repository_name`
WITH BROKER `broker_name`
ON LOCATION "oss://bucket_name/path"
PROPERTIES
(
"fs.oss.accessKeyId" = "xxx",
"fs.oss.accessKeySecret" = "yyy",
"fs.oss.endpoint" = "oss-cn-beijing.aliyuncs.com"
);
```

a. DB1 と DB2 の両方でリポジトリを作成する必要があり、作成される REPOSITORY の名前は同じであるべきです。リポジトリを表示します：

```sql
SHOW REPOSITORIES;
```

b. `broker_name` にはクラスタ内のブローカー名を入力する必要があります。BrokerName を表示します：

```sql
SHOW BROKER;
```

c. `fs.oss.endpoint` の後のパスにバケット名を含める必要はありません。

### データテーブルのバックアップ

DB1 のクラウドリポジトリにバックアップするテーブルを BACKUP します。DB1 で SQL を実行します：

```sql
BACKUP SNAPSHOT [db_name].{snapshot_name}
TO `repository_name`
ON (
`table_name` [PARTITION (`p1`, ...)],
...
)
PROPERTIES ("key"="value", ...);
```

```plain text
PROPERTIES は現在以下のプロパティをサポートしています：
"type" = "full": これは完全更新であることを示します（デフォルト）。
"timeout" = "3600": タスクのタイムアウトです。デフォルトは1日で、単位は秒です。
```

StarRocks は現在、データベース全体のバックアップをサポートしていません。バックアップするテーブルやパーティションを ON (...) で指定する必要があり、これらのテーブルやパーティションは並行してバックアップされます。

進行中のバックアップタスクを表示します（一度に実行できるバックアップタスクは1つのみです）：

```sql
SHOW BACKUP FROM db_name;
```

バックアップが完了したら、OSS にバックアップデータが既に存在するかどうかを確認できます（不要なバックアップは OSS で削除する必要があります）：

```sql
SHOW SNAPSHOT ON `repository_name`; 
```

### データのリストア

DB2 でのデータリストアには、リストアするテーブル構造を DB2 で事前に作成する必要はありません。リストア操作中に自動的に作成されます。リストア SQL を実行します：

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM `repository_name`
ON (
    `table_name` [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

リストアの進行状況を表示します：

```sql
SHOW RESTORE;
```
