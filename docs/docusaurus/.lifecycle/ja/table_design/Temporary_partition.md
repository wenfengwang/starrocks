```---
displayed_sidebar: "Japanese"
---

# 一時的なパーティション

このトピックでは、一時的なパーティション機能の使用方法について説明します。

定義済みのパーティションルールを持つパーティションテーブルに一時的なパーティションを作成し、これらの一時的なパーティションに新しいデータ分配戦略を定義することができます。一時的なパーティションは、パーティション内のデータをアトミックに上書きする場合や、パーティショニングおよびバケット戦略を調整する場合に、一時的なデータキャリアとして機能できます。一時的なパーティションでは、パーティション範囲、バケットの数、そしてレプリカの数、およびストレージメディアなどのデータ分配戦略を特定の要件に合わせてリセットすることができます。

一時的なパーティション機能は、以下のシナリオで使用できます：

- アトミックな上書き操作

  パーティション内のデータを再書き込む必要があり、その間にクエリが実行できるようにする場合、最初に元の正式なパーティションに基づいて一時的なパーティションを作成し、新しいデータを一時的なパーティションにロードし、その後、置換操作を使用して元の正式なパーティションを一時的なパーティションとアトミックに置き換えることができます。非パーティションのテーブルに関するアトミックな上書き操作については、[ALTER TABLE - SWAP](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md#swap)を参照してください。

- パーティションデータのクエリ同時実行の調整

  パーティションのバケット数を変更する必要がある場合、元の正式なパーティションと同じパーティション範囲で一時的なパーティションを最初に作成し、新しいバケット数を指定します。その後、`INSERT INTO`コマンドを使用して元の正式なパーティションのデータを一時的なパーティションにロードできます。最後に、元の正式なパーティションを一時的なパーティションとアトミックに置き換えることができます。

- パーティションルールの変更

  パーティショニング戦略を変更したい場合（パーティションのマージや大きなパーティションを複数の小さなパーティションに分割するなど）、期待されるマージまたは分割範囲で一時的なパーティションを最初に作成できます。その後、`INSERT INTO`コマンドを使用して元の正式なパーティションのデータを一時的なパーティションにロードできます。最後に、元の正式なパーティションを一時的なパーティションとアトミックに置き換えることができます。

## 一時的なパーティションの作成

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)コマンドを使用して、1回に複数のパーティションを作成できます。

### 構文

#### 1つの一時的なパーティションの作成

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

#### 1回に複数のパーティションを作成

```SQL
ALTER TABLE <table_name>
ADD TEMPORARY PARTITIONS START ("value1") END ("value2") EVERY {(INTERVAL <num> <time_unit>)|<num>}
[(partition_desc)]
[DISTRIBUTED BY HASH(<bucket_key>)];
```

### パラメータ

`partition_desc`：一時的なパーティションのバケット数およびレプリカ数、ストレージメディアなどのプロパティを指定します。

### 例

`site_access`テーブルに一時的なパーティション`tp1`を作成し、`VALUES [(...), (...)]`構文を使用してその範囲を`[2020-01-01, 2020-02-01)`と指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp1 VALUES [("2020-01-01"), ("2020-02-01"));
```

`site_access`テーブルに一時的なパーティション`tp2`を作成し、その上限を`2020-03-01`と指定します。StarRocksは前の一時的なパーティションの上限をこの一時的なパーティションの下限として使用し、左が閉じて右が開いた範囲`[2020-02-01, 2020-03-01)`の一時的なパーティションを生成します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp2 VALUES LESS THAN ("2020-03-01");
```

`site_access`テーブルに一時的なパーティション`tp3`を作成し、その上限を`2020-04-01`と指定し、レプリカ数を`1`と指定します。

```SQL
ALTER TABLE site_access
ADD TEMPORARY PARTITION tp3 VALUES LESS THAN ("2020-04-01")
 ("replication_num" = "1")
