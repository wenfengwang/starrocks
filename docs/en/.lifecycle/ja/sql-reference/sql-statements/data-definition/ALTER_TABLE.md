---
displayed_sidebar: "Japanese"
---

# ALTER TABLE

## 説明

既存のテーブルを変更します。以下の操作を含みます。

- [テーブル、パーティション、インデックスの名前変更](#rename)
- [テーブルコメントの変更](#alter-table-comment-from-v31)
- [アトミックスワップ](#swap)
- [パーティションの追加/削除とパーティション属性の変更](#modify-partition)
- [スキーマ変更](#schema-change)
- [ロールアップインデックスの作成/削除](#modify-rollup-index)
- [ビットマップインデックスの変更](#modify-bitmap-indexes)
- [手動データバージョンの圧縮](#manual-compaction-from-31)

> **注意**
>
> この操作には、宛先テーブルに対するALTER権限が必要です。

## 構文

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
alter_clause1[, alter_clause2, ...]
```

`alter_clause`は、パーティション、ロールアップ、スキーマ変更、名前変更、インデックス、スワップ、コメント、および圧縮の6つの操作に分類されます。

- 名前変更: テーブル、ロールアップインデックス、またはパーティションの名前を変更します。**ただし、列名は変更できません。**
- コメント: テーブルコメントを変更します（**v3.1以降でサポート**）。
- スワップ: 2つのテーブルをアトミックに交換します。
- パーティション: パーティションのプロパティを変更したり、パーティションを削除したり、パーティションを追加したりします。
- スキーマ変更: 列の追加、削除、並べ替え、または型の変更を行います。
- ロールアップ: ロールアップインデックスを作成または削除します。
- インデックス: インデックスを変更します（ビットマップインデックスのみ変更可能）。
- 圧縮: ロードされたデータのバージョンをマージするための手動圧縮を実行します（**v3.1以降でサポート**）。

:::note

- スキーマ変更、ロールアップ、パーティション操作は、ALTER TABLEステートメントで一度に実行することはできません。
- スキーマ変更とロールアップは非同期操作です。タスクが送信された後、すぐに成功メッセージが返されます。進行状況を確認するには、[SHOW ALTER TABLE](../data-manipulation/SHOW_ALTER.md)コマンドを実行できます。
- パーティション、名前変更、スワップ、およびインデックスは同期操作であり、コマンドの返却は実行が完了したことを示します。
:::

### 名前変更

名前変更は、テーブル名、ロールアップインデックス名、およびパーティション名の変更をサポートしています。

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

### テーブルコメントの変更（v3.1以降）

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

1. Partition_descは、次の2つの式をサポートしています:

    ```plain
    VALUES LESS THAN [MAXVALUE|("value1", ...)]
    VALUES ("value1", ...), ("value1", ...)
    ```

2. パーティションは左閉右開区間です。ユーザーが右境界のみを指定した場合、システムは自動的に左境界を決定します。
3. バケットモードが指定されていない場合、ビルトインテーブルで使用されるバケットメソッドが自動的に使用されます。
4. バケットモードが指定されている場合、バケット番号のみを変更できます。バケットモードまたはバケット列を変更することはできません。
5. ユーザーは`["key"="value"]`でパーティションのいくつかのプロパティを設定できます。詳細については、[CREATE TABLE](CREATE_TABLE.md)を参照してください。

#### パーティションの削除

構文:

```sql
-- 2.0以前
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS | FORCE] <partition_name>
-- 2.0以降
ALTER TABLE [<db_name>.]<tbl_name>
DROP PARTITION [IF EXISTS] <partition_name> [FORCE]
```

注意:

1. パーティションテーブルでは、少なくとも1つのパーティションを保持しておく必要があります。
2. DROP PARTITIONを実行してからしばらくすると、RECOVERステートメントで削除されたパーティションを回復することができます。詳細については、RECOVERステートメントを参照してください。
3. DROP PARTITION FORCEを実行すると、パーティションは直接削除され、パーティション上の未完了のアクティビティがあるかどうかを確認せずに回復することはできません。したがって、通常はこの操作は推奨されません。

#### 一時パーティションの追加

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name> 
ADD TEMPORARY PARTITION [IF NOT EXISTS] <partition_name>
partition_desc ["key"="value"]
[DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num]]
```

#### 現在のパーティションを一時パーティションで置き換える

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

**使用法**

- パーティションの次のプロパティを変更できます:

  - storage_medium
  - storage_cooldown_ttlまたはstorage_cooldown_time
  - replication_num

- 1つのパーティションのみを持つテーブルの場合、パーティション名はテーブル名と同じです。テーブルが複数のパーティションに分割されている場合、すべてのパーティションのプロパティを変更するために`(*)`を使用できます。これはより便利です。

- 変更後のパーティションのプロパティを表示するには、`SHOW PARTITIONS FROM <tbl_name>`を実行します。

### スキーマ変更

スキーマ変更は、次の変更をサポートしています。

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

1. 集計テーブルに値列を追加する場合は、agg_typeを指定する必要があります。
2. 非集計テーブル（重複キーテーブルなど）にキーカラムを追加する場合は、KEYキーワードを指定する必要があります。
3. ベースインデックスに既に存在する列をロールアップインデックスに追加することはできません（必要な場合はロールアップインデックスを再作成できます）。

#### 複数の列を指定されたインデックスに追加

構文:

- 複数の列を追加

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value", ...)
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

- 複数の列を追加し、追加された列の位置をAFTERで指定する

  ```sql
  ALTER TABLE [<db_name>.]<tbl_name>
  ADD COLUMN (column_name1 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  ADD COLUMN (column_name2 column_type [KEY | agg_type] DEFAULT "default_value" AFTER (column_name))
  [TO rollup_index_name]
  [PROPERTIES ("key"="value", ...)]
  ```

注意:

1. 集計テーブルに値列を追加する場合は、agg_typeを指定する必要があります。
2. 非集計テーブルにキーカラムを追加する場合は、KEYキーワードを指定する必要があります。
3. ベースインデックスに既に存在する列をロールアップインデックスに追加することはできません（必要な場合は別のロールアップインデックスを作成できます）。

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

#### 指定されたインデックスの列の型と位置を変更

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
MODIFY COLUMN column_name column_type [KEY | agg_type] [NULL | NOT NULL] [DEFAULT "default_value"]
[AFTER column_name|FIRST]
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意:

1. 集計モデルの値列を変更する場合は、agg_typeを指定する必要があります。
2. 非集計モデルのキーカラムを変更する場合は、KEYキーワードを指定する必要があります。
3. 列の型のみを変更できます。列の他のプロパティは現在のままです（つまり、他のプロパティは元のプロパティに従って明示的にステートメントに書く必要があります。例8を参照）。
4. パーティション列は変更できません。
5. 現在、次の型の変換がサポートされています（ユーザーによって精度の損失が保証されます）。

   - TINYINT/SMALLINT/INT/BIGINTをTINYINT/SMALLINT/INT/BIGINT/DOUBLEに変換します。
   - TINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLE/DECIMALをVARCHARに変換します。VARCHARは最大長の変更をサポートしています。
   - VARCHARをTINTINT/SMALLINT/INT/BIGINT/LARGEINT/FLOAT/DOUBLEに変換します。
   - VARCHARをDATEに変換します（現在、6つの形式がサポートされています: "%Y-%m-%d"、"%y-%m-%d"、"%Y%m%d"、"%y%m%d"、"%Y/%m/%d、"%y/%m/%d"）。
   - DATETIMEをDATEに変換します（年月日の情報のみが保持されます。つまり、`2019-12-09 21:47:05` `<-->` `2019-12-09`）。
   - DATEをDATETIMEに変換します（時間、分、秒をゼロに設定します。例: `2019-12-09` `<-->` `2019-12-09 00:00:00`）。
   - FLOATをDOUBLEに変換します。
   - INTをDATEに変換します（INTデータの変換に失敗した場合、元のデータは変更されません）。

6. NULLからNOT NULLへの変換はサポートされていません。

#### 指定されたインデックスの列の順序を変更

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ORDER BY (column_name1, column_name2, ...)
[FROM rollup_index_name]
[PROPERTIES ("key"="value", ...)]
```

注意:

1. インデックスのすべての列を記述する必要があります。
2. 値列はキーカラムの後にリストアップされます。

#### 生成列の追加（v3.1以降）

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD COLUMN col_name data_type [NULL] AS generation_expr [COMMENT 'string']
```

生成列を追加し、その式を指定できます。[生成列](../generated_columns.md)は、式の結果を事前計算して格納することができ、同じ複雑な式を持つクエリのパフォーマンスを大幅に向上させます。v3.1以降、StarRocksは生成列をサポートしています。

#### テーブルプロパティの変更

現在、StarRocksは次のテーブルプロパティの変更をサポートしています。

- `replication_num`
- `default.replication_num`
- `storage_cooldown_ttl`
- `storage_cooldown_time`
- 動的パーティショニング関連のプロパティ
- `enable_persistent_index`
- `bloom_filter_columns`
- `colocate_with`

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SET ("key" = "value",...)
```

注意:
上記のスキーマ変更操作にマージしてプロパティを変更することもできます。以下の例を参照してください。

### ロールアップの変更

#### ロールアップインデックスの作成

構文:

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)]
```

PROPERTIES: タイムアウト時間を設定することができます。デフォルトのタイムアウト時間は1日です。

例:

```SQL
ALTER TABLE [<db_name>.]<tbl_name> 
ADD ROLLUP r1(col1,col2) from r0;
```

#### ロールアップインデックスを一括作成

構文:

```SQL
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP [rollup_name (column_name1, column_name2, ...)
[FROM from_index_name]
[PROPERTIES ("key"="value", ...)],...];
```

例:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
ADD ROLLUP r1(col1,col2) from r0, r2(col3,col4) from r0;
```

注意:

1. from_index_nameが指定されていない場合、デフォルトでベースインデックスから作成します。
2. ロールアップテーブルの列は、from_indexに存在する列である必要があります。
3. プロパティでは、ストレージ形式を指定できます。詳細については、CREATE TABLEを参照してください。

#### ロールアップインデックスの削除

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP rollup_name [PROPERTIES ("key"="value", ...)];
```

例:

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1;
```

#### ロールアップインデックスを一括削除

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
DROP ROLLUP [rollup_name [PROPERTIES ("key"="value", ...)],...];
```

例:

```sql
ALTER TABLE [<db_name>.]<tbl_name> DROP ROLLUP r1, r2;
```

注意: ベースインデックスを削除することはできません。

### ビットマップインデックスの変更

ビットマップインデックスは、次の変更をサポートしています。

#### ビットマップインデックスの作成

構文:

```sql
 ALTER TABLE [<db_name>.]<tbl_name>
ADD INDEX index_name (column [, ...],) [USING BITMAP] [COMMENT 'balabala'];
```

注意:

```plain text
1. ビットマップインデックスは、現在のバージョンのみサポートされています。
2. BITMAPインデックスは、単一の列にのみ作成されます。
```

#### インデックスの削除

構文:

```sql
DROP INDEX index_name;
```

### スワップ

2つのテーブルをアトミックに交換するスワップをサポートしています。

構文:

```sql
ALTER TABLE [<db_name>.]<tbl_name>
SWAP WITH <tbl_name>;
```

### 手動圧縮（v3.1以降）

StarRocksは、異なるバージョンのロードされたデータをマージするための圧縮メカニズムを使用しています。この機能により、小さなファイルを大きなファイルに結合することができ、クエリのパフォーマンスが劇的に向上します。

v3.1以前では、圧縮は次の2つの方法で実行されます。

- システムによる自動圧縮: バックエンドレベルでバックグラウンドで圧縮が実行されます。ユーザーは圧縮のためにデータベースまたはテーブルを指定することはできません。
- ユーザーはHTTPインターフェースを呼び出すことで圧縮を実行できます。

v3.1以降、StarRocksはユーザーがSQLコマンドを実行して手動で圧縮を実行するためのSQLインターフェースを提供しています。特定のテーブルまたはパーティションを選択して圧縮することができます。これにより、圧縮プロセスに対してより柔軟性と制御を持つことができます。

構文:

```sql
-- テーブル全体で圧縮を実行します。
ALTER TABLE <tbl_name> COMPACT

-- 単一のパーティションで圧縮を実行します。
ALTER TABLE <tbl_name> COMPACT <partition_name>

-- 複数のパーティションで圧縮を実行します。
ALTER TABLE <tbl_name> COMPACT (<partition1_name>[,<partition2_name>,...])

-- 累積圧縮を実行します。
ALTER TABLE <tbl_name> CUMULATIVE COMPACT (<partition1_name>[,<partition2_name>,...])

-- ベース圧縮を実行します。
ALTER TABLE <tbl_name> BASE COMPACT (<partition1_name>[,<partition2_name>,...])
```

`information_schema`データベースの`be_compactions`テーブルには、圧縮結果が記録されます。`SELECT * FROM information_schema.be_compactions;`を実行して圧縮後のデータバージョンをクエリできます。

## 例

### テーブル

1. テーブルのデフォルトのレプリカ数を変更します。これは、新しく追加されるパーティションのデフォルトのレプリカ数として使用されます。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("default.replication_num" = "2");
    ```

2. 単一パーティションテーブルの実際のレプリカ数を変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replication_num" = "3");
    ```

3. データの書き込みとレプリケーションモードをレプリカ間で変更します。

    ```sql
    ALTER TABLE example_db.my_table
    SET ("replicated_storage" = "false");
    ```

   この例では、データの書き込みとレプリケーションモードを「リーダーレスレプリケーション」に設定します。これは、データがプライマリとセカンダリのレプリカを区別せずに複数のレプリカに同時に書き込まれることを意味します。詳細については、[CREATE TABLE](CREATE_TABLE.md)を参照してください。

### パーティション

1. パーティションを追加し、デフォルトのバケットモードを使用します。既存のパーティションは[MIN, 2013-01-01)です。追加されたパーティションは[2013-01-01, 2014-01-01)です。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2014-01-01");
    ```

2. 新しいバケット数を使用してパーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES LESS THAN ("2015-01-01")
    DISTRIBUTED BY HASH(k1);
    ```

3. 新しいレプリカ数を使用してパーティションを追加します。

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

5. 指定されたパーティションのレプリカ数を一括変更します。

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

8. 上限と下限を持つパーティションを追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD PARTITION p1 VALUES [("2014-01-01"), ("2014-02-01"));
    ```

### ロールアップ

1. ベースインデックス（k1、k2、k3、v1、v2）に基づいて、インデックス`example_rollup_index`を作成します。列ベースのストレージが使用されます。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index(k1, k3, v1, v2)
    PROPERTIES("storage_type"="column");
    ```

2. `example_rollup_index(k1,k3,v1,v2)`に基づいて、インデックス`example_rollup_index2`を作成します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index2 (k1, v1)
    FROM example_rollup_index;
    ```

3. ベースインデックス（k1、k2、k3、v1）に基づいて、インデックス`example_rollup_index3`を作成します。ロールアップのタイムアウト時間を1時間に設定します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD ROLLUP example_rollup_index3(k1, k3, v1)
    PROPERTIES("storage_type"="column", "timeout" = "3600");
    ```

4. インデックス`example_rollup_index2`を削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP ROLLUP example_rollup_index2;
    ```

### スキーマ変更

1. `example_rollup_index`の`col1`列の後にキーカラム`new_col`（非集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT KEY DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

2. `example_rollup_index`の`col1`列の後に値列`new_col`（非集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

3. `example_rollup_index`の`col1`列の後にキーカラム`new_col`（集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

4. `example_rollup_index`の`col1`列の後に値列`new_col SUM`（集計列）を追加します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN new_col INT SUM DEFAULT "0" AFTER col1
    TO example_rollup_index;
    ```

5. `example_rollup_index`に複数の列を追加します（集計）。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN (col1 INT DEFAULT "1", col2 FLOAT SUM DEFAULT "2.3")
    TO example_rollup_index;
    ```

6. `example_rollup_index`に複数の列を追加し、`AFTER`を使用して追加された列の位置を指定します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN col1 INT DEFAULT "1" AFTER `k1`,
    ADD COLUMN col2 FLOAT SUM AFTER `v2`,
    TO example_rollup_index;
    ```

7. `example_rollup_index`から列を削除します。

    ```sql
    ALTER TABLE example_db.my_table
    DROP COLUMN col2
    FROM example_rollup_index;
    ```

8. ベースインデックスのcol1の列の型をBIGINTに変更し、col2の列の後に配置します。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN col1 BIGINT DEFAULT "1" AFTER col2;
    ```

9. ベースインデックスのval1の列の最大長を64に変更します。元の長さは32です。

    ```sql
    ALTER TABLE example_db.my_table
    MODIFY COLUMN val1 VARCHAR(64) REPLACE DEFAULT "abc";
    ```

10. `example_rollup_index`の列の順序を変更します。元の列の順序はk1、k2、k3、v1、v2です。

    ```sql
    ALTER TABLE example_db.my_table
    ORDER BY (k3,k1,k2,v2,v1)
    FROM example_rollup_index;
    ```

11. 2つの操作（ADD COLUMNとORDER BY）を同時に実行します。

    ```sql
    ALTER TABLE example_db.my_table
    ADD COLUMN v2 INT MAX DEFAULT "0" AFTER k2 TO example_rollup_index,
    ORDER BY (k3,k1,k2,v2,v1) FROM example_rollup_index;
    ```

12. テーブルのブルームフィルタ列を変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("bloom_filter_columns"="k1,k2,k3");
     ```

     この操作は、上記のスキーマ変更操作にマージすることもできます（複数の句の構文がわずかに異なることに注意してください）。

     ```sql
     ALTER TABLE example_db.my_table
     DROP COLUMN col2
     PROPERTIES ("bloom_filter_columns"="k1,k2,k3");
     ```

13. テーブルのColocateプロパティを変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("colocate_with" = "t1");
     ```

14. テーブルのバケットモードをランダムディストリビューションからハッシュディストリビューションに変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("distribution_type" = "hash");
     ```

15. テーブルの動的パーティショニングプロパティを変更します。

     ```sql
     ALTER TABLE example_db.my_table
     SET ("dynamic_partition.enable" = "false");
     ```

     動的パーティショニングプロパティを設定していないテーブルに動的パーティショニングプロパティを追加する場合は、すべての動的パーティショニングプロパティを指定する必要があります。

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

1. `table1`を`table2`に名前変更します。

    ```sql
    ALTER TABLE table1 RENAME table2;
    ```

2. `example_table`のロールアップインデックス`rollup1`を`rollup2`に名前変更します。

    ```sql
    ALTER TABLE example_table RENAME ROLLUP rollup1 rollup2;
    ```

3. `example_table`のパーティション`p1`を`p2`に名前変更します。

    ```sql
    ALTER TABLE example_table RENAME PARTITION p1 p2;
    ```

### インデックス

1. `table1`の`siteid`列にビットマップインデックスを作成します。

    ```sql
    ALTER TABLE table1
    ADD INDEX index_1 (siteid) [USING BITMAP] COMMENT 'balabala';
    ```

2. `table1`の`siteid`列のビットマップインデックスを削除します。

    ```sql
    ALTER TABLE table1
    DROP INDEX index_1;
    ```

### スワップ

`table1`と`table2`の間でアトミックなスワップを実行します。

```sql
ALTER TABLE table1 SWAP WITH table2
```

### 手動圧縮の例

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
- [DROP TABLE](./DROP_TABLE.md)
