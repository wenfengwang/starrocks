---
displayed_sidebar: Chinese
---

# json_query

## **機能**

指定されたパス(`json_path`)の JSON オブジェクトの値をクエリし、JSON 型で出力します。

## **文法**

```Plain Text
JSON_QUERY(json_object_expr, json_path)
```

## **パラメータ説明**

- `json_object_expr`：JSON オブジェクトの式で、JSON 型の列や PARSE_JSON などの JSON 関数で構築された JSON オブジェクトが可能です。

- `json_path`：JSON オブジェクトをクエリする際のパス。サポートされるデータ型は文字列です。StarRocks がサポートする JSON Path の文法については、[JSON Path 文法](../overview-of-json-functions-and-operators.md#json-path)を参照してください。

## **戻り値の説明**

JSON 型の値を返します。

> クエリしたフィールドが存在しない場合は、SQL 型の NULL を返します。

## **例**

例1：JSON オブジェクト内のパス式 `'$.a.b'` で指定された値をクエリし、JSON 型の 1 を返します。

```Plain Text
mysql> SELECT JSON_QUERY(PARSE_JSON('{"a": {"b": 1}}'), '$.a.b') ;
       -> 1
```

例2：JSON オブジェクト内のパス式 `'$.a.c'` で指定された値をクエリすると、その値が存在しないため、SQL 型の NULL を返します。

```Plain Text
mysql> SELECT JSON_QUERY(PARSE_JSON('{"a": {"b": 1}}'), '$.a.c') ;
       -> NULL
```

例3：JSON オブジェクト内のパス式 `'$.a[2]'` （a 配列の第2要素）で指定された値をクエリし、JSON 型の 3 を返します。

```Plain Text
mysql> SELECT JSON_QUERY(PARSE_JSON('{"a": [1,2,3]}'), '$.a[2]') ;
       -> 3
```

例4：JSON オブジェクト内のパス式 `'$.a[3]'` （a 配列の第3要素）で指定された値をクエリすると、その値が存在しないため、SQL 型の NULL を返します。

```Plain Text
mysql> SELECT JSON_QUERY(PARSE_JSON('{"a": [1,2,3]}'), '$.a[3]') ;
       -> NULL
```
