---
displayed_sidebar: "Japanese"
---

# CASE（ケース）

## 説明

CASEは条件式です。WHEN句の条件がtrueに評価される場合、THEN句で結果を返します。条件がどれもtrueに評価されない場合、オプションのELSE句で結果を返します。ELSEが存在しない場合、NULLが返されます。

## 構文

CASE式には2つの形式があります。

- シンプルCASE

```SQL
CASE expression
    WHEN expression1 THEN result1
    [WHEN expression2 THEN result2]
    ...
    [WHEN expressionN THEN resultN]
    [ELSE result]
END
```

この構文では、`expression`はWHEN句の各式と比較されます。等しい式が見つかった場合、THEN句の結果が返されます。等しい式が見つからない場合、ELSEが存在する場合はELSE句の結果が返されます。

- サーチドCASE

```SQL
CASE WHEN condition1 THEN result1
    [WHEN condition2 THEN result2]
    ...
    [WHEN conditionN THEN resultN]
    [ELSE result]
END
```

この構文では、WHEN句の各条件が評価され、trueになった場合、対応するTHEN句の結果が返されます。条件がtrueに評価されない場合、ELSEが存在する場合はELSE句の結果が返されます。

最初のCASE式は次のように2番目のものと等しいです：

```SQL
CASE WHEN expression = expression1 THEN result1
    [WHEN expression = expression2 THEN result2]
    ...
    [WHEN expression = expressionN THEN resultN]
    [ELSE result]
END
```

## パラメータ

- `expressionN`: 比較する式。複数の式はデータ型で互換性が必要です。

- `conditionN`: BOOLEAN値に評価できる条件です。

- `resultN`はデータ型で互換性が必要です。

## 戻り値

戻り値はTHEN句のすべての型の共通型です。

## 例

テーブル`test_case`に次のデータがあるとします：

```SQL
CREATE TABLE test_case(
    name          STRING,
    gender         INT,
    ) DISTRIBUTED BY HASH(name);

INSERT INTO test_case VALUES
    ("Andy",1),
    ("Jules",0),
    ("Angel",-1),
    ("Sam",null);

SELECT * FROM test_case;
+-------+--------+
| name  | gender |
+-------+--------+
| Angel |     -1 |
| Andy  |      1 |
| Sam   |   NULL |
| Jules |      0 |
+-------+--------+-------+
```

### シンプルCASEの使用

- ELSEが指定され、等しい式が見つからない場合はELSEで指定された結果が返されます。

```plain
mysql> select gender, case gender 
                    when 1 then 'male'
                    when 0 then 'female'
                    else 'error'
               end gender_str
from test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|   NULL | error      |
|      0 | female     |
|      1 | male       |
|     -1 | error      |
+--------+------------+
```

- ELSEが指定されず、条件がtrueに評価されない場合はNULLが返されます。

```plain
select gender, case gender 
                    when 1 then 'male'
                    when 0 then 'female'
               end gender_str
from test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|      1 | male       |
|     -1 | NULL       |
|   NULL | NULL       |
|      0 | female     |
+--------+------------+
```

### ELSEが指定されていないサーチドCASEの使用

```plain
mysql> select gender, case when gender = 1 then 'male'
                           when gender = 0 then 'female'
                      end gender_str
from test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|   NULL | NULL       |
|     -1 | NULL       |
|      1 | male       |
|      0 | female     |
+--------+------------+
```

## キーワード

case when、case、case_when、case...when