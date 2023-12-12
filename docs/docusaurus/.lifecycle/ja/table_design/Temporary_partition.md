---
displayed_sidebar: "Japanese"
---

# 一時分割

このトピックでは、一時分割機能の使用方法について説明します。

定義済みの分割ルールを持つパーティションテーブルに一時的なパーティションを作成し、これらの一時的なパーティションに対して新しいデータ分散戦略を定義することができます。 一時的なパーティションは、パーティション内のデータを原子的に上書きする際やパーティショニングおよびバケット化戦略を調整する際に一時的なデータキャリアとして機能します。 一時的なパーティションに対して、パーティションの範囲、バケットの数、およびレプリカの数、ストレージメディアなどのデータ分散戦略をリセットして、特定の要件を満たすことができます。

次のシナリオで一時パーティション機能を使用できます：

- 原子的な上書き操作

  パーティション内のデータを書き換える際に、データが書き換えプロセス中にクエリできるようにする必要がある場合、元の正式なパーティションに基づいて最初に一時的なパーティションを作成し、新しいデータを一時的なパーティションにロードします。 次に、元の正式なパーティションを一時的なパーティションで原子的に置き換えるために replace 操作を使用できます。 非パーティションテーブルでの原子上書き操作については、[ALTER TABLE - SWAP](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#swap)を参照してください。

- パーティションデータのクエリ並行性の調整

  パーティションのバケツ数を変更する必要がある場合、元の正式なパーティションと同じパーティション範囲を持つ一時的なパーティションをまず作成し、新しいバケツ数を指定します。 次に、INSERT INTO コマンドを使用して元の正式なパーティションのデータを一時的なパーティションにロードできます。 最後に、元の正式なパーティションを一時的なパーティションで原子的に置き換えるために replace 操作を使用できます。

- パーティションのルールの変更

  パーティション戦略を変更したい場合、パーティションをマージしたり大きなパーティションを複数の小さなパーティションに分割したい場合、期待されるマージ済みまたは分割済み範囲を持つ一時的なパーティションをまず作成できます。 次に、INSERT INTO コマンドを使用して元の正式なパーティションのデータを一時的なパーティションにロードできます。 最後に、元の正式なパーティションを一時的なパーティションで原子的に置き換えるために replace 操作を使用できます。

## 一時的なパーティションを作成する

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) コマンドを使用して一度に1つ以上のパーティションを作成できます。

### 構文

#### 1 つの一時的なパーティションを作成する

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES [("value1"), {MAXVALUE|("value2")})]
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
ALTER TABLE <table_name> 
ADD TEMPORARY PARTITION <temporary_partition_name> VALUES LESS THAN {MAXVALUE|(<"value">)}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

#### 一度に複数のパーティションを作成する

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITIONS START ("value1") END ("value2") EVERY {(INTERVAL <num> <time_unit>)|<num>}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

### パラメータ

`partition_desc`：一時的なパーティションのバケツ数およびレプリカの数、ストレージメディアなどのプロパティを指定します。

### 例

`site_access` テーブルにおいて、一時的なパーティション `tp1` を作成し、その範囲を `2020-01-01` から `2020-02-01` までとして、 `VALUES [(...), (...)]` 構文を使用します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp1 VALUES [("2020-01-01"), ("2020-02-01"));
```

`site_access` テーブルにおいて、一時的なパーティション `tp2` を作成し、その上限を `2020-03-01` として、 `VALUES LESS THAN (...)` 構文を使用します。 StarRocks は前の一時的なパーティションの上限をこの一時的なパーティションの下限として使用し、左側を閉じた範囲で `[2020-02-01, 2020-03-01)` の一時的なパーティションを生成します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp2 VALUES LESS THAN ("2020-03-01");
```

`site_access` テーブルにおいて、一時的なパーティション `tp3` を作成し、その上限を `2020-04-01` として、 `VALUES LESS THAN (...)` 構文を使用し、レプリカ数を `1` として指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp3 VALUES LESS THAN ("2020-04-01")
 ("replication_num" = "1")
