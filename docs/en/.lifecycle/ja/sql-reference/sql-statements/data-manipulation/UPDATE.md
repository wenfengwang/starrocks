---
displayed_sidebar: "Japanese"
---

# UPDATE

プライマリキーテーブルの行を更新します。

StarRocksは、v2.3以降、UPDATEステートメントをサポートしています。このステートメントは、単一テーブルのUPDATEのみをサポートし、共通テーブル式（CTE）はサポートしていません。バージョン3.0以降、StarRocksはシンタックスを拡張して、マルチテーブルの結合とCTEをサポートしています。更新するテーブルを他のテーブルと結合する必要がある場合は、FROM句またはCTEでこれらの他のテーブルを参照することができます。バージョン3.1以降、UPDATEステートメントは列モードでの部分更新をサポートしており、少数の列が関与するが行数が多いシナリオに適しており、更新速度が速くなります。

このコマンドを実行するには、更新するテーブルに対するUPDATE権限が必要です。

## 使用上の注意

複数のテーブルを含むUPDATEステートメントを実行する場合、StarRocksはUPDATEステートメントのFROM句のテーブル式を等価なJOINクエリステートメントに変換します。したがって、UPDATEステートメントのFROM句で指定するテーブル式がこの変換をサポートしていることを確認してください。たとえば、UPDATEステートメントは 'UPDATE t0 SET v1=t1.v1 FROM t1 WHERE t0.pk = t1.pk;' です。FROM句のテーブル式は 't0 JOIN t1 ON t0.pk=t1.pk;' に変換できます。StarRocksは、JOINクエリの結果セットに基づいて更新するデータ行を一致させます。結果セット内の複数の行が特定の行と一致する場合があります。この場合、その行はこれらの複数の行の中からランダムな行の値に基づいて更新されます。

## 構文

### 単一テーブルのUPDATE

更新するテーブルのデータ行がWHERE条件を満たす場合、これらのデータ行の指定された列に新しい値が割り当てられます。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    WHERE <where_condition>
```

### マルチテーブルのUPDATE

マルチテーブルの結合結果セットが更新するテーブルと一致します。更新するテーブルのデータ行が結果セットとWHERE条件を満たす場合、これらのデータ行の指定された列に新しい値が割り当てられます。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    [ FROM <from_item> [, ...] ]
    WHERE <where_condition>
```

## パラメータ

`with_query`

UPDATEステートメントで名前で参照できる1つ以上のCTE。CTEは一時的な結果セットであり、複雑なステートメントの可読性を向上させることができます。

`table_name`

更新するテーブルの名前。

`column_name`

更新する列の名前。テーブル名を含めることはできません。たとえば、'UPDATE t1 SET col = 1' は無効です。

`expression`

列に新しい値を割り当てる式。

`from_item`

データベース内の他の1つ以上のテーブル。これらのテーブルは、WHERE句で指定された条件に基づいて更新するテーブルと結合することができます。結果セットの行の値は、更新するテーブルの一致する行の指定された列の値を更新するために使用されます。たとえば、FROM句が `FROM t1 WHERE t0.pk = t1.pk` の場合、StarRocksはUPDATEステートメントを実行する際にFROM句のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk` に変換します。

`where_condition`

行を更新するための条件。WHERE条件を満たす行のみが更新されます。このパラメータは必須です。なぜなら、テーブル全体を誤って更新することを防ぐのに役立つからです。テーブル全体を更新する場合は、'WHERE true' を使用することができます。ただし、このパラメータは[列モードでの部分更新](#列モードでの部分更新-v31以降)には必須ではありません。

## 列モードでの部分更新（v3.1以降）

列モードでの部分更新は、少数の列だけが更新されるが、行数が多いシナリオに適しています。このようなシナリオでは、列モードを有効にすることで更新速度が向上します。たとえば、100列のテーブルで、すべての行に対して10列（合計の10%）だけが更新される場合、列モードの更新速度は10倍速くなります。

システム変数 `partial_update_mode` は、部分更新のモードを制御し、次の値をサポートしています。

- `auto`（デフォルト）：システムは、UPDATEステートメントと関連する列を分析して、部分更新のモードを自動的に決定します。次の基準を満たす場合、システムは自動的に列モードを使用します。
  - 更新される列の割合が総列数の30%未満であり、更新される列の数が4つ未満である。
  - UPDATEステートメントがWHERE条件を使用していない。
それ以外の場合、システムは列モードを使用しません。

- `column`：部分更新に列モードを使用します。これは、少数の列と多数の行が関与する部分更新に特に適しています。

`EXPLAIN UPDATE xxx` を使用して、部分更新のモードを表示することができます。

## 例

### 単一テーブルのUPDATE

従業員情報を記録する`Employees`テーブルを作成し、テーブルに5つのデータ行を挿入します。

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

すべての従業員に10%の昇給を与える必要がある場合、次のステートメントを実行できます。

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる。
WHERE true;
```

平均給与よりも低い給与をもつ従業員に10%の昇給を与える必要がある場合、次のステートメントを実行できます。

```SQL
UPDATE Employees
SET Salary = Salary * 1.1   -- 給与を10%増加させる。
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

上記のステートメントを可読性を向上させるためにCTEを使用して書き直すこともできます。

```SQL
WITH AvgSalary AS (
    SELECT AVG(Salary) AS AverageSalary
    FROM Employees
)
UPDATE Employees
SET Salary = Salary * 1.1   -- 給与を10%増加させる。
FROM AvgSalary
WHERE Employees.Salary < AvgSalary.AverageSalary;
```

### マルチテーブルのUPDATE

アカウント情報を記録する`Accounts`テーブルを作成し、テーブルに3つのデータ行を挿入します。

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

`Employees`テーブルでAcme Corporationのアカウントを管理する従業員に10%の昇給を与える必要がある場合、次のステートメントを実行できます。

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる。
FROM Accounts
WHERE Accounts.name = 'Acme Corporation'
    AND Employees.Name = Accounts.Sales_person;
```

上記のステートメントを可読性を向上させるためにCTEを使用して書き直すこともできます。

```SQL
WITH Acme_Accounts as (
    SELECT * from Accounts
    WHERE Accounts.name = 'Acme Corporation'
)
UPDATE Employees SET Salary = Salary * 1.1 -- 給与を10%増加させる。
FROM Acme_Accounts
WHERE Employees.Name = Acme_Accounts.Sales_person;
```
