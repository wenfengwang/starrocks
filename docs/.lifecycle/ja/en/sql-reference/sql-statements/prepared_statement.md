---
displayed_sidebar: English
---

# プリペアドステートメント

v3.2以降、StarRocksは、同じ構造で異なる変数を持つSQLステートメントを複数回実行するためのプリペアドステートメントを提供します。この機能により、実行効率が大幅に向上し、SQLインジェクションが防止されます。

## 説明

プリペアドステートメントは、基本的に次のように機能します。

1. **準備**: ユーザーは、変数がプレースホルダー `?` で表されるSQLステートメントを準備します。FEはSQL文を解析し、実行計画を生成します。
2. **実行**: 変数を宣言した後、ユーザーはこれらの変数をステートメントに渡し、ステートメントを実行します。ユーザーは、異なる変数を使用して同じステートメントを複数回実行できます。

**利点**

- **解析のオーバーヘッドの節約**: 実際のビジネスシナリオでは、アプリケーションは多くの場合、同じ構造で異なる変数を使用してステートメントを複数回実行します。プリペアドステートメントがサポートされている場合、StarRocksは準備フェーズ中に一度だけステートメントを解析する必要があります。異なる変数を使用して同じステートメントを後続で実行すると、事前に生成された解析結果を直接使用できます。そのため、特に複雑なクエリの場合、ステートメントの実行パフォーマンスが大幅に向上します。
- **SQLインジェクション攻撃の防止**: 変数を直接連結してステートメントにするのではなく、ステートメントを変数から分離し、ユーザー入力データをパラメーターとして渡すことで、StarRocksは悪意のあるユーザーが悪意のあるSQLコードを実行するのを防ぐことができます。

**用途**

プリペアドステートメントは、現在のセッションでのみ有効であり、他のセッションでは使用できません。現在のセッションが終了すると、そのセッションで作成されたプリペアドステートメントは自動的に削除されます。

## 構文

プリペアドステートメントの実行は、以下のフェーズで構成されます。

- PREPARE: 変数がプレースホルダー `?` で表されるステートメントを準備します。
- SET: ステートメント内で変数を宣言します。
- EXECUTE: 宣言された変数をステートメントに渡して実行します。
- DROP PREPARE または DEALLOCATE PREPARE: プリペアドステートメントを削除します。

### PREPARE

**構文:**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**パラメーター:**

- `stmt_name`: プリペアドステートメントに付けられた名前で、その後、そのプリペアドステートメントの実行または割り当て解除に使用されます。名前は1つのセッション内で一意である必要があります。
- `preparable_stmt`: 準備するSQLステートメントで、変数のプレースホルダーは疑問符 `?` です。現在、**SELECTステートメントのみ**がサポートされています。

**例:**

プレースホルダー `?` で表される特定の値を持つ `SELECT` ステートメントを準備します。

```SQL
PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
```

### SET

**構文:**

```SQL
SET @var_name = expr [, ...];
```

**パラメーター:**

- `var_name`: ユーザー定義変数の名前。
- `expr`: ユーザー定義変数。

**例:** 変数を宣言します。

```SQL
SET @id1 = 1, @id2 = 2;
```

詳細については、[ユーザー定義変数](../../reference/user_defined_variables.md)を参照してください。

### EXECUTE

**構文:**

```SQL
EXECUTE <stmt_name> [USING @var_name [, @var_name] ...]
```

**パラメーター:**

- `var_name`: `SET` ステートメントで宣言された変数の名前。
- `stmt_name`: プリペアドステートメントの名前。

**例:**

変数を `SELECT` ステートメントに渡し、そのステートメントを実行します。

```SQL
EXECUTE select_by_id_stmt USING @id1;
```

### DROP PREPARE または DEALLOCATE PREPARE

**構文:**

```SQL
{DEALLOCATE | DROP} PREPARE <stmt_name>
```

**パラメーター:**

- `stmt_name`: プリペアドステートメントの名前。

**例:**

プリペアドステートメントを削除します。

```SQL
DROP PREPARE select_by_id_stmt;
```

## 例

### プリペアドステートメントの使用

次の例は、プリペアドステートメントを使用して、StarRocksテーブルからデータを挿入、削除、更新、およびクエリする方法を示しています。

次のデータベース `demo` とテーブル `users` が既に作成されていると仮定します。

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

1. 実行用のステートメントを準備します。

    ```SQL
    PREPARE select_all_stmt FROM 'SELECT * FROM users';
    PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
    ```

2. これらのステートメントで変数を宣言します。

    ```SQL
    SET @id1 = 1, @id2 = 2;
    ```

3. 宣言された変数を使用してステートメントを実行します。

    ```SQL
    -- テーブルからすべてのデータをクエリします。
    EXECUTE select_all_stmt;

    -- ID 1 または 2 のデータを個別にクエリします。
    EXECUTE select_by_id_stmt USING @id1;
    EXECUTE select_by_id_stmt USING @id2;
    ```

### Javaアプリケーションでのプリペアドステートメントの使用

次の例は、JavaアプリケーションがJDBCドライバーを使用して、StarRocksテーブルからデータを挿入、削除、更新、およびクエリする方法を示しています。

1. JDBCでStarRocksの接続URLを指定する場合、サーバー側のプリペアドステートメントを有効にする必要があります。

    ```Plaintext
    jdbc:mysql://<fe_ip>:<fe_query_port>/?useServerPrepStmts=true
    ```

2. StarRocks GitHubプロジェクトは、JDBCドライバーを使用してStarRocksテーブルからデータを挿入、削除、更新、およびクエリする方法を説明する[Javaコードの例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)を提供しています。
