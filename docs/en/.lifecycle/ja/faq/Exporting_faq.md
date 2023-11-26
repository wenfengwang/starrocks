---
displayed_sidebar: "Japanese"
---

# データのエクスポート

## Alibaba Cloud OSS のバックアップとリストア

StarRocks は、データを alicloud OSS / AWS S3（またはS3プロトコルに互換性のあるオブジェクトストレージ）にバックアップすることができます。DB1クラスタとDB2クラスタの2つのStarRocksクラスタがあるとします。DB1のデータをalicloud OSSにバックアップし、必要な場合にDB2にリストアする必要があります。バックアップとリストアの一般的な手順は次のとおりです：

### クラウドリポジトリの作成

それぞれDB1とDB2でSQLを実行します：

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

a. DB1とDB2の両方を作成する必要があり、作成したREPOSITORY名は同じである必要があります。リポジトリを表示します：

```sql
SHOW REPOSITORIES;
```

b. broker_nameにはクラスタ内のブローカー名を入力する必要があります。BrokerNameを表示します：

```sql
SHOW BROKER;
```

c. fs.oss.endpointの後のパスにはバケット名は必要ありません。

### データテーブルのバックアップ

DB1のクラウドリポジトリにバックアップするテーブルをバックアップします。DB1で以下のSQLを実行します：

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
PROPERTIESは現在、次のプロパティをサポートしています：
"type" = "full"：これが完全な更新であることを示します（デフォルト）。
"timeout" = "3600"：タスクのタイムアウト。デフォルトは1日です。単位は秒です。
```

StarRocksは現在、完全なデータベースのバックアップをサポートしていません。バックアップするテーブルまたはパーティションをON(...)で指定する必要があり、これらのテーブルまたはパーティションは並列にバックアップされます。

進行中のバックアップタスクを表示します（同時に実行できるバックアップタスクは1つだけです）：

```sql
SHOW BACKUP FROM db_name;
```

バックアップが完了したら、OSSにバックアップデータが既に存在するかどうかを確認できます（不要なバックアップはOSSで削除する必要があります）：

```sql
SHOW SNAPSHOT ON OSS repository name; 
```

### データのリストア

DB2でのデータのリストアでは、リストアするテーブル構造をDB2で作成する必要はありません。リストア操作中に自動的に作成されます。リストアSQLを実行します：

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM repository_name
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

リストアの進行状況を表示します：

```sql
SHOW RESTORE;
```
