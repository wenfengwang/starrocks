---
displayed_sidebar: "Japanese"
---

# プリペアドステートメント

v3.2以降、StarRocksは、同じ構造を持ち異なる変数で複数回SQLステートメントを実行するためのプリペアドステートメントを提供しています。この機能により、実行効率が大幅に向上し、SQLインジェクションを防止できます。

## 説明

プリペアドステートメントは基本的に次のように機能します。

1. **準備**: ユーザーは変数がプレースホルダー`?`で表されるSQLステートメントを準備します。FEはSQLステートメントを解析し、実行計画を生成します。
2. **実行**: 変数を宣言した後、ユーザーはこれらの変数をステートメントに渡し、ステートメントを実行します。ユーザーは異なる変数で同じステートメントを複数回実行できます。

**利点**

- **解析のオーバーヘッドを節約**: 実世界のビジネスシナリオでは、アプリケーションが同じ構造を持ち異なる変数でステートメントを複数回実行することがよくあります。プリペアドステートメントがサポートされることで、StarRocksは準備段階でステートメントを1回だけ解析する必要があります。異なる変数で同じステートメントを後続して実行する場合、事前に生成された解析結果を直接使用できます。そのため、特に複雑なクエリの場合、ステートメントの実行パフォーマンスが大幅に向上します。
- **SQLインジェクション攻撃を防ぐ**: ステートメントを変数から分離し、ユーザー入力データをパラメータとして直接変数をステートメントに連結するのではなく渡すことで、StarRocksは悪意のあるユーザーが悪意のあるSQLコードを実行するのを防ぐことができます。

**使用方法**

プリペアドステートメントは現在のセッションでのみ有効であり、他のセッションでは使用できません。現在のセッションが終了すると、そのセッションで作成されたプリペアドステートメントは自動的に削除されます。

## 構文

プリペアドステートメントの実行は次の段階で行われます。

- PREPARE: プレースホルダー`?`で変数を表したステートメントを準備します。
- SET: ステートメント内で変数を宣言します。
- EXECUTE: 宣言した変数をステートメントに渡し、ステートメントを実行します。
- DROP PREPAREまたはDEALLOCATE PREPARE: プリペアドステートメントを削除します。

### PREPARE

**構文:**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**パラメータ:**

- `stmt_name`: プリペアドステートメントに付けられた名前で、後でそのプリペアドステートメントを実行または解除するために使用されます。名前は単一のセッション内で一意である必要があります。
- `preparable_stmt`: プリペアドするSQLステートメントで、変数のプレースホルダーは`?`です。

**例:**

特定の値をプレースホルダー`?`で表現した`INSERT INTO VALUES()`ステートメントを準備します。

```SQL
PREPARE insert_stmt FROM 'INSERT INTO users (id, country, city, revenue) VALUES (?, ?, ?, ?)';
```

### SET

**構文:**

```SQL
SET @var_name = expr [, ...];
```

**パラメータ:**

- `var_name`: ユーザー定義変数の名前です。
- `expr`: ユーザー定義変数です。

**例:**

4つの変数を宣言します。

```SQL
SET @id1 = 1, @country1 = 'USA', @city1 = 'New York', @revenue1 = 1000000;
```

詳細は[ユーザー定義変数](../../reference/user_defined_variables.md)を参照してください。

### EXECUTE

**構文:**

```SQL
EXECUTE <stmt_name> [USING @var_name [, @var_name] ...]
```

**パラメータ:**

- `var_name`: `SET`ステートメントで宣言された変数の名前です。
- `stmt_name`: プリペアドステートメントの名前です。

**例:**

変数を`INSERT`ステートメントに渡し、そのステートメントを実行します。

```SQL
EXECUTE insert_stmt USING @id1, @country1, @city1, @revenue1;
```

### DROP PREPAREまたはDEALLOCATE PREPARE

**構文:**

```SQL
{DEALLOCATE | DROP} PREPARE <stmt_name>
```

**パラメータ:**

- `stmt_name`: プリペアドステートメントの名前です。

**例:**

プリペアドステートメントを削除します。

```SQL
DROP PREPARE insert_stmt;
```

## 例

### プリペアドステートメントの使用

次の例では、StarRocksテーブルからデータを挿入、削除、更新、およびクエリするためにプリペアドステートメントを使用する方法を示しています。

次のように名前が`demo`のデータベースと`users`のテーブルが作成されているとします。

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

    -- IDが1または2のデータを別々にクエリします。
    EXECUTE select_by_id_stmt USING @id1;
    EXECUTE select_by_id_stmt USING @id2;

    -- ID 1の行の一部を更新します。収益カラムのみを更新します。
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

次の例では、JavaアプリケーションがJDBCドライバを使用してStarRocksテーブルからデータを挿入、削除、更新、およびクエリする方法を示しています。

1. JDBCでStarRocksの接続URLを指定する際に、サーバーサイドのプリペアドステートメントを有効にする必要があります。

    ```Plaintext
    jdbc:mysql://<fe_ip>:<fe_query_port>/useServerPrepStmts=true
    ```

2. StarRocks GitHubプロジェクトは、[Javaコードの例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)を提供しており、JDBCドライバを介してStarRocksテーブルからデータを挿入、削除、更新、およびクエリする方法について説明しています。