DISTRIBUTED BY HASH (site_id);
```

`site_access`テーブルに1度に複数のパーティション用に、パーティション間隔を月次にして範囲`[2020-04-01, 2021-01-01)`を指定して一時的なパーティションを作成します。

```SQL
ALTER TABLE site_access 
ADD TEMPORARY PARTITIONS START ("2020-04-01") END ("2021-01-01") EVERY (INTERVAL 1 MONTH);
```

### 使用上の注意

- 一時的なパーティションのパーティション列は、元の正式なパーティションに基づいて一時的なパーティションを作成する際のパーティション列と同じでなければなりません。変更できません。
- 一時的なパーティションの名前は、正式なパーティションまたは他の一時的なパーティションの名前と同じにすることはできません。
- テーブル内のすべての一時的なパーティションの範囲は重なることはできませんが、一時的なパーティションと正式なパーティションの範囲は重なることがあります。

## 一時的なパーティションの表示

[SHOW TEMPORARY PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md)コマンドを使用して一時的なパーティションを表示できます。

```SQL
SHOW TEMPORARY PARTITIONS FROM [db_name.]table_name [WHERE] [ORDER BY] [LIMIT]
```

## 一時的なパーティションへのデータのロード

`INSERT INTO`コマンド、STREAM LOAD、またはBROKER LOADを使用して、1つまたは複数の一時的なパーティションにデータをロードできます。

### `INSERT INTO`コマンドを使用してデータをロード

例：

```SQL
INSERT INTO site_access TEMPORARY PARTITION (tp1) VALUES ("2020-01-01",1,"ca","lily",4);
INSERT INTO site_access TEMPORARY PARTITION (tp2) SELECT * FROM site_access_copy PARTITION p2;
INSERT INTO site_access TEMPORARY PARTITION (tp3, tp4,...) SELECT * FROM site_access_copy PARTITION (p3, p4,...);
```

詳細な構文やパラメータの説明については、[INSERT INTO](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

### STREAM LOADを使用してデータをロード

例：

```bash
curl --location-trusted -u root: -H "label:123" -H "Expect:100-continue" -H "temporary_partitions: tp1, tp2, ..." -T testData \
    http://host:port/api/example_db/site_access/_stream_load    
```

詳細な構文やパラメータの説明については、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。

### BROKER LOADを使用してデータをロード

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

ここで、`StorageCredentialParams`は、選択した認証メソッドに応じて異なる認証パラメータのグループを表します。詳細な構文やパラメータの説明については、[BROKER LOAD](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md)を参照してください。

### ROUTINE LOADを使用してデータをロード

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

詳細な構文やパラメータの説明については、[CREATE ROUTINE LOAD](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md)を参照してください。

## 一時的なパーティションでのデータのクエリ

[SELECT](../sql-reference/sql-statements/data-manipulation/SELECT.md)文を使用して、指定された一時的なパーティション内のデータをクエリできます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1);

SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);

SELECT event_day,site_id,pv FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...);
```

JOIN句を使用して、2つのテーブルからの一時的なパーティション内のデータをクエリできます。

```SQL
SELECT * FROM
site_access TEMPORARY PARTITION (tp1, tp2, ...)
JOIN
site_access_copy TEMPORARY PARTITION (tp1, tp2, ...)
ON site_access.site_id=site_access1.site_id and site_access.event_day=site_access1.event_day;
```

## 元の正式なパーティションを一時的なパーティションで置き換える

[ALTER TABLE](../sql-reference/sql-statements/data-definition/ALTER_TABLE.md)文を使用して、元の正式なパーティションを一時的なパーティションで置き換えることで、新しい正式なパーティションを作成できます。

