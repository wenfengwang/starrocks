---
displayed_sidebar: Chinese
---

# 予め準備されたステートメント

バージョン3.2以降、StarRocksは予め準備されたステートメント（prepared statement）を提供しており、構造が同じで一部の変数だけが異なるSQLステートメントを複数回実行する際に、実行効率を大幅に向上させることができ、SQLインジェクションを防ぐこともできます。

## 機能

**ワークフロー**

1. 予め準備：SQLステートメントを準備します。ステートメント内の変数はプレースホルダー `?` で表されます。FEはステートメントを解析し、実行計画を生成します。
2. 実行：変数を宣言した後、ステートメントに変数を渡して実行します。また、異なる変数を使って同じステートメントを複数回実行することができます。

**利点**

- SQLステートメントの解析コストを節約します。ビジネスアプリケーションでは、通常、異なる変数を使って同じ構造のステートメントを複数回実行します。予め準備されたステートメントを使用すると、StarRocksは予め準備段階で一度だけステートメントを解析し、その後は解析結果を直接使用して同じステートメント（変数が異なる）を複数回実行するため、特に複雑なステートメントの実行性能を大幅に向上させることができます。
- SQLインジェクション攻撃を防ぐ。ステートメントと変数を分離し、ユーザーが入力したデータをパラメータとして渡し、直接ステートメントに結合することはありません。これにより、悪意のあるユーザーが入力を利用して悪意のあるSQLコードを実行することを防ぐことができます。

**使用説明**

予め準備されたステートメントは現在のセッションでのみ有効であり、他のセッションでは使用できません。現在のセッションが終了すると、作成された予め準備されたステートメントは自動的に削除されます。

## 文法

予め準備されたステートメントは以下の通りです：

- PREPARE：SQLステートメントを準備し、ステートメント内の変数はプレースホルダー `?` で表されます。
- SET：ステートメント内の変数を宣言します。
- EXECUTE：宣言された変数を渡してステートメントを実行します。
- DEALLOCATE PREPARE または DROP PREPARE：予め準備されたステートメントを削除します。

### PREPARE

**文法：**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**パラメータ説明：**

- `stmt_name`：予め準備されたステートメントに名前を付け、後で実行や削除する際に参照します。この名前はセッション内で一意でなければなりません。
- `preparable_stmt`：予め準備が必要なSQLステートメントです。注意：**現在は `SELECT` ステートメントのみをサポートしています**。変数のプレースホルダーは英語の半角クエスチョンマーク (`?`) です。

**例：**

`SELECT` ステートメントを準備し、変数は `?` でプレースホルダーを使用します。そして、そのステートメントに `select_by_id_stmt` という名前を付けます。

```SQL
PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
```

### SET

**文法：**

```Plain
SET @var_name = expr [, ...];
```

**パラメータ説明：**

- `var_name`：カスタム変数の名前。
- `expr`：カスタム変数の値。

**例：**

変数を宣言します。

```SQL
SET @id1 = 1, @id2 = 2;
```

詳細は[ユーザー定義変数](../../reference/user_defined_variables.md)を参照してください。

### EXECUTE

**文法：**

```SQL
EXECUTE <stmt_name> [USING @var_name [, @var_name] ...]
```

**パラメータ説明：**

- `var_name`：`SET` ステートメントで宣言された変数の名前。
- `stmt_name`：予め準備されたステートメントの名前。

**例：**

変数を渡して `SELECT` ステートメントを実行します。

```SQL
EXECUTE select_by_id_stmt USING @id1;
```

### DEALLOCATE PREPARE または DROP PREPARE

```SQL
{DEALLOCATE | DROP} PREPARE <stmt_name>
```

**パラメータ説明：**

`stmt_name`：予め準備されたステートメントの名前。

**例：**

予め準備されたステートメントを削除します。

```SQL
DROP PREPARE select_by_id_stmt;
```

## 例

### 予め準備されたステートメントの使用

この例では、予め準備されたステートメントを使用してStarRocksのテーブルデータを操作する方法について説明します。

以下のようなデータベース `demo` とテーブル `users` を作成したとします。

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
   PREPARE select_all_stmt FROM 'SELECT * FROM users';
   PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
   ```

2. ステートメント内の変数を宣言します。

   ```SQL
   SET @id1 = 1, @id2 = 2;
   ```

3. 宣言された変数を使用してステートメントを実行します。

   ```SQL
   -- テーブル内のすべてのデータをクエリします
   EXECUTE select_all_stmt;
   
   -- IDが1と2のデータをそれぞれクエリします
   EXECUTE select_by_id_stmt USING @id1;
   EXECUTE select_by_id_stmt USING @id2;
   ```

4. 予め準備されたステートメントを削除します。

   ```SQL
   DROP PREPARE select_all_stmt;
   DROP PREPARE select_by_id_stmt;
   ```

### JAVAアプリケーションが予め準備されたステートメントを使用する

この例では、JAVAアプリケーションがJDBCドライバを通じて予め準備されたステートメントを使用してStarRocksのテーブルデータを操作する方法について説明します。

1. JDBC URLでStarRocksの接続アドレスを設定する際に、サーバーサイドの予め準備されたステートメントを有効にする必要があります。

    ```Plaintext
    jdbc:mysql://<fe_ip>:<fe_query_port>/?useServerPrepStmts=true
    ```

2. StarRocksのGitHubプロジェクトでは、StarRocksのテーブルデータを操作する方法を説明する[JAVAコードの例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)を提供しています。
