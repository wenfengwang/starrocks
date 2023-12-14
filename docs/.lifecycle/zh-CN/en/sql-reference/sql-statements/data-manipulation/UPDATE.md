---
displayed_sidebar: "English"
---

# 更新

更新主键表中的行。

StarRocks自v2.3开始支持UPDATE语句，该语句仅支持单表更新，不支持常规表达式（CTEs）。从3.0版本开始，StarRocks丰富了语法以支持多表连接和CTEs。如果您需要将要更新的表与数据库中的其他表进行连接，可以在FROM子句或CTE中引用这些其他表。自3.1版本以来，UPDATE语句支持列模式下的部分更新，适用于涉及少量列但大量行的场景，从而提高更新速度。

此命令需要对要更新的表具有UPDATE权限。

## 使用说明

执行涉及多个表的UPDATE语句时，StarRocks将UPDATE语句的FROM子句中的表达式转换为等效的JOIN查询语句。因此，请确保您在UPDATE语句的FROM子句中指定的表达式支持此转换。例如，UPDATE语句为'UPDATE t0 SET v1=t1.v1 FROM t1 WHERE t0.pk = t1.pk;'。FROM子句中的表达式可以转换为't0 JOIN t1 ON t0.pk=t1.pk;'。StarRocks根据JOIN查询的结果集匹配要更新的数据行。可能会有多行在结果集中与要更新的表中的某一行匹配。在这种情况下，该行将根据这些多行中的任意一行的值进行更新。

## 语法

### 单表更新

如果要更新的表的数据行满足WHERE条件，则将这些数据行的指定列分配新值。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    WHERE <where_condition>
```

### 多表更新

多表连接的结果集与要更新的表进行匹配。如果要更新的表的数据行与结果集匹配并满足WHERE条件，则将这些数据行的指定列分配新值。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    [ FROM <from_item> [, ...] ]
    WHERE <where_condition>
```

## 参数

`with_query`

可以在UPDATE语句中按名称引用的一个或多个CTE。CTE是可以改善复杂语句的可读性的临时结果集。

`table_name`

要更新的表的名称。

`column_name`

要更新的列的名称。不能包括表名。例如，'UPDATE t1 SET col = 1'是无效的。

`expression`

将新值分配给列的表达式。

`from_item`

数据库中的一个或多个其他表。可以根据WHERE子句中指定的条件将这些表与要更新的表进行连接。结果集中行的值用于更新要更新表中匹配行的指定列的值。例如，如果FROM子句为`FROM t1 WHERE t0.pk = t1.pk`，当执行UPDATE语句时，StarRocks将FROM子句中的表达式转换为`t0 JOIN t1 ON t0.pk=t1.pk`。

`where_condition`

基于该条件，您希望更新行。只有满足WHERE条件的行才能被更新。此参数是必需的，因为它有助于防止您意外更新整个表。如果要更新整个表，可以使用'WHERE true'。但是，对于[列模式下的部分更新](#列模式下的部分更新-自v31起)，此参数不是必需的。

## 列模式下的部分更新（自v3.1起）

列模式下的部分更新适用于仅需要更新少量列但需要更新大量行的情况。在这种情况下，启用列模式可以提供更快的更新速度。例如，在一个具有100列的表中，如果对所有行只更新了10列（总数的10%），则列模式的更新速度将快10倍。

系统变量`partial_update_mode`控制部分更新模式，并支持以下值：

- `auto`（默认）：系统根据分析UPDATE语句和涉及的列，自动确定部分更新的模式。如果满足以下条件，系统将自动使用列模式：
  - 更新的列占总列数的百分比小于30%，且更新的列数不到4个。
  - 更新语句不使用WHERE条件。
否则，系统不使用列模式。

- `column`：对部分更新使用列模式，特别适用于涉及少量列和大量行的部分更新。

您可以使用`EXPLAIN UPDATE xxx`查看部分更新模式。

## 示例

### 单表更新

创建一个表`Employees`以记录员工信息，并向表中插入5行数据。

```SQL
CREATE TABLE Employees (
    EmployeeID INT,
    Name VARCHAR(50),
    Salary DECIMAL(10, 2)
)
PRIMARY KEY (EmployeeID) 
DISTRIBUTED BY HASH (EmployeeID)
PROPERTIES ("replication_num" = "3");

INSERT INTO Employees VALUES
    (1, 'John Doe', 5000),
    (2, 'Jane Smith', 6000),
    (3, 'Robert Johnson', 5500),
    (4, 'Emily Williams', 4500),
    (5, 'Michael Brown', 7000);
```

如果您需要给所有员工涨工资10%，可以执行以下语句：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 将薪水提高10%。
WHERE true;
```

如果您需要给薪水低于平均薪水的员工涨工资10%，可以执行以下语句：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1   -- 将薪水提高10%。
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

您还可以使用CTE来重写上述语句以提高可读性。

```SQL
WITH AvgSalary AS (
    SELECT AVG(Salary) AS AverageSalary
    FROM Employees
)
UPDATE Employees
SET Salary = Salary * 1.1   -- 将薪水提高10%。
FROM AvgSalary
WHERE Employees.Salary < AvgSalary.AverageSalary;
```

### 多表更新

创建一个表`Accounts`以记录账户信息，并向表中插入3行数据。

```SQL
CREATE TABLE Accounts (
    Accounts_id BIGINT NOT NULL,
    Name VARCHAR(26) NOT NULL,
    Sales_person VARCHAR(50) NOT NULL
) 
PRIMARY KEY (Accounts_id)
DISTRIBUTED BY HASH (Accounts_id)
PROPERTIES ("replication_num" = "3");

INSERT INTO Accounts VALUES
    (1,'Acme Corporation','John Doe'),
    (2,'Acme Corporation','Robert Johnson'),
    (3,'Acme Corporation','Lily Swift');
```

如果您需要给管理Acme Corporation账户的员工涨薪水10%，可以执行以下语句：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 将薪水提高10%。
FROM Accounts
WHERE Accounts.name = 'Acme Corporation'
    AND Employees.Name = Accounts.Sales_person;
```

您还可以使用CTE来重写上述语句以提高可读性。

```SQL
WITH Acme_Accounts as (
    SELECT * from Accounts
    WHERE Accounts.name = 'Acme Corporation'
)
UPDATE Employees SET Salary = Salary * 1.1 -- 将薪水提高10%。
FROM Acme_Accounts
WHERE Employees.Name = Acme_Accounts.Sales_person;
```