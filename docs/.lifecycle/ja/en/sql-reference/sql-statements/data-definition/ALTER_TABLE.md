---
displayed_sidebar: English
---

# ALTER TABLE

## 説明

既存のテーブルを変更することができます。変更内容には以下が含まれます：

- [テーブル、パーティション、インデックスの名前を変更する](#rename)
- [テーブルのコメントを変更する](#alter-table-comment-from-v31)
- [パーティションの変更（パーティションの追加/削除およびパーティション属性の変更）](#modify-partition)
- [バケット化方法とバケット数の変更](#modify-the-bucketing-method-and-number-of-buckets-from-v32)
- [列の変更（列の追加/削除、列の順序の変更）](#modify-columns-adddelete-columns-change-the-order-of-columns)
- [ロールアップインデックスの作成/削除](#modify-rollup-index)
- [ビットマップインデックスの変更](#modify-bitmap-indexes)
- [テーブルプロパティの変更](#modify-table-properties)
- [アトミックスワップ](#swap)
- [データバージョンの手動圧縮](#manual-compaction-from-31)

> **注記**
>
> この操作には、対象テーブルに対するALTER権限が必要です。

## 構文

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

`alter_clause` は以下の操作を含むことができます：rename、comment、partition、bucket、column、rollup index、bitmap index、table property、swap、compaction。

- rename: テーブル、ロールアップインデックス、またはパーティションの名前を変更します。**列名は変更できないことに注意してください。**
- comment: テーブルのコメントを変更します（**v3.1以降でサポート**）。
- partition: パーティションのプロパティを変更、パーティションを削除、またはパーティションを追加します。
- bucket: バケット化方法とバケット数を変更します。
- column: 列を追加、削除、または並べ替え、または列の型を変更します。
- rollup index: ロールアップインデックスを作成または削除します。
- bitmap index: インデックスを変更します（変更可能なのはビットマップインデックスのみです）。
- swap: 二つのテーブルをアトミックに交換します。
- compaction: 手動圧縮を実行して、ロードされたデータのバージョンをマージします（**v3.1以降でサポート**）。

:::note

- パーティション、カラム、ロールアップインデックスに関する操作は、一つのALTER TABLEステートメントでは実行できません。
- バケット、カラム、ロールアップインデックスに関する操作は非同期操作です。タスクが送信された直後に成功メッセージが返されます。[SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)コマンドを実行して進行状況を確認し、[CANCEL ALTER TABLE](../data-definition/CANCEL_ALTER_TABLE.md)コマンドを実行して操作をキャンセルできます。
- rename、comment、partition、bitmap index、swapに関する操作は同期操作であり、コマンドの戻り値が実行完了を示します。
:::

### 名前の変更

Renameは、テーブル名、ロールアップインデックス、およびパーティション名の変更をサポートしています。

#### テーブルの名前を変更する

```sql
ALTER TABLE <tbl_name> RENAME TO <new_tbl_name>
```

#### ロールアップインデックスの名前を変更する

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME ROLLUP <old_rollup_name> TO <new_rollup_name>
```

#### パーティションの名前を変更する

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME PARTITION <old_partition_name> TO <new_partition_name>
```

### テーブルコメントの変更（v3.1以降）

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name> SET COMMENT = "<new table comment>";
```

### パーティションの変更

#### パーティションを追加する

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]];
```

注記：

1. partition_descは以下の二つの表現をサポートしています：

    ```plain
    VALUES LESS THAN [MAXVALUE|("value1", ...)]
    VALUES ("value1", ...), ("value1", ...)
    ```

2. partitionは左閉じ右開きの区間です。ユーザーが右境界のみを指定した場合、システムは自動的に左境界を決定します。
3. バケットモードが指定されていない場合、既存のテーブルで使用されているバケット方式が自動的に使用されます。
4. バケットモードが指定された場合、バケット数のみを変更でき、バケット方式やバケット列は変更できません。
5. ユーザーは`["key"="value"]`でパーティションのいくつかのプロパティを設定することができます。詳細は[CREATE TABLE](CREATE_TABLE.md)を参照してください。

#### パーティションを削除する

構文：

```sql
-- 2.0以前
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS | FORCE] <partition_name>
-- 2.0以降
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS] <partition_name> [FORCE]
```

注記：

1. パーティションテーブルには少なくとも1つのパーティションが必要です。
2. DROP PARTITIONを実行してからしばらくすると、RECOVER文で削除されたパーティションを復旧することができます。詳細はRECOVERステートメントを参照してください。
3. DROP PARTITION FORCEを実行すると、パーティションが直接削除され、パーティションに未完了のアクティビティがあるかどうかを確認せずに復旧することはできません。したがって、通常はこの操作は推奨されません。

#### 一時パーティションを追加する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name> 
ADD TEMPORARY PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]]
```

#### 現在のパーティションと一時パーティションを置き換える

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
REPLACE PARTITION <partition_name>
WITH TEMPORARY PARTITION <temporary_partition_name>
[PROPERTIES ("key"="value", ...)]
```

#### 一時パーティションを削除する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP TEMPORARY PARTITION <partition_name>
```

#### パーティションプロパティの変更

**構文**

```sql
ALTER TABLE [<db_name>.]<tbl_name>
    MODIFY PARTITION { <partition_name> | partition_name_list | (*) }
        SET ("key" = "value", ...);
```

**使用法**

- パーティションの以下のプロパティを変更できます：

  - storage_medium
  - storage_cooldown_ttl または storage_cooldown_time
  - replication_num

- 一つのパーティションしかないテーブルの場合、そのパーティション名はテーブル名と同じです。複数のパーティションに分割されたテーブルでは、`(*)`を使用してすべてのパーティションのプロパティを変更することができ、これはより便利です。

- `SHOW PARTITIONS FROM <tbl_name>`を実行して、変更後のパーティションプロパティを確認します。

### バケット化方法とバケット数の変更（v3.2以降）

構文：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ partition_names ]
[ distribution_desc ]

partition_names ::= 
    (PARTITION | PARTITIONS) ( <partition_name> [, <partition_name> ...] )

distribution_desc ::=
    DISTRIBUTED BY RANDOM [ BUCKETS <num> ] |
    DISTRIBUTED BY HASH ( <column_name> [, <column_name> ...] ) [ BUCKETS <num> ]
```

例：


たとえば、元のテーブルはDuplicate Keyテーブルで、ハッシュバケッティングが使用され、バケットの数はStarRocksによって自動的に設定されます。

```SQL
CREATE TABLE IF NOT EXISTS details (
    event_time DATETIME NOT NULL COMMENT "イベントの日時",
    event_type INT NOT NULL COMMENT "イベントの種類",
    user_id INT COMMENT "ユーザーID",
    device_code INT COMMENT "デバイスコード",
    channel INT COMMENT "チャネル"
)
DUPLICATE KEY(event_time, event_type)
PARTITION BY date_trunc('day', event_time)
DISTRIBUTED BY HASH(user_id);

-- 複数日のデータを挿入
INSERT INTO details (event_time, event_type, user_id, device_code, channel) VALUES
-- 11月26日のデータ
('2023-11-26 08:00:00', 1, 101, 12345, 2),
('2023-11-26 09:15:00', 2, 102, 54321, 3),
('2023-11-26 10:30:00', 1, 103, 98765, 1),
-- 11月27日のデータ
('2023-11-27 08:30:00', 1, 104, 11111, 2),
('2023-11-27 09:45:00', 2, 105, 22222, 3),
('2023-11-27 11:00:00', 1, 106, 33333, 1),
-- 11月28日のデータ
('2023-11-28 08:00:00', 1, 107, 44444, 2),
('2023-11-28 09:15:00', 2, 108, 55555, 3),
('2023-11-28 10:30:00', 1, 109, 66666, 1);
```

#### バケッティング方法のみを変更する

> **注意**
>
> - 変更はテーブル内の全てのパーティションに適用され、特定のパーティションのみには適用できません。
> - バケッティング方法のみを変更する場合でも、コマンドでバケット数を指定する必要があります。`BUCKETS <num>`が指定されていない場合、バケット数はStarRocksによって自動的に決定されることを意味します。

- バケッティング方法をハッシュバケッティングからランダムバケッティングに変更し、バケット数はStarRocksによって自動的に設定されたままにします。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY RANDOM;
  ```

- ハッシュバケッティングのキーを`event_time, event_type`から`user_id, event_time`に変更し、バケット数はStarRocksによって自動的に設定されたままにします。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time);
  ```

#### バケット数のみを変更する

> **注意**
>
> バケット数のみを変更する場合でも、コマンドでバケッティング方法を指定する必要があります（例: `HASH(user_id)`）。

- 全てのパーティションのバケット数をStarRocksによって自動的に設定されるものから10に変更します。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id) BUCKETS 10;
  ```

- 指定されたパーティションのバケット数をStarRocksによって自動的に設定されるものから15に変更します。

  ```SQL
  ALTER TABLE details PARTITIONS (p20231127, p20231128) DISTRIBUTED BY HASH(user_id) BUCKETS 15;
  ```

  > **注記**
  >
  > パーティション名は`SHOW PARTITIONS FROM <table_name>;`を実行することで確認できます。

#### バケッティング方法とバケット数の両方を変更する

> **注意**
>
> 変更はテーブル内の全てのパーティションに適用され、特定のパーティションのみには適用できません。

- バケッティング方法をハッシュバケッティングからランダムバケッティングに変更し、バケット数をStarRocksによって自動的に設定されるものから10に変更します。

   ```SQL
   ALTER TABLE details DISTRIBUTED BY RANDOM BUCKETS 10;
   ```

- ハッシュバケッティングのキーを`event_time, event_type`から`user_id, event_time`に変更し、バケット数をStarRocksによって自動的に設定されるものから10に変更します。

  ```SQL
  ALTER TABLE details DISTRIBUTED BY HASH(user_id, event_time) BUCKETS 10;
  ```

### 列の変更（列の追加/削除、列の順序の変更）

#### 指定されたインデックスの特定の位置に列を追加する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN column_name column_type [KEY | agg_type] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[TO rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注記：

1. 集約テーブルに値列を追加する場合は、agg_typeを指定する必要があります。
2. 非集約テーブル（Duplicate Keyテーブルなど）にキー列を追加する場合は、KEYキーワードを指定する必要があります。
3. ベースインデックスに既に存在する列をロールアップインデックスに追加することはできません。（必要に応じてロールアップインデックスを再作成できます。）

#### 指定されたインデックスに複数の列を追加する

構文：

- 複数の列を追加

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 複数の列を追加し、AFTERを使用して追加される列の位置を指定

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name,
  ADD COLUMN column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER column_name
  [, ...]
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注記：

1. 集約テーブルに値列を追加する場合は、agg_typeを指定する必要があります。

2. 非集約テーブルにキー列を追加する場合は、KEYキーワードを指定する必要があります。

3. ベースインデックスに既に存在する列をロールアップインデックスに追加することはできません。（必要に応じて別のロールアップインデックスを作成できます。）

#### 生成列を追加する（v3.1以降）

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

生成列を追加し、その式を指定できます。[生成列](../generated_columns.md)を使用すると、式の結果を事前に計算して格納することができ、同じ複雑な式を含むクエリの速度を大幅に向上させることができます。v3.1以降、StarRocksは生成列をサポートしています。

#### 指定されたインデックスから列を削除する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注記：

1. パーティション列を削除することはできません。
2. 列がベースインデックスから削除された場合、ロールアップインデックスに含まれている場合も削除されます。

#### 指定されたインデックスの列タイプと列位置を変更する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
MODIFY COLUMN column_name column_type [KEY | agg_type] [NULL | NOT NULL] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注記：

1. 集約モデルで値列を変更する場合は、agg_typeを指定する必要があります。
2. 非集約モデルでキー列を変更する場合は、KEYキーワードを指定する必要があります。
3. 変更できるのは列のタイプのみです。列のその他のプロパティは、現在のままです。（つまり、他のプロパティは、元のプロパティに従ってステートメントに明示的に記述する必要があります。[列](#column)の部分を参照してください。）
4. パーティション列を変更することはできません。
5. 現在、次のタイプの変換がサポートされています（精度の損失はユーザーが保証します）。

   - TINYINT/SMALLINT/INT/BIGINTをTINYINT/SMALLINT/INT/BIGINT/DOUBLEに変換します。
   - TINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMAL を VARCHAR に変換します。VARCHAR は、最大長の変更をサポートします。
   - VARCHARをTINYINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLEに変換します。
   - VARCHARをDATEに変換します（現在、"%Y-%m-%d"、"%y-%m-%d"、"%Y%m%d"、"%y%m%d"、"%Y/%m/%d"、"%y/%m/%d"の6つの形式をサポートしています）
   - DATETIMEをDATEに変換します（年月日の情報のみが保持されます。例：`2019-12-09 21:47:05` `<-->` `2019-12-09`）
   - DATEをDATETIMEに変換します（時、分、秒を0に設定します。例：`2019-12-09` `<-->` `2019-12-09 00:00:00`）
   - FLOATをDOUBLEに変換します
   - INTをDATEに変換します（INTデータの変換に失敗した場合、元のデータはそのままです）

6. NULLからNOT NULLへの変換はサポートされていません。

#### 指定されたインデックスの列を並べ替える

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注記：

- インデックス内のすべての列を記述する必要があります。
- 値列はキー列の後にリストされます。

#### プライマリキーテーブルのソートキー列を変更する

<!--Supported Versions-->

構文：

```SQL
ALTER TABLE [<db_name>.]<table_name>
[ order_desc ]

order_desc ::=
    ORDER BY <column_name> [, <column_name> ...]
```

例：

例えば、元のテーブルはソートキーとプライマリキーが結合されているプライマリキーテーブルで、それは `dt, order_id` です。

```SQL
create table orders (
    dt date NOT NULL,
    order_id bigint NOT NULL,
    user_id int NOT NULL,
    merchant_id int NOT NULL,
    good_id int NOT NULL,
    good_name string NOT NULL,
    price int NOT NULL,
    cnt int NOT NULL,
    revenue int NOT NULL,
    state tinyint NOT NULL
) PRIMARY KEY (dt, order_id)
PARTITION BY date_trunc('day', dt)
DISTRIBUTED BY HASH(order_id);
```

ソートキーをプライマリキーから切り離し、ソートキーを `dt, revenue, state` に変更します。

```SQL
ALTER TABLE orders ORDER BY (dt, revenue, state);
```

### ロールアップインデックスの変更

#### ロールアップインデックスを作成する

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)]
```

PROPERTIES: タイムアウト時間の設定をサポートし、デフォルトのタイムアウト時間は1日です。

例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1, col2) FROM r0;
```

#### ロールアップインデックスをバッチで作成する

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1, col2) FROM r0, r2(col3, col4) FROM r0;
```

注記：

1. from_index_nameが指定されていない場合は、デフォルトでベースインデックスから作成されます。
2. ロールアップテーブルの列は、from_indexの既存の列でなければなりません。
3. propertiesでは、ユーザーはストレージフォーマットを指定できます。詳細はCREATE TABLEを参照してください。

#### ロールアップインデックスを削除する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### ロールアップインデックスをバッチで削除する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

注：ベースインデックスは削除できません。

### ビットマップインデックスの変更

ビットマップインデックスは以下の変更をサポートしています：

#### ビットマップインデックスを作成する

構文：

```sql
 ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...]) [USING BITMAP] [COMMENT 'balabala'];
```

注記：

```plain text
1. ビットマップインデックスは現バージョンのみでサポートされています。
2. ビットマップインデックスは単一の列にのみ作成されます。
```

#### ビットマップインデックスを削除する

構文：

```sql
DROP INDEX index_name;
```

### テーブルプロパティの変更

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SET ("key" = "value",...)
```

現在、StarRocksは以下のテーブルプロパティの変更をサポートしています：

- `replication_num`
- `default.replication_num`
- `storage_cooldown_ttl`
- `storage_cooldown_time`
- 動的パーティショニングに関連するプロパティ
- `enable_persistent_index`
- `bloom_filter_columns`
- `colocate_with`
- `bucket_size`（3.2以降でサポート）

注記：
また、列に対する上記の操作を組み合わせることで、プロパティを変更することもできます。詳細は[以下の例](#examples)を参照してください。

### スワップ

スワップは、2つのテーブルをアトミックに交換することをサポートします。

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手動コンパクション（3.1以降）

StarRocksはコンパクションメカニズムを使用して、ロードされたデータの異なるバージョンをマージします。この機能により、小さなファイルを大きなファイルに結合でき、クエリパフォーマンスが効果的に向上します。

v3.1以前では、コンパクションは以下の2つの方法で実行されます：

- システムによる自動コンパクション：コンパクションはバックグラウンドでBEレベルで実行されます。ユーザーはコンパクションを実行するデータベースやテーブルを指定できません。
- ユーザーはHTTPインターフェースを呼び出してコンパクションを実行できます。

v3.1から、StarRocksはSQLコマンドを実行して手動でコンパクションを実行するためのSQLインターフェースを提供します。これにより、特定のテーブルやパーティションを選択してコンパクションを実行できます。これにより、コンパクションプロセスの柔軟性と制御が向上します。

構文：

```sql
-- テーブル全体にコンパクションを実行します。
ALTER TABLE <tbl_name> COMPACT

-- 単一のパーティションにコンパクションを実行します。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 複数のパーティションにコンパクションを実行します。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[, <partition2_name>,...])

-- 累積コンパクションを実行します。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[, <partition2_name>,...])

-- ベースコンパクションを実行します。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[, <partition2_name>,...])
```

`information_schema`データベースの`be_compactions`テーブルにはコンパクション結果が記録されています。`SELECT * FROM information_schema.be_compactions;`を実行して、コンパクション後のデータバージョンを照会できます。

## 例

### テーブル

1. 新しく追加されたパーティションのデフォルトのレプリカ数として使用されるテーブルのデフォルトレプリカ数を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("default.replication_num" = "2");
    ```

2. 単一パーティションテーブルの実際のレプリカ数を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replication_num" = "3");
    ```

3. レプリカ間でのデータ書き込みとレプリケーションモードを変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
    ```


   この例では、レプリカ間のデータ書き込みとレプリケーションモードを「リーダーレスレプリケーション」に設定しています。これは、プライマリレプリカとセカンダリレプリカを区別せずに、複数のレプリカに同時にデータを書き込むことを意味します。詳細については、[CREATE TABLE](CREATE_TABLE.md)の`replicated_storage`パラメータを参照してください。

### パーティション

1. パーティションを追加し、デフォルトのバケッティングモードを使用します。既存のパーティションは[MIN, 2013-01-01)です。追加されたパーティションは[2013-01-01, 2014-01-01)です。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
    ```

2. パーティションを追加し、新しいバケット数を使用します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    DISTRIBUTED BY HASH(k1);
    ```

3. パーティションを追加し、新しいレプリカ数を使用します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    ("replication_num"="1");
    ```

4. パーティションのレプリカ数を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION p1 SET("replication_num"="1");
    ```

5. 指定されたパーティションのレプリカ数をバッチで変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (p1, p2, p4) SET("replication_num"="1");
    ```

6. すべてのパーティションのストレージ媒体をバッチで変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (*) SET("storage_medium"="HDD");
    ```

7. パーティションを削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP PARTITION p1;
    ```

8. 上限と下限を持つパーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
    ```

### ロールアップインデックス

1. ベースインデックス(k1, k2, k3, v1, v2)に基づいてロールアップインデックス`example_rollup_index`を作成します。列ベースのストレージが使用されます。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
    PROPERTIES("storage_type"="column");
    ```

2. `example_rollup_index(k1, k3, v1, v2)`に基づいてインデックス`example_rollup_index2`を作成します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index2 (k1, v1)
    FROM example_rollup_index;
    ```

3. ベースインデックス(k1, k2, k3, v1)に基づいてインデックス`example_rollup_index3`を作成します。ロールアップタイムアウト時間は1時間に設定されます。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index3(k1, k3, v1)
    PROPERTIES("storage_type"="column", "timeout"="3600");
    ```

4. インデックス`example_rollup_index2`を削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP ROLLUP example_rollup_index2;
    ```

### カラム

1. `example_rollup_index`の`col1`カラムの後にキーカラム`new_col`(非集約カラム)を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

2. `example_rollup_index`の`col1`カラムの後に値カラム`new_col`(非集約カラム)を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

3. `example_rollup_index`の`col1`カラムの後にキーカラム`new_col`(集約カラム)を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

4. `example_rollup_index`の`col1`カラムの後に値カラム`new_col SUM`(集約カラム)を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

5. `example_rollup_index`に複数のカラム(集約)を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
    TO example_rollup_index;
    ```

6. `AFTER`を使用して追加されたカラムの位置を指定しながら、`example_rollup_index`に複数のカラム(集約)を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
    ADD COLUMN col2 FLOAT SUM AFTER `v2`,
    TO example_rollup_index;
    ```

7. `example_rollup_index`からカラムを削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    FROM example_rollup_index;
    ```

8. ベースインデックスの`col1`カラムの型をBIGINTに変更し、`col2`の後に配置します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
    ```

9. ベースインデックスの`val1`カラムの最大長を64に変更します。元の長さは32です。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
    ```

10. `example_rollup_index`のカラムの順序を変更します。元のカラムの順序はk1, k2, k3, v1, v2です。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 一度に2つの操作(ADD COLUMNとORDER BY)を実行します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. テーブルのブルームフィルターカラムを変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("bloom_filter_columns"="k1,k2,k3");
     ```

     この操作は、上記のカラム操作に統合することもできます（複数の節の構文が若干異なることに注意してください）。

     ```sql
     ALTER TABLE example_db.my_table
     DROP COLUMN col2
     PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
     ```

### テーブルプロパティ

1. テーブルのコロケートプロパティを変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("colocate_with" = "t1");
     ```

2. テーブルの動的パーティションプロパティを変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("dynamic_partition.enable" = "false");
     ```

     動的パーティションプロパティが設定されていないテーブルに動的パーティションプロパティを追加する必要がある場合は、すべての動的パーティションプロパティを指定する必要があります。

     ```sql
     ALTER TABLE example_db.my_table
     SET (
         "dynamic_partition.enable" = "true",
         "dynamic_partition.time_unit" = "DAY",
         "dynamic_partition.end" = "3",
         "dynamic_partition.prefix" = "p",
         "dynamic_partition.buckets" = "32"
         );
     ```

### 名前変更

1. `table1`の名前を`table2`に変更します。

    ```sql
    ALTER TABLE table1 RENAME TO table2;
    ```

2. `example_table`のロールアップインデックス`rollup1`の名前を`rollup2`に変更します。

    ```sql
    ALTER TABLE example_table RENAME ROLLUP rollup1 TO rollup2;
    ```

3. `example_table`のパーティション`p1`の名前を`p2`に変更します。

    ```sql
    ALTER TABLE example_table RENAME PARTITION p1 TO p2;
    ```

### ビットマップインデックス

1. `table1`のカラム`siteid`にビットマップインデックスを作成します。

    ```sql
    ALTER TABLE table1
    ADD INDEX index_1 (siteid) USING BITMAP COMMENT 'balabala';
    ```

2. `table1`のカラム`siteid`のビットマップインデックスを削除します。

    ```sql
    ALTER TABLE table1
    DROP INDEX index_1;
    ```

### スワップ

`table1`と`table2`間のアトミックスワップ。

```sql
ALTER TABLE table1 SWAP WITH table2;
```

### 手動コンパクション

```sql
CREATE TABLE compaction_test( 
    event_day DATE,
    pv BIGINT)
DUPLICATE KEY(event_day)
PARTITION BY date_trunc('month', event_day);
DISTRIBUTED BY HASH(event_day) BUCKETS 8
PROPERTIES("replication_num" = "3");

INSERT INTO compaction_test VALUES
('2023-02-14', 2),
('2033-03-01',2);
{'label':'insert_734648fa-c878-11ed-90d6-00163e0dcbfc', 'status':'VISIBLE', 'txnId':'5008'}

INSERT INTO compaction_test VALUES
('2023-02-14', 2),('2033-03-01',2);
{'label':'insert_85c95c1b-c878-11ed-90d6-00163e0dcbfc', 'status':'VISIBLE', 'txnId':'5009'}

ALTER TABLE compaction_test COMPACT;

ALTER TABLE compaction_test COMPACT p203303;

ALTER TABLE compaction_test COMPACT (p202302,p203303);

ALTER TABLE compaction_test CUMULATIVE COMPACT (p202302,p203303);

ALTER TABLE compaction_test BASE COMPACT (p202302,p203303);
```

## 参照文献

- [CREATE TABLE](./CREATE_TABLE.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)
- [DROP TABLE](./DROP_TABLE.md)
