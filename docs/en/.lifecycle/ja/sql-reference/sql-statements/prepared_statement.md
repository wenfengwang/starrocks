---
displayed_sidebar: "Japanese"
---

# プリペアドステートメント

v3.2以降、StarRocksはプリペアドステートメントを提供し、同じ構造を持つが異なる変数を使用して複数回SQLステートメントを実行することができます。この機能により、実行効率が大幅に向上し、SQLインジェクションを防ぐことができます。

## 説明

プリペアドステートメントは基本的に以下のように動作します。

1. **準備**: ユーザーは変数がプレースホルダー `?` で表されるSQLステートメントを準備します。FEはSQLステートメントを解析し、実行計画を生成します。
2. **実行**: 変数を宣言した後、ユーザーはこれらの変数をステートメントに渡して実行します。ユーザーは異なる変数を使用して同じステートメントを複数回実行することができます。

**利点**

- **パースのオーバーヘッドを削減**: 実際のビジネスシナリオでは、アプリケーションはしばしば同じ構造を持つが異なる変数を使用してステートメントを複数回実行します。プリペアドステートメントがサポートされている場合、StarRocksは準備フェーズでステートメントを1回だけパースする必要があります。異なる変数を使用して同じステートメントを実行する場合、事前に生成されたパース結果を直接使用することができます。そのため、ステートメントの実行パフォーマンスが大幅に向上し、特に複雑なクエリの場合に効果があります。
- **SQLインジェクション攻撃を防止**: ステートメントと変数を分離し、ユーザーの入力データをパラメータとして渡すことで、StarRocksは悪意のあるユーザーが悪意のあるSQLコードを実行することを防ぐことができます。

**使用方法**

プリペアドステートメントは現在のセッションでのみ有効であり、他のセッションでは使用することができません。現在のセッションが終了すると、そのセッションで作成されたプリペアドステートメントは自動的に削除されます。

## 構文

プリペアドステートメントの実行は以下のフェーズで構成されます。

- PREPARE: 変数がプレースホルダー `?` で表されるステートメントを準備します。
- SET: ステートメント内で変数を宣言します。
- EXECUTE: 宣言された変数をステートメントに渡して実行します。
- DROP PREPAREまたはDEALLOCATE PREPARE: プリペアドステートメントを削除します。

### PREPARE

**構文:**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**パラメータ:**

- `stmt_name`: プリペアドステートメントに与えられる名前であり、その後の実行または解放に使用されます。名前はセッション内で一意である必要があります。
- `preparable_stmt`: プリペアするSQLステートメントであり、変数のプレースホルダーは疑問符 (`?`) です。

**例:**

特定の値をプレースホルダー `?` で表す `INSERT INTO VALUES()` ステートメントを準備します。

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

- `var_name`: `SET` ステートメントで宣言された変数の名前です。
- `stmt_name`: プリペアドステートメントの名前です。

**例:**

変数を `INSERT` ステートメントに渡してそのステートメントを実行します。

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

以下の例では、StarRocksテーブルからデータを挿入、削除、更新、クエリするためにプリペアドステートメントを使用する方法を示しています。

次のデータベース `demo` とテーブル `users` が既に作成されているとします。

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

1. 実行のためにステートメントを準備します。

    ```SQL
    PREPARE insert_stmt FROM 'INSERT INTO users (id, country, city, revenue) VALUES (?, ?, ?, ?)';
    PREPARE select_all_stmt FROM 'SELECT * FROM users';
    PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
    PREPARE update_stmt FROM 'UPDATE users SET revenue = ? WHERE id = ?';
    PREPARE delete_stmt FROM 'DELETE FROM users WHERE id = ?';
    ```

2. ステートメント内で変数を宣言します。

    ```SQL
    SET @id1 = 1, @id2 = 2;
    SET @country1 = 'USA', @country2 = 'Canada';
    SET @city1 = 'New York', @city2 = 'Toronto';
    SET @revenue1 = 1000000, @revenue2 = 1500000, @revenue3 = (SELECT (revenue) * 1.1 FROM users);
    ```

3. 宣言された変数を使用してステートメントを実行します。

    ```SQL
    -- データを2行挿入します。
    EXECUTE insert_stmt USING @id1, @country1, @city1, @revenue1;
    EXECUTE insert_stmt USING @id2, @country2, @city2, @revenue2;

    -- テーブルからすべてのデータをクエリします。
    EXECUTE select_all_stmt;

    -- IDが1または2のデータを個別にクエリします。
    EXECUTE select_by_id_stmt USING @id1;
    EXECUTE select_by_id_stmt USING @id2;

    -- IDが1の行を部分的に更新します。revenue列のみ更新します。
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

以下の例では、JavaアプリケーションがJDBCドライバを使用してStarRocksテーブルからデータを挿入、削除、更新、クエリする方法を示しています。

1. JDBCでStarRocksの接続URLを指定する際に、サーバーサイドのプリペアドステートメントを有効にする必要があります。

    ```Plaintext
    jdbc:mysql://<fe_ip>:<fe_query_port>/useServerPrepStmts=true
    ```

2. StarRocksのGitHubプロジェクトは、JDBCドライバを介してStarRocksテーブルからデータを挿入、削除、更新、クエリする方法を説明する[Javaのコード例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)を提供しています。
