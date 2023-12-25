---
displayed_sidebar: English
---

# UPDATE

プライマリキーを持つテーブルの行を更新します。

StarRocksはv2.3からUPDATE文をサポートしており、単一テーブルのUPDATEのみをサポートし、共通テーブル式（CTE）はサポートしていません。バージョン3.0からは、StarRocksは構文を拡張して複数テーブルの結合とCTEをサポートするようになりました。更新するテーブルをデータベース内の他のテーブルと結合する必要がある場合、FROM句またはCTEでこれらの他のテーブルを参照できます。バージョン3.1からは、UPDATE文は列モードでの部分更新をサポートし、これは少数の列だが多数の行を持つシナリオに適しており、更新速度が速くなります。

このコマンドを使用するには、更新したいテーブルに対するUPDATE権限が必要です。

## 使用上の注意

複数のテーブルを含むUPDATE文を実行する際、StarRocksはUPDATE文のFROM句にあるテーブル式を同等のJOINクエリ文に変換します。したがって、UPDATE文のFROM句で指定するテーブル式がこの変換をサポートしていることを確認してください。例えば、UPDATE文は 'UPDATE t0 SET v1=t1.v1 FROM t1 WHERE t0.pk = t1.pk;' です。FROM句のテーブル式は 't0 JOIN t1 ON t0.pk=t1.pk;' に変換できます。StarRocksはJOINクエリの結果セットに基づいて更新対象のデータ行を照合します。結果セット内の複数の行が更新対象のテーブル内の特定の行に一致する可能性があります。このシナリオでは、その行はこれら複数の行の中からランダムに選ばれた行の値に基づいて更新されます。

## 構文

### 単一テーブルのUPDATE

更新対象のテーブルのデータ行がWHERE条件を満たす場合、これらのデータ行の指定された列に新しい値が割り当てられます。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    WHERE <where_condition>
```

### 複数テーブルのUPDATE

複数テーブルの結合から得られた結果セットが更新対象のテーブルと照合されます。更新対象のテーブルのデータ行が結果セットと一致し、WHERE条件を満たす場合、これらのデータ行の指定された列に新しい値が割り当てられます。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    [ FROM <from_item> [, ...] ]
    WHERE <where_condition>
```

## パラメータ

`with_query`

UPDATE文で名前を指定して参照できる1つ以上のCTE。CTEは、複雑な文の可読性を向上させるための一時的な結果セットです。

`table_name`

更新されるテーブルの名前。

`column_name`

更新される列の名前。テーブル名を含めることはできません。例えば、「UPDATE t1 SET col = 1」というのは無効です。

`expression`

列に新しい値を割り当てる式。

`from_item`

データベース内の1つ以上の他のテーブル。これらのテーブルは、WHERE句で指定された条件に基づいて更新対象のテーブルと結合できます。結果セットの行の値は、更新対象のテーブル内の一致する行の指定された列の値を更新するために使用されます。例えば、FROM句が `FROM t1 WHERE t0.pk = t1.pk` の場合、StarRocksはUPDATE文を実行する際にFROM句のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk` に変換します。

`where_condition`

行を更新するための条件。WHERE条件を満たす行のみが更新されます。このパラメータは、誤ってテーブル全体を更新するのを防ぐために必要です。テーブル全体を更新したい場合は、「WHERE true」と使用できます。ただし、このパラメータは[列モードでの部分更新](#partial-updates-in-column-mode-since-v31)では必要ありません。

## 列モードでの部分更新（v3.1以降）

列モードでの部分更新は、更新する必要がある列は少ないが行は多いシナリオに適しています。このようなシナリオでは、列モードを有効にすることで更新速度が速くなります。例えば、100列あるテーブルで、全行のうち10列（全体の10％）のみを更新する場合、列モードの更新速度は10倍速くなります。

システム変数`partial_update_mode`は部分更新のモードを制御し、以下の値をサポートしています：

- `auto`（デフォルト）：システムはUPDATE文と関連する列を分析することによって、部分更新のモードを自動的に決定します。以下の基準が満たされる場合、システムは自動的に列モードを使用します：
  - 更新される列の割合が全体の列数の30％未満で、更新される列の数が4未満である。
  - UPDATE文がWHERE条件を使用していない。
それ以外の場合、システムは列モードを使用しません。

- `column`：列モードは部分更新に使用され、少数の列と多数の行を含む部分更新に特に適しています。

`EXPLAIN UPDATE xxx`を使用して部分更新のモードを確認できます。

## 例

### 単一テーブルのUPDATE

従業員情報を記録する`Employees`テーブルを作成し、5つのデータ行を挿入します。

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

すべての従業員に10%の昇給を与える必要がある場合、以下のステートメントを実行します。

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる。
WHERE true;
```

平均給与よりも低い給与をもらっている従業員に10%の昇給を与える必要がある場合、以下のステートメントを実行します。

```SQL
UPDATE Employees
SET Salary = Salary * 1.1   -- 給与を10%増加させる。
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

CTEを使用して上記のステートメントを書き換え、可読性を向上させることもできます。

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

### 複数テーブルのUPDATE

アカウント情報を記録する`Accounts`テーブルを作成し、3つのデータ行を挿入します。

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

Acme Corporationのアカウントを管理する`Employees`テーブルの従業員に10%の昇給を与える必要がある場合、以下のステートメントを実行します。

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加させる。
FROM Accounts
WHERE Accounts.Name = 'Acme Corporation'
    AND Employees.Name = Accounts.Sales_person;
```

CTEを使用して上記のステートメントを書き換え、可読性を向上させることもできます。

```SQL
WITH Acme_Accounts AS (
    SELECT * FROM Accounts
    WHERE Accounts.Name = 'Acme Corporation'
)
UPDATE Employees SET Salary = Salary * 1.1 -- 給与を10%増加させる。
FROM Acme_Accounts
WHERE Employees.Name = Acme_Accounts.Sales_person;
```