> **注意**
>```
> ALTER TABLE文で操作したオリジナルフォーマルパーティションと一時パーティションは削除され、復元できません。

### 構文

```SQL
ALTER TABLE table_name REPLACE PARTITION (partition_name) WITH TEMPORARY PARTITION (temporary_partition_name1, ...)
PROPERTIES ("key" = "value");
```

### パラメータ

- **strict_range**
  
  デフォルト値: `true`。

  このパラメータを`true`に設定すると、元のフォーマルパーティションの全ての範囲の結合が置換に使用される一時パーティションの範囲の結合と完全に一致する必要があります。このパラメータを`false`に設定すると、置換後の新しいフォーマルパーティションの範囲が他のフォーマルパーティションと重ならないようにするだけで十分です。

  - 例1:
  
    次の例では、元のフォーマルパーティション`p1`、`p2`、`p3`の結合が、`tp1`と`tp2`の一時パーティションの結合と同じであり、`tp1`と`tp2`を`p1`、`p2`、`p3`に置き換えることができます。

      ```plaintext
      # 元のフォーマルパーティションp1、p2、p3の範囲 => これらの範囲の結合
      [10, 20), [20, 30), [40, 50) => [10, 30), [40, 50)
      
      # 一時パーティションtp1とtp2の範囲 => これらの範囲の結合
      [10, 30), [40, 45), [45, 50) => [10, 30), [40, 50)
      ```

  - 例2:

    次の例では、元のフォーマルパーティションの範囲の結合が一時パーティションの範囲の結合と異なります。パラメータ`strict_range`の値が`true`の場合、一時パーティション`tp1`と`tp2`は元のフォーマルパーティション`p1`を置き換えることができません。値が`false`の場合、一時パーティションの範囲[10, 30)と[40, 50)が他のフォーマルパーティションと重ならないようにする限り、一時パーティションは元のフォーマルパーティションを置き換えることができます。

      ```plaintext
      # 元のフォーマルパーティションp1の範囲 => 範囲の結合
      [10, 50) => [10, 50)
      
      # 一時パーティションtp1とtp2の範囲 => これらの範囲の結合
      [10, 30), [40, 50) => [10, 30), [40, 50)
      ```

- **use_temp_partition_name**
  
  デフォルト値: `false`。

  置換に使用される一時パーティションの数が元のフォーマルパーティションの数と同じである場合、このパラメータを`false`に設定すると、置換後の新しいフォーマルパーティションの名前は変更されず、元の名前が維持されます。パラメータを`true`に設定すると、置換後の新しいフォーマルパーティションの名前として一時パーティションの名前が使用されます。

  次の例では、パラメータを`false`に設定すると、置換後の新しいフォーマルパーティションの名前は`p1`のままです。ただし、関連するデータとプロパティは一時パーティション`tp1`のデータとプロパティで置き換えられます。一方、パラメータを`true`に設定すると、置換後の新しいフォーマルパーティションの名前が`tp1`に変更されます。この場合、元のフォーマルパーティション`p1`は存在しなくなります。

    ```sql
    ALTER TABLE tbl1 REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
    ```

  置換対象のフォーマルパーティションの数が置換に使用される一時パーティションの数と異なる場合、このパラメータはデフォルト値`false`のままであると、このパラメータの`false`の値は無効です。

  次の例では、置換後の新しいフォーマルパーティションの名前が`tp1`に変更され、元のフォーマルパーティション`p1`と`p2`は存在しなくなります。

    ```SQL
    ALTER TABLE site_access REPLACE PARTITION (p1, p2) WITH TEMPORARY PARTITION (tp1);
    ```

### 例

元のフォーマルパーティション`p1`を一時パーティション`tp1`で置き換える。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p1) WITH TEMPORARY PARTITION (tp1);
```

元のフォーマルパーティション`p2`と`p3`を一時パーティション`tp2`と`tp3`で置き換える。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p2, p3) WITH TEMPORARY PARTITION (tp2, tp3);
```

元のフォーマルパーティション`p4`と`p5`を一時パーティション`tp4`と`tp5`で置き換え、パラメータ`strict_range`を`false`、`use_temp_partition_name`を`true`に指定します。

```SQL
ALTER TABLE site_access REPLACE PARTITION (p4, p5) WITH TEMPORARY PARTITION (tp4, tp5)
PROPERTIES (
    "strict_range" = "false",
    "use_temp_partition_name" = "true"
);
```

### 使用上の注意

- テーブルに一時パーティションが存在する場合、`ALTER`コマンドを使用してテーブルのSchema Change操作を行うことはできません。
- テーブルにSchema Change操作を行う際、テーブルに一時パーティションを追加することはできません。

## 一時パーティションの削除

次のコマンドを使用して一時パーティション`tp1`を削除します。

```SQL
ALTER TABLE site_access DROP TEMPORARY PARTITION tp1;
```

以下の制限に注意してください：

- `DROP`コマンドを使用してデータベースまたはテーブルを直接削除した場合、`RECOVER`コマンドを使用して限られた時間内にデータベースやテーブルを回復できます。ただし、一時パーティションは回復できません。
- `ALTER`コマンドを使用してフォーマルパーティションを削除した場合、限られた時間内に`RECOVER`コマンドを使用してフォーマルパーティションを回復できます。一時パーティションはフォーマルパーティションとは関連付けられていないため、一時パーティションの操作はフォーマルパーティションに影響しません。
- `ALTER`コマンドを使用して一時パーティションを削除した場合、`RECOVER`コマンドを使用して回復することはできません。
- テーブルのデータを削除するために`TRUNCATE`コマンドを使用すると、テーブルの一時パーティションが削除され、回復できません。
- フォーマルパーティション内のデータを削除するために`TRUNCATE`コマンドを使用すると、一時パーティションは影響を受けません。
- `TRUNCATE`コマンドは、一時パーティション内のデータを削除するために使用できません。