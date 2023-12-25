---
displayed_sidebar: Chinese
---

# エクスポート

## 阿里云OSSバックアップとリストア

StarRocksは、阿里云OSSやAWS S3（またはS3プロトコル互換のオブジェクトストレージ）へのデータバックアップをサポートしています。2つのStarRocksクラスターがあると仮定し、それぞれDB1クラスターとDB2クラスターとします。DB1のデータを阿里云OSSにバックアップし、必要に応じてDB2にリストアする必要があります。バックアップとリストアの大まかなプロセスは以下の通りです：

### クラウドリポジトリの作成

DB1とDB2でそれぞれ以下のSQLを実行します：

```sql
CREATE REPOSITORY `リポジトリ名`
WITH BROKER `broker_name`
ON LOCATION "oss://バケット名/パス"
PROPERTIES
(
"fs.oss.accessKeyId" = "xxx",
"fs.oss.accessKeySecret" = "yyy",
"fs.oss.endpoint" = "oss-cn-beijing.aliyuncs.com"
);
```

a. DB1とDB2の両方で作成が必要であり、作成するREPOSITORYの名前は同じでなければなりません。リポジトリの確認は以下の通りです：

```sql
SHOW REPOSITORIES;
```

b. broker_nameには、クラスタ内のブローカー名を入力する必要があります。BrokerNameの確認は以下の通りです：

```sql
SHOW BROKER;
```

c. fs.oss.endpointのパスにはバケット名を含めないでください。

### データテーブルのバックアップ

DB1でバックアップする必要があるテーブルをクラウドリポジトリにBACKUPします。DB1で以下のSQLを実行します：

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
PROPERTIESでは現在以下の属性がサポートされています：
"type" = "full"：これは全量バックアップを意味します（デフォルト）。
"timeout" = "3600"：タスクのタイムアウト時間で、デフォルトは1日、単位は秒です。
```

StarRocksは現在、データベース全体のバックアップをサポートしていません。ON (......)でバックアップする必要があるテーブルやパーティションを指定する必要があります。これらのテーブルやパーティションは並行してバックアップされます。
進行中のバックアップタスクを確認するには（同時に実行できるバックアップタスクは1つだけです）：

```sql
SHOW BACKUP FROM db_name;
```

バックアップが完了したら、OSSでバックアップデータが存在するかどうかを確認できます（不要なバックアップはOSSで削除する必要があります）：

```sql
SHOW SNAPSHOT ON `OSSリポジトリ名`; 
```

### データのリストア

DB2でデータをリストアします。DB2ではリストアするテーブルの構造を事前に作成する必要はありません。Restore操作を実行する過程で自動的に作成されます。リストアSQLを実行します：

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM `repository_name`
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

リストアの進行状況を確認するには：

```sql
SHOW RESTORE;
```
