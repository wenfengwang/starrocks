---
displayed_sidebar: Chinese
---

# ケース

## 機能

CASE は条件式であり、2つの形式があります：シンプル CASE 式とサーチ CASE 式。

- シンプル CASE 式では、ある式 `expression` を値と比較します。一致するものが見つかれば、THEN の結果を返します。一致するものがなければ、ELSE の結果を返します。ELSE が指定されていない場合は、NULL を返します。

- サーチ CASE 式では、ブール式 `condition` の結果が TRUE かどうかを判断します。TRUE であれば THEN の結果を返し、そうでなければ ELSE の結果を返します。ELSE が指定されていない場合は、NULL を返します。

## 文法

- シンプル CASE 式

```SQL
CASE expression
    WHEN expression1 THEN result1
    [WHEN expression2 THEN result2]
    ...
    [WHEN expressionN THEN resultN]
    [ELSE result]
END
```

- サーチ CASE 式

```SQL
CASE WHEN condition1 THEN result1
    [WHEN condition2 THEN result2]
    ...
    [WHEN conditionN THEN resultN]
    [ELSE result]
END
```

シンプル CASE 式は、以下のサーチ CASE 式で表現することもでき、機能的には等価です。

```SQL
CASE WHEN expression = expression1 THEN result1
    [WHEN expression = expression2 THEN result2]
    ...
    [WHEN expression = expressionN THEN resultN]
    [ELSE result]
END
```

## パラメータ説明

- `expressionN`：比較する式。複数の式はデータ型が互換性がある必要があります。

- `conditionN`：判断する条件。

- `resultN`：返される結果。複数の結果はデータ型が互換性がある必要があります。

## 戻り値

戻り値の型は、すべての THEN 節の結果の共通型 (common type) です。

## 例

`test_case` というテーブルがあり、以下のデータがあるとします：

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

### シンプル CASE 式

- ELSE を指定し、一致するものが見つからない場合は ELSE の結果を返します。

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

- ELSE を指定していない場合、一致するものが見つからないと NULL を返します。

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

### サーチ CASE 式（ELSE を指定）

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

case, case, case_when, case...when
