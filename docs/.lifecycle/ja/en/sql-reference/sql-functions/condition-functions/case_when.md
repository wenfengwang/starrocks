---
displayed_sidebar: English
---

# case

## 説明

CASE は条件式です。WHEN 句の条件が true と評価される場合、THEN 句の結果を返します。どの条件も true と評価されない場合、オプションの ELSE 句の結果を返します。ELSE が存在しない場合は、NULL が返されます。

## 構文

CASE 式には、次の 2 つの形式があります。

- シンプル CASE

```SQL
CASE expression
    WHEN expression1 THEN result1
    [WHEN expression2 THEN result2]
    ...
    [WHEN expressionN THEN resultN]
    [ELSE result]
END
```

この構文では、`expression` は WHEN 句の各式と比較されます。等しい式が見つかった場合、THEN 句の結果が返されます。等しい式が見つからない場合、ELSE が存在する場合は ELSE 句の結果が返されます。

- 検索 CASE

```SQL
CASE WHEN condition1 THEN result1
    [WHEN condition2 THEN result2]
    ...
    [WHEN conditionN THEN resultN]
    [ELSE result]
END
```

この構文では、WHEN 句の各条件が true と評価されるまで評価され、対応する THEN 句の結果が返されます。どの条件も true と評価されない場合、ELSE が存在する場合は ELSE 句の結果が返されます。

最初の CASE は、以下のように第二の CASE と等価です。

```SQL
CASE WHEN expression = expression1 THEN result1
    [WHEN expression = expression2 THEN result2]
    ...
    [WHEN expression = expressionN THEN resultN]
    [ELSE result]
END
```

## パラメータ

- `expressionN`: 比較する式です。複数の式はデータ型が互換性がある必要があります。

- `conditionN`: BOOLEAN 値に評価可能な条件です。

- `resultN`: データ型が互換性がある必要があります。

## 戻り値

戻り値は THEN 句のすべての型の共通の型です。

## 例

`test_case` テーブルに次のデータがあるとします。

```SQL
CREATE TABLE test_case(
    name          STRING,
    gender        INT
) DISTRIBUTED BY HASH(name);

INSERT INTO test_case VALUES
    ("Andy",1),
    ("Jules",0),
    ("Angel",-1),
    ("Sam",NULL);

SELECT * FROM test_case;
+-------+--------+
| name  | gender |
+-------+--------+
| Angel |     -1 |
| Andy  |      1 |
| Sam   |   NULL |
| Jules |      0 |
+-------+--------+
```

### シンプル CASE の使用

- ELSE が指定されており、等しい式が見つからない場合は ELSE の結果が返されます。

```plain
mysql> SELECT gender, CASE gender 
                    WHEN 1 THEN 'male'
                    WHEN 0 THEN 'female'
                    ELSE 'error'
               END gender_str
FROM test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|   NULL | error      |
|      0 | female     |
|      1 | male       |
|     -1 | error      |
+--------+------------+
```

- ELSE が指定されておらず、true と評価される条件がない場合は NULL が返されます。

```plain
SELECT gender, CASE gender 
                    WHEN 1 THEN 'male'
                    WHEN 0 THEN 'female'
               END gender_str
FROM test_case;
+--------+------------+
| gender | gender_str |
+--------+------------+
|      1 | male       |
|     -1 | NULL       |
|   NULL | NULL       |
|      0 | female     |
+--------+------------+
```

### ELSE が指定されていない検索 CASE の使用

```plain
mysql> SELECT gender, CASE WHEN gender = 1 THEN 'male'
                           WHEN gender = 0 THEN 'female'
                      END gender_str
FROM test_case;
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

case, case_when, case...when