DISTRIBUTED BY HASH (site_id);
```

`site_access` テーブルにおいて、 `START ("2020-04-01") END ("2021-01-01") EVERY (INTERVAL 1 MONTH)` 構文を使用して、一度に複数のパーティションを作成します。

```SQL
ALTER TABLE site_access 
ADD TEMPORARY PARTITIONS START ("2020-04-01") END ("2021-01-01") EVERY (INTERVAL 1 MONTH);
```

### 使用上の注意

- 一時的なパーティションのパーティション列は、一時的なパーティションを作成する元の正式なパーティションと同じでなければならず、変更できません。
- 一時的なパーティションの名前は、正式なパーティションや他の一時的なパーティションの名前と同じにすることはできません。
- テーブル内のすべての一時的なパーティションの範囲は重複してはならず、一時的なパーティションと正式なパーティションの範囲は重複しても構いません。

## 一時的なパーティションを表示する

[SHOW TEMPORARY PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md) コマンドを使用して一時的なパーティションを表示できます。

```SQL
SHOW TEMPORARY PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

## 一時的なパーティションにデータをロードする

`INSERT INTO` コマンド、STREAM LOAD、または BROKER LOADを使用して、1つ以上の一時的なパーティションにデータをロードできます。

### `INSERT INTO` コマンドを使用してデータをロードする

例：

```SQL
INSERT INTO site_access TEMPORARY PARTITION (tp1) VALUES ("2020-01-01",1,"ca","lily",4);
INSERT INTO site_access TEMPORARY PARTITION (tp2) SELECT * FROM site_access_copy PARTITION p2;
INSERT INTO site_access TEMPORARY PARTITION (tp3, tp4,...) SELECT * FROM site_access_copy PARTITION (p3, p4,...);
```

構文とパラメータの詳細については、「[INSERT INTO](../sql-reference/sql-statements/data-manipulation/INSERT.md)」を参照してください。

### STREAM LOAD を使用してデータをロードする

例：

```bash
curl --location-trusted -u root: -H "label:123" -H "Expect:100-continue" -H "temporary_partitions: tp1, tp2, ..." -T testData \
    http://host:port/api/example_db/site_access/_stream_load    
```

構文とパラメータの詳細については、「[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)」を参照してください。

### BROKER LOAD を使用してデータをロードする

例：

```SQL
LOAD LABEL example_db.label1
(
    DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starrocks/data/input/file")
    INTO TABLE my_table
    TEMPORARY PARTITION (tp1, tp2, ...)
    ...
)
WITH BROKER
(
    StorageCredentialParams
);
```

注意：`StorageCredentialParams` は、選択した認証方法に応じて異なる認証パラメータのグループを表します。 構文とパラメータの詳細については、「[Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)」を参照してください。

### ROUTINE LOAD を使用してデータをロードする

例：

```SQL
CREATE ROUTINE LOAD example_db.site_access ON example_tbl
COLUMNS(col, col2,...),
TEMPORARY PARTITIONS(tp1, tp2, ...)
FROM KAFKA
(
    "kafka_broker_list" ="<kafka_broker1_ip>:<kafka_broker1_port>,<kafka_broker2_ip>:<kafka_broker2_port>",
    "kafka_topic" = "ordertest"
);
```

構文とパラメータの詳細については、「[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)」を参照してください。

## 一時的なパーティション内のデータをクエリする

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md) 文を使用して、指定された一時的なパーティション内のデータをクエリできます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1);

SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);

SELECT event_day,site_id,pv FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);
```

JOIN 句を使用して、2つのテーブルから一時的なパーティション内のデータをクエリできます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...)
JOIN
site_access_copy TEMPORARY PARTITION (tp1, tp2, ...)
ON site_access.site_id=site_access1.site_id and site_access.event_day=site_access1.event_day;
```

## 元の正式なパーティションを一時的なパーティションで置き換える

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md) 文を使用して、元の正式なパーティションを一時的なパーティションで置き換え、新しい正式なパーティションを作成できます。

> **注意**
> ALTER TABLE ステートメントで操作した元の形式パーティションと一時パーティションは削除され、回復できません。

### 構文

```SQL
ALTER TABLE table_name REPLACE PARTITION (partition_name) WITH TEMPORARY PARTITION (temporary_partition_name1, ...)
PROPERTIES ("key" = "value");
```

### パラメータ

