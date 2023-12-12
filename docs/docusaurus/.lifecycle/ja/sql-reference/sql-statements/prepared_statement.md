---
displayed_sidebar: "Japanese"
---

# プリペアドステートメント

v3.2以降、StarRocksではプリペアドステートメントを提供しており、同じ構造で異なる変数を使用して複数回SQLステートメントを実行できます。この機能により、実行効率が大幅に向上し、SQLインジェクションを防ぐことができます。

## 説明

プリペアドステートメントは基本的に以下のように機能します。

1. **準備**: ユーザーは変数がプレースホルダー`?`で表されるSQLステートメントを準備します。フロントエンドはSQLステートメントを解析し、実行計画を生成します。
2. **実行**: 変数を宣言した後、ユーザーはこれらの変数をステートメントに渡し、ステートメントを実行します。ユーザーは異なる変数で同じステートメントを複数回実行できます。

**利点**

- **解析のオーバーヘッドを省略**: 実際のビジネスシナリオでは、アプリケーションはしばしば同じ構造で異なる変数を使用してステートメントを複数回実行します。利用可能なプリペアドステートメントをサポートすることで、StarRocksは準備フェーズ中にステートメントを一度だけ解析する必要があります。異なる変数で同じステートメントを実行する場合、事前に生成された解析結果を直接使用できます。そのため、特に複雑なクエリに対してステートメントの実行パフォーマンスが大幅に向上します。
- **SQLインジェクション攻撃を防止**: ステートメントを変数から分離し、ユーザー入力データをパラメータとして渡すことで、悪意のあるユーザーが悪意のあるSQLコードを実行するのを防ぐことができます。

**使用方法**

プリペアドステートメントは現在のセッションでのみ有効であり、他のセッションでは使用できません。現在のセッションが終了すると、そのセッションで作成されたプリペアドステートメントは自動的に削除されます。

## 構文

プリペアドステートメントの実行は以下のフェーズで構成されています。

- PREPARE: プレースホルダー`?`で変数が表されるステートメントを準備する。
- SET: ステートメント内で変数を宣言する。
- EXECUTE: 宣言した変数をステートメントに渡し、それを実行する。
- DROP PREPAREまたはDEALLOCATE PREPARE: プリペアドステートメントを削除する。

### PREPARE

**構文:**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**パラメータ:**

- `stmt_name`: プリペアドステートメントに与えられた名前であり、その後そのプリペアドステートメントを実行または解除する際に使用されます。名前は1つのセッション内で一意である必要があります。
- `preparable_stmt`: プレースホルダー`?`で変数が表される準備するSQLステートメントです。

**例:**

具体的な値をプレースホルダー`?`で表される`INSERT INTO VALUES()`ステートメントを準備します。

```SQL
PREPARE insert_stmt FROM 'INSERT INTO users (id, country, city, revenue) VALUES (?, ?, ?, ?)';
```

### SET

**構文:**

```SQL
SET @var_name = expr [, ...];
```

**パラメータ:**

- `var_name`: ユーザー定義変数の名前。
- `expr`: ユーザー定義変数。

**例:** 4つの変数を宣言します。

```SQL
SET @id1 = 1, @country1 = 'USA', @city1 = 'New York', @revenue1 = 1000000;
```

詳細については、[ユーザー定義変数](../../reference/user_defined_variables.md)を参照してください。

### EXECUTE

**構文:**

```SQL
EXECUTE <stmt_name> [USING @var_name [, @var_name] ...]
```

**パラメータ:**

- `var_name`: `SET`ステートメントで宣言された変数の名前。
- `stmt_name`: プリペアドステートメントの名前。

**例:**

`INSERT`ステートメントに変数を渡し、そのステートメントを実行します。

```SQL
EXECUTE insert_stmt USING @id1, @country1, @city1, @revenue1;
```

### DROP PREPAREまたはDEALLOCATE PREPARE

**構文:**

```SQL
{DEALLOCATE | DROP} PREPARE <stmt_name>
```

**パラメータ:**

- `stmt_name`: プリペアドステートメントの名前。

**例:**

プリペアドステートメントを削除します。

```SQL
DROP PREPARE insert_stmt;
```

## 例

### プリペアドステートメントの使用

以下の例では、StarRocksテーブルからのデータの挿入、削除、更新、およびクエリにプリペアドステートメントを使用する方法を示しています。

次のような`demo`という名前のデータベースと`users`という名前のテーブルが既に作成されているとします。

```SQL
CREATE DATABASE IF NOT EXISTS demo;
USE demo;
CREATE TABLE users (
  id BIGINT NOT NULL,
  country STRING,
  city STRING,
  revenue BIGINT
)
PRIMARY KEY (id)
DISTRIBUTED BY HASH(id);
```

1. 実行するためのステートメントを準備します。

    ```SQL
    PREPARE insert_stmt FROM 'INSERT INTO users (id, country, city, revenue) VALUES (?, ?, ?, ?)';
    PREPARE select_all_stmt FROM 'SELECT * FROM users';
    PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
    PREPARE update_stmt FROM 'UPDATE users SET revenue = ? WHERE id = ?';
    PREPARE delete_stmt FROM 'DELETE FROM users WHERE id = ?';
    ```

2. これらのステートメントで変数を宣言します。

    ```SQL
    SET @id1 = 1, @id2 = 2;
    SET @country1 = 'USA', @country2 = 'Canada';
    SET @city1 = 'New York', @city2 = 'Toronto';
    SET @revenue1 = 1000000, @revenue2 = 1500000, @revenue3 = (SELECT (revenue) * 1.1 FROM users);
    ```

3. 宣言した変数を使用してステートメントを実行します。

    ```SQL
    -- データを2行挿入します。
    EXECUTE insert_stmt USING @id1, @country1, @city1, @revenue1;
    EXECUTE insert_stmt USING @id2, @country2, @city2, @revenue2;

    -- テーブルからすべてのデータをクエリします。
    EXECUTE select_all_stmt;

    -- IDが1または2のデータを個別にクエリします。
    EXECUTE select_by_id_stmt USING @id1;
    EXECUTE select_by_id_stmt USING @id2;

    -- IDが1の行を部分的に更新します。収益の列のみを更新します。
    EXECUTE update_stmt USING @revenue3, @id1;

    -- IDが1のデータを削除します。
    EXECUTE delete_stmt USING @id1;

    -- プリペアドステートメントを削除します。
    DROP PREPARE insert_stmt;
    DROP PREPARE select_all_stmt;
    DROP PREPARE select_by_id_stmt;
    DROP PREPARE update_stmt;
    DROP PREPARE delete_stmt;
    ```

### Javaアプリケーションでのプリペアドステートメントの使用

以下の例では、JavaアプリケーションがJDBCドライバを使用してStarRocksテーブルからのデータの挿入、削除、更新、およびクエリにどのように使用されるかを示しています。

1. JDBCでStarRocksの接続URLを指定する際、サーバーサイドのプリペアドステートメントを有効にする必要があります。

    ```Plaintext
    jdbc:mysql://<fe_ip>:<fe_query_port>/useServerPrepStmts=true
    ```

2. StarRocksのGitHubプロジェクトでは、JDBCドライバを介してStarRocksテーブルにデータを挿入、削除、更新、およびクエリする方法を説明する[Javaコード例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)が提供されています。