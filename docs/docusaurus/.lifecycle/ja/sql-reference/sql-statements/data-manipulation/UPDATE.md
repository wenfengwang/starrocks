```yaml
---
displayed_sidebar: "English"
---

# UPDATE（更新）

プライマリ キー テーブル内の行を更新します。

StarRocksはv2.3以降UPDATEステートメントをサポートしており、これは単一テーブルのUPDATEのみをサポートし、共通テーブル式（CTE）はサポートしていませんでした。v3.0以降、StarRocksはシンタックスを拡張し、複数テーブルの結合とCTEをサポートしています。データベース内の他のテーブルと更新するテーブルを結合する必要がある場合は、FROM句またはCTEで他のテーブルを参照できます。v3.1以降、UPDATEステートメントは列モードで部分更新をサポートしており、少数の列で大量の行が関与するシナリオに適しており、更新速度が速くなります。

このコマンドを実行するには、更新したいテーブルにUPDATE権限が必要です。

## 使用上の注意

複数のテーブルを含むUPDATEステートメントを実行すると、StarRocksはUPDATEステートメントのFROM句内のテーブル式を同等のJOINクエリステートメントに変換します。したがって、UPDATEステートメントのFROM句に指定するテーブル式がこの変換をサポートしていることを確認してください。たとえば、UPDATEステートメントが 'UPDATE t0 SET v1=t1.v1 FROM t1 WHERE t0.pk = t1.pk;' の場合、FROM句のテーブル式は 't0 JOIN t1 ON t0.pk=t1.pk;' に変換されます。 StarRocksはJOINクエリの結果セットに基づいて更新するデータ行を一致させます。結果セット内の複数の行が特定の行と一致する可能性があるため、その行はこれらの複数の行の中からランダムな行の値に基づいて更新されます。

## シンタックス

### 単一テーブルのUPDATE

UPDATEステートメントは、UPDATEされるテーブルのデータ行がWHERE条件を満たす場合、これらのデータ行の指定された列に新しい値を割り当てます。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    WHERE <where_condition>
```

### 複数テーブルのUPDATE

複数テーブルの結合の結果セットは、UPDATEされるテーブルに一致するようになります。テーブルのデータ行が結果セットに一致し、WHERE条件を満たす場合、これらのデータ行の指定された列に新しい値が割り当てられます。

```SQL
[ WITH <with_query> [, ...] ]
UPDATE <table_name>
    SET <column_name> = <expression> [, ...]
    [ FROM <from_item> [, ...] ]
    WHERE <where_condition>
```

## パラメータ

`with_query`

UPDATEステートメントで名前で参照できる1つ以上のCTE（共通テーブル式）。CTEは一時的な結果セットであり、複雑なステートメントの可読性を向上することができます。

`table_name`

更新するテーブルの名前。

`column_name`

更新する列の名前。テーブル名を含めることはできません。たとえば、'UPDATE t1 SET col = 1' は無効です。

`expression`

列に新しい値を割り当てる式。

`from_item`

データベース内の1つ以上の他のテーブル。これらのテーブルは、WHERE句で指定された条件に基づいて更新されるテーブル内の一致した行の指定された列の値を更新するためにテーブルと結合できます。たとえば、FROM句が `FROM t1 WHERE t0.pk = t1.pk` の場合、StarRocksはUPDATEステートメントを実行する際にFROM句内のテーブル式を `t0 JOIN t1 ON t0.pk=t1.pk` に変換します。

`where_condition`

行を更新したい基になる条件。WHERE条件を満たす行のみを更新できます。このパラメータは必須ですが、これにより誤ってテーブル全体を更新することを防ぐことができます。テーブル全体を更新したい場合は、'WHERE true' を使用できます。ただし、このパラメータは[列モードでの部分的な更新には必須ではありません](#partial-updates-in-column-mode-since-v31)。

## 列モードでの部分的な更新（v3.1以降）

列モードでの部分的な更新は、更新する列が少数であるが行が大量に関与する場合に適しています。このようなシナリオでは、列モードを有効にすると更新速度が速くなります。たとえば、100個の列を持つテーブルのうち、すべての行に対して更新されるのは10個の列（合計の10%）のみである場合、列モードの更新速度は10倍速くなります。

システム変数 `partial_update_mode` は部分的な更新のモードを制御し、以下の値をサポートします: 

- `auto` (デフォルト): システムはUPDATEステートメントと関連する列を分析して部分的な更新のモードを自動的に決定します。以下の基準を満たす場合、システムは自動的に列モードを使用します：
  - 更新される列の割合が総列数に対して30%未満であり、更新される列の数が4つ未満である。
  - UPDATE文にWHERE条件が含まれていない場合。
それ以外の場合、システムは列モードを使用しません。

- `column`: 部分的な更新に列モードを使用し、特に少数の列と大量の行が関与する部分的な更新に適しています。

`EXPLAIN UPDATE xxx` を使用して部分的な更新のモードを表示できます。

## 例

### 単一テーブルのUPDATE

従業員情報を記録するために `Employees` テーブルを作成し、このテーブルに5つのデータ行を挿入します。

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

全従業員に10%の昇給を与える必要がある場合、次のステートメントを実行できます:

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加。
WHERE true;
```

平均給与以下の従業員に10%の昇給を与える必要がある場合、次のステートメントを実行できます:

```SQL
UPDATE Employees
SET Salary = Salary * 1.1   -- 給与を10%増加。
WHERE Salary < (SELECT AVG(Salary) FROM Employees);
```

上記のステートメントを読みやすくするために、CTEを使用することもできます。

```SQL
WITH AvgSalary AS (
    SELECT AVG(Salary) AS AverageSalary
    FROM Employees
)
UPDATE Employees
SET Salary = Salary * 1.1   -- 給与を10%増加。
FROM AvgSalary
WHERE Employees.Salary < AvgSalary.AverageSalary;
```

### 複数テーブルのUPDATE

アカウント情報を記録するために `Accounts` テーブルを作成し、このテーブルに3つのデータ行を挿入します。

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

`Employees` テーブルでAcme Corporationのアカウントを管理する従業員に10%の昇給を与える必要がある場合、次のステートメントを実行できます:

```SQL
UPDATE Employees
SET Salary = Salary * 1.1  -- 給与を10%増加。
FROM Accounts
WHERE Accounts.name = 'Acme Corporation'
    AND Employees.Name = Accounts.Sales_person;
```

上記のステートメントを読みやすくするために、CTEを使用することもできます。

```SQL
WITH Acme_Accounts as (
    SELECT * from Accounts
    WHERE Accounts.name = 'Acme Corporation'
)
UPDATE Employees SET Salary = Salary * 1.1 -- 給与を10%増加。
FROM Acme_Accounts
WHERE Employees.Name = Acme_Accounts.Sales_person;
```
```