- **strict_range**
  
  デフォルト値：`true`。

  このパラメータを`true`に設定すると、元の形式パーティションのすべての範囲の合併が置換に使用される一時パーティションの範囲の合併と完全に一致する必要があります。 このパラメータを`false`に設定すると、置換後の新しい形式パーティションの範囲が他の形式パーティションと重複しないことだけを確認する必要があります。

  - 例1：
  
    次の例では、元の形式パーティション`p1`、`p2`、`p3`の合併が一時パーティション`tp1`と`tp2`の合併と同じであり、`tp1`と`tp2`を`p1`、`p2`、`p3`に置換できます。

      ```plaintext
      # 元の形式パーティションp1、p2、p3の範囲 => これらの範囲の合併
      [10, 20), [20, 30), [40, 50) => [10, 30), [40, 50)
      
      # 一時パーティションtp1とtp2の範囲 => これらの範囲の合併
      [10, 30), [40, 45), [45, 50) => [10, 30), [40, 50)
      ```

  - 例2：

    次の例では、元の形式パーティションの範囲の合併が一時パーティションの範囲の合併と異なります。パラメータ`strict_range`の値が`true`に設定されている場合、一時パーティション`tp1`と`tp2`は元の形式パーティション`p1`を置換できません。値が`false`に設定されており、一時パーティションの範囲[10, 30)および[40, 50)が他の形式パーティションと重複しない場合、一時パーティションは元の形式パーティションに置換できます。

      ```plaintext
      # 元の形式パーティションp1の範囲 => 範囲の合併
      [10, 50) => [10, 50)
      
      # 一時パーティションtp1とtp2の範囲 => これらの範囲の合併
      [10, 30), [40, 50) => [10, 30), [40, 50)
      ```

- **use_temp_partition_name**
  
  デフォルト値：`false`。

  置換に使用される一時パーティションの数が元の形式パーティションの数と同じである場合、このパラメータを`false`に設定すると、置換後の新しい形式パーティションの名前は変更されません。このパラメータを`true`に設定すると、置換後の新しい形式パーティションの名前として一時パーティションの名前が使用されます。

  次の例では、パラメータが`false`に設定されている場合、置換後の新しい形式パーティションのパーティション名は`p1`のままです。ただし、関連するデータとプロパティは一時パーティション`tp1`のデータとプロパティに置換されます。パラメータを`true`に設定すると、置換後の新しい形式パーティションのパーティション名は`tp1`に変更されます。元の形式パーティション`p1`は存在しなくなります。

    ```sql
    ALTER TABLE tbl1 REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
    ```

  置換する形式パーティションの数が置換に使用される一時パーティションの数と異なる場合、かつこのパラメータがデフォルト値`false`のままの場合、このパラメータの値を`false`に設定することは無効です。

  次の例では、置換後、新しい形式パーティションの名前が`tp1`に変更され、元の形式パーティション`p1`と`p2`は存在しなくなります。

    ```SQL
    ALTER TABLE site_access REPLACE PARTITION (p1, p2) WITH TEMPORARY PARTITION (tp1);
    ```

### 例

元の形式パーティション`p1`を一時パーティション`tp1`で置換します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
```

元の形式パーティション`p2`および`p3`を一時パーティション`tp2`および`tp3`で置換します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p2, p3) WITH TEMPORARY PARTITION (tp2, tp3);
```

元の形式パーティション`p4`および`p5`を一時パーティション`tp4`および`tp5`で置換し、パラメータ`strict_range`を`false`、`use_temp_partition_name`を`true`と指定します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p4, p5) WITH TEMPORARY PARTITION (tp4, tp5)
PROPERTIES (
    "strict_range" = "false",
    "use_temp_partition_name" = "true"
);
```

### 使用上の注意

- テーブルに一時パーティションがある場合、`ALTER`コマンドを使用してテーブルのスキーマ変更操作を実行することはできません。
- テーブルのスキーマ変更操作を実行する場合、一時パーティションをテーブルに追加することはできません。

## 一時パーティションの削除

次のコマンドを使用して一時パーティション`tp1`を削除します。

```SQL
ALTER TABLE site_access DROP TEMPORARY PARTITION tp1;
```

次の制限に注意してください：

- `DROP`コマンドを使用してデータベースまたはテーブルを直接削除すると、`RECOVER`コマンドを使用して一定期間内にデータベースまたはテーブルを回復できます。ただし、一時パーティションは回復できません。
- `ALTER`コマンドを使用して形式パーティションを直接削除した後、`RECOVER`コマンドを使用して一定期間内に回復できます。一時パーティションは形式パーティションとは関連付けられていないため、一時パーティションの操作は形式パーティションに影響しません。
- `ALTER`コマンドを使用して一時パーティションを直接削除した後、`RECOVER`コマンドを使用して回復することはできません。
- テーブルのデータを削除するために`TRUNCATE`コマンドを使用すると、テーブルの一時パーティションが削除され、回復できません。
- 形式パーティションのデータを削除するために`TRUNCATE`コマンドを使用すると、一時パーティションには影響しません。
- `TRUNCATE`コマンドは一時パーティションのデータを削除するために使用できません。