---
displayed_sidebar: English
---

# 预处理语句

从 v3.2 开始，StarRocks 提供了预处理语句，用于多次以相同结构但不同变量执行 SQL 语句。该功能显著提高了执行效率，并防止了 SQL 注入。

## 描述

预处理语句的基本工作如下：

1. **准备**: 用户准备一个 SQL 语句，其中变量由占位符 `?` 表示。FE 解析 SQL 语句并生成执行计划。
2. **执行**: 在声明变量后，用户将这些变量传递给语句并执行。用户可以使用不同的变量多次执行同一语句。

**优势**

- **节省解析开销**: 在实际业务场景中，应用程序经常以相同结构但不同变量多次执行语句。有了预处理语句的支持，StarRocks 只需在准备阶段解析一次语句。随后使用不同变量执行相同语句时，可以直接使用预先生成的解析结果。因此，语句执行性能得到了显著提高，尤其是对于复杂查询。
- **防止 SQL 注入攻击**: 通过将语句与变量分离，并将用户输入的数据作为参数传递，而不是直接将变量串联到语句中，StarRocks 可以防止恶意用户执行恶意 SQL 代码。

**用法**

预处理语句仅在当前会话中有效，不能在其他会话中使用。当前会话退出后，将自动删除在该会话中创建的预处理语句。

## 语法

预处理语句的执行包括以下阶段：

- PREPARE: 准备语句，其中变量由占位符 `?` 表示。
- SET: 在语句中声明变量。
- EXECUTE: 将声明的变量传递给语句并执行它。
- DROP PREPARE 或 DEALLOCATE PREPARE: 删除预处理语句。

### PREPARE

**语法:**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**参数:**

- `stmt_name`: 为预处理语句指定的名称，随后用于执行或解除分配该预处理语句。该名称在单个会话中必须是唯一的。
- `preparable_stmt`: 要准备的 SQL 语句，其中变量的占位符是问号 (`?`)。目前仅支持**`SELECT` 语句**。

**例:**

准备一个带有特定值占位符 `?` 的 `SELECT` 语句。

```SQL
PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
```

### SET

**语法:**

```SQL
SET @var_name = expr [, ...];
```

**参数:**

- `var_name`: 用户定义的变量名称。
- `expr`: 用户定义的变量。

**例:** 声明变量。

```SQL
SET @id1 = 1, @id2 = 2;
```

有关详细信息，请参阅[用户定义变量](../../reference/user_defined_variables.md)。

### EXECUTE

**语法:**

```SQL
EXECUTE <stmt_name> [USING @var_name [, @var_name] ...]
```

**参数:**

- `var_name`: 在`SET`语句中声明的变量名称。
- `stmt_name`: 预处理语句的名称。

**例:**

将变量传递给`SELECT`语句并执行该语句。

```SQL
EXECUTE select_by_id_stmt USING @id1;
```

### DROP PREPARE 或 DEALLOCATE PREPARE

**语法:**

```SQL
{DEALLOCATE | DROP} PREPARE <stmt_name>
```

**参数:**

- `stmt_name`: 预处理语句的名称。

**例:**

删除预处理语句。

```SQL
DROP PREPARE select_by_id_stmt;
```

## 例子

### 使用预处理语句

以下示例演示了如何使用预处理语句从 StarRocks 表中插入、删除、更新和查询数据：

假设已经创建了名为 `demo` 的数据库和名为 `users` 的表：

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

1. 准备要执行的语句。

    ```SQL
    PREPARE select_all_stmt FROM 'SELECT * FROM users';
    PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
    ```

2. 在这些语句中声明变量。

    ```SQL
    SET @id1 = 1, @id2 = 2;
    ```

3. 使用声明的变量来执行语句。

    ```SQL
    -- 从表中查询所有数据。
    EXECUTE select_all_stmt;

    -- 分别查询 ID 为 1 或 2 的数据。
    EXECUTE select_by_id_stmt USING @id1;
    EXECUTE select_by_id_stmt USING @id2;
    ```

### 在 Java 应用程序中使用预处理语句

以下示例演示了如何使用 JDBC 驱动程序从 StarRocks 表中插入、删除、更新和查询数据的 Java 应用程序：

1. 在 JDBC 中指定 StarRocks 的连接 URL 时，需要启用服务端预处理语句：

    ```Plaintext
    jdbc:mysql://<fe_ip>:<fe_query_port>/useServerPrepStmts=true
    ```

2. StarRocks GitHub 项目提供了一个[Java 代码示例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)，解释了如何通过 JDBC 驱动程序从 StarRocks 表中插入、删除、更新和查询数据。
