---
displayed_sidebar: "Japanese"
---

# ALTER TABLE (テーブルの変更)

## 説明

以下のように、既存のテーブルを変更できます。

- [テーブル、パーティション、インデックスの名前変更](#rename)
- [テーブルコメントの変更](#alter-table-comment-from-v31)
- [アトミックスワップ](#swap)
- [パーティションの追加/削除およびパーティション属性の変更](#modify-partition)
- [スキーマ変更](#schema-change)
- [ロールアップインデックスの作成/削除](#modify-rollup-index)
- [ビットマップインデックスの変更](#modify-bitmap-indexes)
- [手動データバージョンの圧縮](#manual-compaction-from-31)

> **注意**
>
> この操作には、宛先のテーブルに対する ALTER 権限が必要です。

## 構文

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

`alter_clause` は、以下の 6 つの操作に分類されます: partition、rollup、schema change、rename、index、swap、comment、およびcompact。

- rename: テーブル、ロールアップインデックス、またはパーティションの名前を変更します。**列名は変更できません。**
- comment: テーブルコメントを変更します（**v3.1 以降**でサポートされています）。
- swap: 2 つのテーブルのアトミックな交換。
- partition: パーティションのプロパティを変更し、パーティションを削除または追加します。
- schema change: 列の追加、削除、または順序の変更、または列の型の変更を行います。
- rollup: ロールアップインデックスの作成または削除。
- index: インデックスの変更（ビットマップインデックスのみ変更可能）。
- compact: ロードされたデータのバージョンをマージするための手動の圧縮を実行します（**v3.1 以降**でサポートされています）。

:::note

- スキーマ変更、ロールアップ、およびパーティション操作は、1 つの ALTER TABLE ステートメントで実行できません。
- スキーマ変更およびロールアップは非同期操作です。タスクを送信した後にすぐに成功メッセージが返されます。進行状況をチェックするには、[SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md) コマンドを実行できます。
- パーティション、名前変更、スワップ、およびインデックスは同期操作であり、コマンドの実行完了を示す返り値があります。
:::

### リネーム

リネームでは、テーブル名、ロールアップインデックス名、およびパーティション名を変更できます。

#### テーブルの名前変更

```sql
ALTER TABLE <tbl_name> RENAME <new_tbl_name>
```

#### ロールアップインデックスの名前変更

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME ROLLUP <old_rollup_name> <new_rollup_name>
```

#### パーティションの名前変更

```sql
ALTER TABLE [<db_name>.]<tbl_name>
RENAME PARTITION <old_partition_name> <new_partition_name>
```

### テーブルコメントの変更（v3.1 から）

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name> COMMENT = "<new table comment>";
```

### パーティションの変更

#### パーティションの追加

構文:

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]];
```

注意:

1. partition_desc では、次の 2 つの式がサポートされています:

    ```plain
    VALUES LESS THAN [MAXVALUE|("value1", ...)]
    VALUES ("value1", ...), ("value1", ...)
    ```

2. パーティションは左側を閉じ右側を開いた区間です。ユーザーが右端のみを指定した場合、システムは自動的に左端を決定します。
3. バケットモードが指定されていない場合は、組み込みテーブルで使用されるバケットメソッドが自動的に使用されます。
4. バケットモードが指定されている場合は、バケット番号のみを変更できます。バケットモードまたはバケット列を変更することはできません。
5. ユーザーは`["key"="value"]`でパーティションのいくつかのプロパティを設定できます。詳細については [CREATE TABLE](CREATE_TABLE.md) を参照してください。

#### パーティションの削除

構文:

```sql
-- 2.0 以前
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS | FORCE] <partition_name>
-- 2.0 以降
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS] <partition_name> [FORCE]
```

注意:

1. パーティションテーブルでは、少なくとも 1 つのパーティションを保持する必要があります。
2. DROP PARTITION を実行してからしばらくすると、削除されたパーティションを RECOVER ステートメントによって復元できます。詳細については、RECOVER ステートメントを参照してください。
3. DROP PARTITION FORCE を実行すると、パーティションは直接削除され、パーティションに未完了のアクティビティがあるかどうかを確認せずに回復することはできません。通常、この操作は推奨されません。

#### 一時パーティションの追加

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name> 
ADD TEMPORARY PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]]
```

#### 現在のパーティションを一時的なパーティションで置き換える

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
REPLACE PARTITION <partition_name>
partition_desc ["key"="value"]
WITH TEMPORARY PARTITION
partition_desc ["key"="value"]
[PROPERTIES ("key"="value", ...)]
```

#### 一時パーティションの削除

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP TEMPORARY PARTITION <partition_name>
```

#### パーティションのプロパティの変更

**構文**

```sql
ALTER TABLE [<db_name>.]<tbl_name>
    MODIFY PARTITION { <partition_name> | partition_name_list | (*) }
        SET ("key" = "value", ...);
```

**使い方**

- パーティションの以下のプロパティを変更できます:

  - storage_medium
  - storage_cooldown_ttl または storage_cooldown_time
  - replication_num

- 1 つのパーティションしか持たないテーブルの場合、パーティション名はテーブル名と同じです。複数のパーティションに分割されているテーブルの場合は、`(*)`を使用してすべてのパーティションのプロパティを変更できます。これはより便利です。

- `SHOW PARTITIONS FROM <tbl_name>` を実行して、修正後のパーティションプロパティを表示します。

### スキーマ変更

スキーマ変更では、以下の変更をサポートしています。

#### 指定されたインデックスの指定された位置に列を追加

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN column_name column_type [KEY | agg_type] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[TO rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意:

1. 集計テーブルに値列を追加する場合、agg_type を指定する必要があります。
2. 非集計テーブル（重複キーテーブルなど）にキーカラムを追加する場合は、KEY キーワードを指定する必要があります。
3. ベースインデックスに既に存在する列をロールアップインデックスに追加することはできません。（必要に応じてロールアップインデックスを再作成できます。）

#### 指定されたインデックスに複数の列を追加

構文:

- 複数の列を追加

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 複数列を追加し、AFTER を使用して追加された列の場所を指定する

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  ADD COLUMN (column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意:

1. 集計テーブルに値列を追加する場合、agg_type を指定する必要があります。
2. 非集計テーブルにキーカラムを追加する場合、KEY キーワードを指定する必要があります。
3. ベースインデックスに既に存在する列をロールアップインデックスに追加することはできません。（必要に応じて別のロールアップインデックスを作成できます。）

#### 指定されたインデックスから列を削除

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP COLUMN column_name
[FROM rollup_index_name];
```

注意:

1. パーティション列は削除できません。
2. ベースインデックスから列を削除すると、ロールアップインデックスに含まれている場合も削除されます。

#### 指定されたインデックスの列のタイプおよび位置を変更

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
MODIFY COLUMN column_name column_type [KEY | agg_type] [NULL | NOT NULL] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意:

1. 集計モデルで値列を変更する場合、agg_type を指定する必要があります。
2. 非集計モデルでキーカラムを変更する場合、KEY キーワードを指定する必要があります。
3. 列の型のみを変更できます。列のその他のプロパティは現在のままです（つまり、その他のプロパティは元のプロパティに従ってステートメントに明示的に記述する必要があります。詳細は例 8 を参照してください）。
4. パーティション列は変更できません。
5. 現在サポートされている変換タイプは次のとおりです（ユーザーによって精度の損失が保証されます）。

   - TINYINT / SMALLINT / INT / BIGINT を TINYINT / SMALLINT / INT / BIGINT / DOUBLE に変換する。
   - TINTINT / SMALLINT / INT / BIGINT / LARGEINT / FLOAT / DOUBLE / DECIMAL を VARCHAR に変換する。VARCHAR は最大長の変更をサポートします。
   - VARCHAR を TINTINT / SMALLINT / INT / BIGINT / LARGEINT / FLOAT / DOUBLE に変換する。
   - VARCHAR を DATE に変換する（現在、6つの形式がサポートされています：「%Y-%m-%d」、「%y-%m-%d」、「%Y%m%d」、「%y%m%d」、「%Y/%m/%d,」、「%y/%m/%d」）
   - DATETIME を DATE に変換する（年月日の情報のみが保持されます、つまり `2019-12-09 21:47:05` `<-->` `2019-12-09`）
   - DATE を DATETIME に変換する（時間、分、秒をゼロに設定します。例： `2019-12-09` `<-->` `2019-12-09 00:00:00`）
   - FLOAT を DOUBLE に変換する
   - INT を DATE に変換する（INT データの変換に失敗する場合、元のデータが変更されません）

6. NULL から NOT NULL への変換はサポートされていません。

#### 特定のインデックスの列を並べ替える

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意：

1. インデックスのすべての列を記述する必要があります。
2. キーカラムの後に値カラムがリストされます。

#### 生成列を追加する（v3.1 以降）

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

生成列を追加し、その式を指定できます。[生成列](../generated_columns.md) は、同じ複雑な式を持つクエリを大幅に高速化するために、式の結果を事前に計算して格納できます。v3.1 以降、StarRocks は生成列をサポートしています。

#### テーブルプロパティの変更

現在、StarRocks は次のテーブルプロパティの変更をサポートしています：

- `replication_num`
- `default.replication_num`
- `storage_cooldown_ttl`
- `storage_cooldown_time`
- 動的パーティション関連のプロパティ
- `enable_persistent_index`
- `bloom_filter_columns`
- `colocate_with`

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SET ("key" = "value",...)
```

注意：
これらのプロパティは、上記のスキーマ変更操作にマージして変更することもできます。以下の例を参照してください。

### ロールアップインデックスの変更

#### ロールアップインデックスを作成する

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)]
```

プロパティ：タイムアウト時間の設定をサポートし、デフォルトのタイムアウト時間は1日です。

例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1,col2) from r0;
```

#### バッチでロールアップインデックスを作成する

構文：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

例：

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1,col2) from r0, r2(col3,col4) from r0;
```

注意：

1. from_index_name が指定されていない場合、デフォルトでベースインデックスから作成されます。
2. ロールアップテーブルの列は、from_index で既存の列である必要があります。
3. プロパティでは、ユーザーはストレージ形式を指定できます。詳細は CREATE TABLE を参照してください。

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

#### バッチでロールアップインデックスを削除する

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

例：

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

注意：ベースインデックスを削除することはできません。

### ビットマップインデックスの変更

ビットマップインデックスは、次の操作をサポートしています：

#### ビットマップインデックスを作成する

構文：

```sql
 ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala'];
```

注意：

```plain text
1. ビットマップインデックスは現在のバージョンのみサポートされています。
2. BITMAP インデックスは、単一の列にのみ作成されます。
```

#### インデックスを削除する

構文：

```sql
DROP INDEX index_name;
```

### スワップ

スワップは、2つのテーブルの間でのアトミックな交換をサポートしています。

構文：

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手動コンパクション（v3.1 以降）

StarRocks は、異なるバージョンのロードされたデータをマージするためのコンパクションメカニズムを使用します。この機能は、小さなファイルを大きなファイルに結合することで、クエリのパフォーマンスを効果的に向上させます。

v3.1 より前では、コンパクションは以下の2つの方法で実行されます：

- システムによる自動コンパクション：コンパクションはバックグラウンドで BE レベルで実行されます。ユーザーはコンパクションのためにデータベースやテーブルを指定することはできません。
- ユーザーは、HTTP インターフェイスを呼び出すことでコンパクションを実行できます。

v3.1 からは、StarRocks はユーザーが SQL コマンドを実行して手動でコンパクションを実行するための SQL インターフェイスを提供します。特定のテーブルやパーティションを指定してコンパクションを行うことができます。これにより、コンパクションプロセスに対する柔軟性と制御が向上します。

構文：

```sql
-- テーブル全体でコンパクションを実行します。
ALTER TABLE <tbl_name> COMPACT

-- 単一のパーティションでコンパクションを実行します。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 複数のパーティションでコンパクションを実行します。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- 累積コンパクションを実行します。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- ベースコンパクションを実行します。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

`information_schema` データベースの `be_compactions` テーブルには、コンパクションの結果が記録されます。`SELECT * FROM information_schema.be_compactions;` を実行してコンパクション後のデータバージョンをクエリできます。

## 例

### テーブル

1. テーブルのデフォルトのレプリカ数を変更し、新しく追加されたパーティションのデフォルトのレプリカ数として使用します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("default.replication_num" = "2");
    ```

2. 単一パーティションテーブルの実際のレプリカ数を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replication_num" = "3");
    ```

3. レプリカ間でのデータの書き込みとレプリケーションモードを変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
    ```

   この例では、データの書き込みとレプリケーションモードを「リーダーレスレプリケーション」に設定し、データはプライマリとセカンダリのレプリカを区別せずに複数のレプリカに同時に書き込まれます。詳細については、[CREATE TABLE](CREATE_TABLE.md) の `replicated_storage` パラメータを参照してください。

### パーティション

1. バケティングモードをデフォルトで使用して、パーティションを追加します。既存のパーティションは [MIN, 2013-01-01) です。追加されるパーティションは [2013-01-01, 2014-01-01) です。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
    ```

2. 新しいバケット数を使用して、パーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    DISTRIBUTED BY HASH(k1);
    ```

3. 新しいレプリカ数を使用して、パーティションを追加します。

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

5. 指定されたパーティションのレプリカ数を一括で変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (p1, p2, p4) SET("replication_num"="1");
    ```
6. すべてのパーティションのストレージメディアを一括変更します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY PARTITION (*) SET("storage_medium"="HDD");
    ```

7. パーティションを削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP PARTITION p1;
    ```

8. 上限と下限境界を持つパーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
    ```

### ロールアップ

1. 基本インデックス（k1、k2、k3、v1、v2）に基づいてインデックス「example_rollup_index」を作成します。列ベースのストレージが使用されます。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
    PROPERTIES("storage_type"="column");
    ```

2. `example_rollup_index(k1,k3,v1,v2)` を基にインデックス「example_rollup_index2」を作成します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index2 (k1, v1)
    FROM example_rollup_index;
    ```

3. 基本インデックス（k1、k2、k3、v1）に基づいてインデックス「example_rollup_index3」を作成します。ロールアップのタイムアウト時間は1時間に設定されます。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index3(k1, k3, v1)
    PROPERTIES("storage_type"="column", "timeout" = "3600");
    ```

4. インデックス「example_rollup_index2」を削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP ROLLUP example_rollup_index2;
    ```

### スキーマ変更

1. `example_rollup_index` の`col1` カラムの後にキーカラム `new_col`（非集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

2. `example_rollup_index` の`col1` カラムの後に値カラム `new_col`（非集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

3. `example_rollup_index` の`col1` カラムの後にキーカラム `new_col`（集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

4. `example_rollup_index` の`col1` カラムの後に値カラム `new_col SUM`（集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

5. `example_rollup_index` に複数の列（集計）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
    TO example_rollup_index;
    ```

6. `example_rollup_index` に複数の列（集計）を追加し、追加された列の場所を `AFTER` で指定します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
    ADD COLUMN col2 FLOAT SUM AFTER `v2`,
    TO example_rollup_index;
    ```

7. `example_rollup_index` から列を削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    FROM example_rollup_index;
    ```

8. 基本インデックスの`col1` カラムの列型をBIGINTに変更し、`col2` の後に配置します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
    ```

9. 基本インデックスの `val1` カラムの最大長を32から64に変更します。元の長さは32です。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
    ```

10. `example_rollup_index` の列を並び替えます。元の列順は k1、k2、k3、v1、v2 です。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 2つの操作（ADD COLUMN と ORDER BY）を同時に実行します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. テーブルのbloomfilter列を変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("bloom_filter_columns"="k1,k2,k3");
     ```

     この操作は上記のスキーマ変更操作にもマージできます（複数の節の文法がわずかに異なることに注意してください）。

     ```sql
     ALTER TABLE example_db.my_table
     DROP COLUMN col2
     PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
     ```

13. テーブルのコロケーションプロパティを変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("colocate_with" = "t1");
     ```

14. テーブルのバケッティングモードをランダムディストリビューションからハッシュディストリビューションに変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("distribution_type" = "hash");
     ```

15. テーブルのダイナミックパーティションプロパティを変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("dynamic_partition.enable" = "false");
     ```

     既存のダイナミックパーティションプロパティが構成されていないテーブルにダイナミックパーティションプロパティを追加する場合は、すべてのダイナミックパーティションプロパティを指定する必要があります。

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

1. `table1` を `table2` に名前を変更します。

    ```sql
    ALTER TABLE table1 RENAME table2;
    ```

2. `example_table` のロールアップインデックス `rollup1` の名前を `rollup2` に変更します。

    ```sql
    ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
    ```

3. `example_table` のパーティション `p1` の名前を `p2` に変更します。

    ```sql
    ALTER TABLE example_table RENAME PARTITION p1 p2;
    ```

### インデックス

1. `table1` の列 `siteid` に対するビットマップインデックスを作成します。

    ```sql
    ALTER TABLE table1
    ADD INDEX index_1 (siteid) [USING BITMAP] COMMENT 'balabala';
    ```

2. `table1` の列 `siteid` のビットマップインデックスを削除します。

    ```sql
    ALTER TABLE table1
    DROP INDEX index_1;
    ```

### スワップ

`table1` と `table2` の間でアトミックスワップを行います。

```sql
ALTER TABLE table1 SWAP WITH table2
```

### 手動コンパクションの例

```sql
CREATE TABLE compaction_test( 
    event_day DATE,
    pv BIGINT)
DUPLICATE KEY(event_day)
PARTITION BY date_trunc('month', event_day)
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

## 参考

- [CREATE TABLE](./CREATE_TABLE.md)
- [SHOW CREATE TABLE](../data-manipulation/SHOW_CREATE_TABLE.md)
- [SHOW TABLES](../data-manipulation/SHOW_TABLES.md)
- [SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)
- [TABLE の削除](./DROP_TABLE.md)