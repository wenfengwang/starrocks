---
displayed_sidebar: Chinese
---

# map_from_arrays

## 機能

二つの ARRAY 配列を Key と Value として組み合わせて、一つの MAP オブジェクトを作成します。

このコマンドはバージョン 3.0 からサポートされています。

## 文法

```Haskell
MAP map_from_arrays(ARRAY keys, ARRAY values)
```

## パラメータ説明

- `keys`: MAP の Key 値を生成するために使用します。`keys` の要素はユニークでなければなりません。
- `values`: MAP の Value 値を生成するために使用します。

## 戻り値の説明

`keys` の要素を Key とし、`values` の要素を Value とする MAP 値を返します。

戻り値のルールは以下の通りです：

- `keys` と `values` の長さ（要素数）は同じでなければならず、異なる場合はエラーを返します。

- `keys` または `values` が NULL の場合、NULL を返します。

## 例

```Plaintext
select map_from_arrays([1, 2], ['Star', 'Rocks']);
+--------------------------------------------+
| map_from_arrays([1, 2], ['Star', 'Rocks']) |
+--------------------------------------------+
| {1:"Star",2:"Rocks"}                       |
+--------------------------------------------+
```

```Plaintext
select map_from_arrays([1, 2], NULL);
+-------------------------------+
| map_from_arrays([1, 2], NULL) |
+-------------------------------+
| NULL                          |
+-------------------------------+
```
