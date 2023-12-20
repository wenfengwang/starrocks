---
displayed_sidebar: English
---

# 预编译语句

从 v3.2 版本开始，StarRocks 提供了预编译语句功能，用于多次执行结构相同但变量不同的 SQL 语句。这一特性显著提高了执行效率并防止了 SQL 注入攻击。

## 描述

预编译语句的基本工作流程如下：

1. **准备阶段**：用户准备一条 SQL 语句，其中的变量用占位符 `?` 表示。前端（FE）解析 SQL 语句并生成执行计划。
2. **执行阶段**：用户声明变量后，将这些变量传递给语句并执行。用户可以多次用不同的变量执行相同的语句。

**优势**

- **节省解析成本**：在现实业务场景中，应用程序通常会多次执行结构相同但变量不同的语句。得益于预编译语句的支持，StarRocks 只需在准备阶段解析一次语句。随后即可直接使用已生成的解析结果执行相同的语句，仅需替换变量。因此，特别是对于复杂查询，语句的执行性能得到了显著提升。
- **预防 SQL 注入攻击**：通过将语句与变量分离，以及将用户输入的数据作为参数传递，而非直接将变量拼接到语句中，StarRocks能够阻止恶意用户执行恶意的 SQL 代码。

**使用方法**

预编译语句仅在当前会话中有效，无法在其他会话中使用。一旦当前会话结束，该会话中创建的预编译语句会被自动丢弃。

## 语法

预编译语句的执行包含以下几个阶段：

- PREPARE：编写预编译语句，其中变量用占位符 ? 表示。
- SET：在语句中声明变量。
- EXECUTE：将声明的变量传递给语句并执行。
- DROP PREPARE 或 DEALLOCATE PREPARE：删除预编译语句。

### PREPARE

**语法:**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**参数：**

- stmt_name：给预编译语句指定的名称，后续用于执行或释放该预编译语句。在同一会话中，该名称必须唯一。
- `preparable_stmt`: 待预编译的 SQL 语句，其中变量的占位符为问号（?）。目前，**只支持** `SELECT` 语句。

**示例:**

为 SELECT 语句准备占位符 ? 代表的特定值。

```SQL
PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
```

### SET

**语法:**

```SQL
SET @var_name = expr [, ...];
```

**参数：**

- var_name：用户定义变量的名称。
- expr：用户定义的变量表达式。

**示例:** 声明变量。

```SQL
SET @id1 = 1, @id2 = 2;
```

更多信息，请参见[user-defined variables](../../reference/user_defined_variables.md)。

### EXECUTE

**语法:**

```SQL
EXECUTE <stmt_name> [USING @var_name [, @var_name] ...]
```

**参数：**

- var_name：在 SET 语句中声明的变量名称。
- stmt_name：预编译语句的名称。

**示例:**

将变量传递给 SELECT 语句并执行。

```SQL
EXECUTE select_by_id_stmt USING @id1;
```

### DROP PREPARE 或 DEALLOCATE PREPARE

**语法:**

```SQL
{DEALLOCATE | DROP} PREPARE <stmt_name>
```

**参数：**

- stmt_name：预编译语句的名称。

**示例**:

删除一个预编译语句。

```SQL
DROP PREPARE select_by_id_stmt;
```

## 示例

### 使用预编译语句

以下示例展示了如何使用预编译语句来对 StarRocks 表进行插入、删除、更新和查询操作：

假设已创建名为 demo 的数据库和名为 users 的表：

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

1. 准备执行的语句。

   ```SQL
   PREPARE select_all_stmt FROM 'SELECT * FROM users';
   PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
   ```

2. 在这些语句中声明变量。

   ```SQL
   SET @id1 = 1, @id2 = 2;
   ```

3. 使用声明的变量执行语句。

   ```SQL
   -- Query all data from the table.
   EXECUTE select_all_stmt;
   
   -- Query data with ID 1 or 2 separately.
   EXECUTE select_by_id_stmt USING @id1;
   EXECUTE select_by_id_stmt USING @id2;
   ```

### 在 Java 应用程序中使用预编译语句

以下示例展示了 Java 应用程序如何通过 JDBC 驱动对 StarRocks 表进行插入、删除、更新和查询操作：

1. 在 JDBC 中指定 StarRocks 的连接 URL 时，需要启用服务端的预编译语句功能：

   ```Plaintext
   jdbc:mysql://<fe_ip>:<fe_query_port>/useServerPrepStmts=true
   ```

2. StarRocks GitHub 项目提供了 [Java 代码示例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)，演示了如何通过 JDBC 驱动对 StarRocks 表进行插入、删除、更新和查询操作。
