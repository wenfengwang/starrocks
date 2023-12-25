---
displayed_sidebar: Chinese
---

# UPDATE

このステートメントは、プライマリキーモデルのテーブル内のデータ行を更新するために使用されます。

バージョン2.3～2.5では、StarRocksはUPDATEステートメントを提供し、シングルテーブルUPDATEのみをサポートし、共通テーブル式（CTE）はサポートしていませんでした。バージョン3.0から、StarRocksはUPDATE構文を拡張し、マルチテーブルの結合とCTEを使用することをサポートしました。更新対象のテーブルをデータベース内の他のテーブルと関連付ける必要がある場合は、FROM句またはCTE内で他のテーブルを参照できます。バージョン3.1からは、列モードの部分更新をサポートし、少数の列だが多数の行に関わるシナリオに適用され、更新速度を向上させます。

## 使用説明

マルチテーブルUPDATEの場合、UPDATEステートメント内のFROM句のテーブル式が同等のJOINクエリステートメントに変換できることを確認する必要があります。StarRocksが実際にUPDATEステートメントを実行する際には、内部でこのような変換を行います。例えばUPDATEステートメントが `UPDATE t0 SET v1=t1.v1 FROM t1 WHERE t0.pk = t1.pk;` である場合、このFROM句のテーブル式は `t0 JOIN t1 ON t0.pk=t1.pk;` に変換できます。そしてStarRocksはJOINクエリの結果セットに基づいて、更新対象テーブルのデータ行をマッチングし、指定された列の値を更新します。結果セットに複数のデータ行が存在し、更新対象テーブルのある行とマッチする場合、その行の指定された列の更新後の値は、結果セットの複数のデータ行の中からランダムに選ばれた行の指定された列の値になります。

## 構文

**シングルテーブル UPDATE**

更新対象テーブルのデータ行がWHERE条件を満たす場合、そのデータ行の指定された列に新しい値を設定します。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    WHERE <where_condition>
```

**マルチテーブル UPDATE**

マルチテーブルの結合クエリの結果セットに基づいて更新対象のテーブルとマッチングし、更新対象テーブルのデータ行が結果セットとマッチし、かつWHERE条件を満たす場合、そのデータ行の指定された列に新しい値を設定します。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    [ FROM <from_item> [, ...] ]
    WHERE <where_condition>
```

## パラメータ説明

`with_query`

UPDATEステートメント内で名前を使って参照できる一つまたは複数のCTE。CTEは一時的な結果セットであり、複雑なステートメントの可読性を向上させることができます。

`table_name`

更新対象のテーブル名。

`column_name`

更新対象の列名。テーブル名を含める必要はありません。例えば `UPDATE t1 SET t1.col = 1` は不正です。

`expression`

列に値を設定するための式。

`from_item`

データベース内の一つまたは複数の他のテーブルを参照します。このテーブルは更新対象のテーブルと結合し、WHERE句で結合条件を指定し、最終的に結合クエリの結果セットに基づいて更新対象のテーブルのマッチング行の列に値を設定します。例えばFROM句が `FROM t1 WHERE t0.pk = t1.pk;` である場合、StarRocksが実際にUPDATEステートメントを実行する際には、このFROM句のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk;` に変換します。

`where_condition`

WHERE条件を満たす行のみが更新されます。このパラメータは必須で、誤ってテーブル全体を更新することを防ぎます。テーブル全体を更新する必要がある場合は、`WHERE true` を使用してください。[列モードの部分更新](#列モードの部分更新自-31)を使用する場合、このパラメータは必須ではありません。

## 列モードの部分更新（バージョン 3.1から）

列モードの部分更新は、少数の列と多数の行に関わるシナリオに適用されます。このシナリオでは、列モードを有効にすることで、更新速度が向上します。例えば、100列を含むテーブルで、毎回10列（10%の割合）を更新し、全行を更新する場合、列モードを有効にすると、更新性能が10倍に向上します。

システム変数 `partial_update_mode` は部分更新のモードを制御するために使用され、以下の値をサポートしています：

* `auto`（デフォルト値）、システムが更新ステートメントと関連する列を分析し、部分更新を実行する際に使用するモードを自動的に判断します。以下の基準を満たす場合、システムは自動的に列モードを使用します：
  * 更新される列の数が全列数の30%未満であり、かつ更新される列の数が4列未満である。
  * 更新ステートメントにWHERE条件が使用されていない。
  それ以外の場合、システムは列モードを使用しません。
* `column`、列モードを指定して部分更新を実行します。これは、少数の列と多数の行に関わる部分的な列更新のシナリオに適しています。

`EXPLAIN UPDATE xxx` を使用して、部分列更新のモードを確認できます。

## 例

### シングルテーブル UPDATE

従業員情報を記録する `Employees` テーブルを作成し、5行のデータを挿入します。

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

全従業員の給与を10%増加させるには、以下のステートメントを実行します：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる
WHERE true;
```

平均給与より低い給与の従業員に10%の給与増加を適用するには、以下のステートメントを実行します：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

CTEを使用して上記のステートメントを書き換え、可読性を向上させることもできます。

```SQL
WITH AvgSalary AS (
    SELECT AVG(Salary) AS AverageSalary
    FROM Employees
)
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる
FROM AvgSalary
WHERE Employees.Salary < AvgSalary.AverageSalary;
```

### マルチテーブル UPDATE

アカウント情報を記録する `Accounts` テーブルを作成し、3行のデータを挿入します。

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

`Employees` テーブルのAcme Corporationのアカウントを管理する従業員の給与を10%増加させるには、以下のステートメントを実行します：

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる
FROM Accounts
WHERE Accounts.name = 'Acme Corporation'
    AND Employees.Name = Accounts.Sales_person;
```

CTEを使用して上記のステートメントを書き換え、可読性を向上させることもできます。

```SQL
WITH Acme_Accounts AS (
    SELECT * FROM Accounts
    WHERE Accounts.name = 'Acme Corporation'
)
UPDATE Employees SET Salary = Salary * 1.1 -- 給与を10%増加させる
FROM Acme_Accounts
WHERE Employees.Name = Acme_Accounts.Sales_person;
```
