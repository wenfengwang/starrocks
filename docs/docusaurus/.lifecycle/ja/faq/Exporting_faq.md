```yaml
---
displayed_sidebar: "Japanese"
---

# データエクスポート

## Alibaba Cloud OSS バックアップとリストア

StarRocks は、alibaba cloud OSS/AWS S3 (またはS3プロトコルに準拠したオブジェクトストレージ) へのデータバックアップをサポートしています。DB1 クラスタおよび DB2 クラスタという2つの StarRocks クラスタがあると仮定します。DB1 のデータを alibaba cloud OSS にバックアップし、必要に応じて DB2 にリストアする必要があります。バックアップおよびリストアの一般的な手順は次のとおりです。

### クラウドリポジトリの作成

それぞれの DB1 および DB2 で以下の SQL を実行します。

```sql
CREATE REPOSITORY `リポジトリ名`
WITH BROKER `ブローカー名`
ON LOCATION "oss://bucket名/パス"
PROPERTIES
(
"fs.oss.accessKeyId" = "xxx",
"fs.oss.accessKeySecret" = "yyy",
"fs.oss.endpoint" = "oss-cn-beijing.aliyuncs.com"
);
```

a. DB1 および DB2 の両方を作成する必要があり、作成したリポジトリ名は同じでなければなりません。リポジトリを表示:

```sql
SHOW REPOSITORIES;
```

b. broker_ name には、クラスタのブローカー名を記入する必要があります。BrokerName を表示:

```sql
SHOW BROKER;
```

c. fs.oss.endpoint の後のパスにはバケット名が必要ありません。

### バックアップデータテーブル

バックアップするテーブルを DB1 のクラウドリポジトリにバックアップします。DB1 で以下の SQL を実行します:

```sql
バックアップ SNAPSHOT [db_name].{snapshot_name}
TO `repositori_name`
ON (
`table_name` [PARTITION (`p1`, ...)],
...
)
PROPERTIES ("key"="value", ...);
```

```plain text
現在、PROPERTIES は以下のプロパティをサポートしています:
"type" = "full": これがフルアップデートであることを示します（デフォルト値）。
"timeout" = "3600": タスクのタイムアウト。デフォルトは1日です。単位は秒です。
```

現在、StarRocks はフルデータベースのバックアップはサポートしていません。バックアップするテーブルまたはパーティションを明示する必要があります（ON (...)）、そしてこれらのテーブルまたはパーティションは並列にバックアップされます。

進行中のバックアップタスクを表示します（注意: 同時に実行できるバックアップタスクは1つだけです）:

```sql
SHOW BACKUP FROM db_name;
```

バックアップが完了したら、OSS にバックアップデータが既に存在しているかどうかを確認できます（不要なバックアップは OSS で削除する必要があります）:

```sql
SHOW SNAPSHOT ON OSS repository name; 
```

### データリストア

DB2 でのデータリストアにおいて、DB2 でリストアするテーブル構造を作成する必要はありません。リストア操作中に自動的に作成されます。リストア SQL を実行します:

```sql
RESTORE SNAPSHOT [db_name].{snapshot_name}
FROM repository_name
ON (
    'table_name' [PARTITION ('p1', ...)] [AS 'tbl_alias'],
    ...
)
PROPERTIES ("key"="value", ...);
```

リストアの進行状況を表示:

```sql
SHOW RESTORE;
```