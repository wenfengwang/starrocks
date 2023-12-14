---
displayed_sidebar: "Chinese"
---

# 预编译语句 

从v3.2版本开始，StarRocks为执行具有相同结构但不同变量的SQL语句多次提供了预编译语句。此功能显著提高了执行效率并防止了SQL注入。

## 描述 

预编译语句的工作原理如下：

1. **准备**: 用户准备一个SQL语句，其中变量由占位符`?`代表。前端解析SQL语句并生成执行计划。
2. **执行**: 声明变量后，用户将这些变量传递给语句并执行。用户可以多次执行相同语句，但使用不同变量。

**好处**

- **节省解析开销**: 在实际业务场景中，应用程序通常多次使用具有相同结构但不同变量的语句。通过支持预编译语句，StarRocks在准备阶段仅需解析一次该语句。随后对具有不同变量的相同语句的执行可以直接使用预先生成的解析结果。因此，语句执行性能得到显着提高，特别是对于复杂查询而言。
- **防止SQL注入攻击**: 通过将语句与变量分离并将用户输入数据作为参数传递，而非直接将变量拼接到语句中，StarRocks可以防止恶意用户执行恶意SQL代码。

**用法**

预编译语句仅在当前会话中生效，无法在其他会话中使用。当前会话退出后，该会话中创建的预编译语句会自动删除。

## 语法 

预编译语句的执行包括以下阶段：

- PREPARE：准备带有占位符`?`的语句。
- SET：在语句中声明变量。
- EXECUTE：传递声明的变量给语句并执行它。
- DROP PREPARE或DEALLOCATE PREPARE：删除预编译语句。

### PREPARE

**语法:**

```SQL
PREPARE <stmt_name> FROM <preparable_stmt>
```

**参数:**

- `stmt_name`：为预编译语句指定的名称，随后用于执行或取消预编译语句。名称在单个会话中必须是唯一的。
- `preparable_stmt`：要准备的SQL语句，其中变量的占位符为问号(`?`)。

**示例:**

准备一个带有特定值的`INSERT INTO VALUES()`语句，其中值由占位符`?`表示。

```SQL
PREPARE insert_stmt FROM 'INSERT INTO users (id, country, city, revenue) VALUES (?, ?, ?, ?)';
```

### SET

**语法:**

```SQL
SET @var_name = expr [, ...];
```

**参数:**

- `var_name`：用户定义变量的名称。
- `expr`：用户定义的变量。

**示例:** 声明四个变量。

```SQL
SET @id1 = 1, @country1 = 'USA', @city1 = 'New York', @revenue1 = 1000000;
```

有关更多信息，请参阅[用户定义变量](../../reference/user_defined_variables.md)。

### EXECUTE

**语法:**

```SQL
EXECUTE <stmt_name> [USING @var_name [, @var_name] ...]
```

**参数:**

- `var_name`：在`SET`语句中声明的变量的名称。
- `stmt_name`：预编译语句的名称。

**示例:**

将变量传递给`INSERT`语句并执行该语句。

```SQL
EXECUTE insert_stmt USING @id1, @country1, @city1, @revenue1;
```

### DROP PREPARE或DEALLOCATE PREPARE

**语法:**

```SQL
{DEALLOCATE | DROP} PREPARE <stmt_name>
```

**参数:**

- `stmt_name`：预编译语句的名称。

**示例:**

删除预编译语句。

```SQL
DROP PREPARE insert_stmt;
```

## 示例 

### 使用预编译语句 

以下示例演示了如何使用预编译语句向StarRocks表中插入、删除、更新和查询数据：

假设已经创建了名为`demo`的数据库和名为`users`的表：

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

1. 为执行准备语句。

    ```SQL
    PREPARE insert_stmt FROM 'INSERT INTO users (id, country, city, revenue) VALUES (?, ?, ?, ?)';
    PREPARE select_all_stmt FROM 'SELECT * FROM users';
    PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';
    PREPARE update_stmt FROM 'UPDATE users SET revenue = ? WHERE id = ?';
    PREPARE delete_stmt FROM 'DELETE FROM users WHERE id = ?';
    ```

2. 在这些语句中声明变量。

    ```SQL
    SET @id1 = 1, @id2 = 2;
    SET @country1 = 'USA', @country2 = 'Canada';
    SET @city1 = 'New York', @city2 = 'Toronto';
    SET @revenue1 = 1000000, @revenue2 = 1500000, @revenue3 = (SELECT (revenue) * 1.1 FROM users);
    ```

3. 使用声明的变量执行语句。

    ```SQL
    -- 插入两行数据。
    EXECUTE insert_stmt USING @id1, @country1, @city1, @revenue1;
    EXECUTE insert_stmt USING @id2, @country2, @city2, @revenue2;

    -- 从表中查询所有数据。
    EXECUTE select_all_stmt;

    -- 分别查询ID为1或2的数据。
    EXECUTE select_by_id_stmt USING @id1;
    EXECUTE select_by_id_stmt USING @id2;

    -- 部分更新ID为1的行。仅更新revenue列。
    EXECUTE update_stmt USING @revenue3, @id1;

    -- 删除ID为1的数据。
    EXECUTE delete_stmt USING @id1;

    -- 删除预编译语句。
    DROP PREPARE insert_stmt;
    DROP PREPARE select_all_stmt;
    DROP PREPARE select_by_id_stmt;
    DROP PREPARE update_stmt;
    DROP PREPARE delete_stmt;
    ```

### 在Java应用程序中使用预编译语句 

以下示例演示了如何使用Java应用程序使用JDBC驱动程序向StarRocks表中插入、删除、更新和查询数据：

1. 在JDBC中指定StarRocks的连接URL时，需要启用服务端预编译语句：

    ```Plaintext
    jdbc:mysql://<fe_ip>:<fe_query_port>/useServerPrepStmts=true
    ```

2. StarRocks GitHub项目提供了一个[Java代码示例](https://github.com/StarRocks/starrocks/blob/main/fe/fe-core/src/test/java/com/starrocks/analysis/PreparedStmtTest.java)，解释了如何通过JDBC驱动程序向StarRocks表中插入、删除、更新和查询数据。