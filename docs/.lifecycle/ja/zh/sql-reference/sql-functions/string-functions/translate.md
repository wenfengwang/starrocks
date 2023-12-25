---
displayed_sidebar: Chinese
---

# 翻訳

## 機能

指定された文字列の中の文字を置換するために使用される関数です。この関数は、`source` 文字列内で `from_string` に含まれる文字を `to_string` の対応する位置の文字に置換します。

この関数はバージョン 3.2 からサポートされています。

## 文法

```SQL
TRANSLATE(source, from_string, to_string)
```

## パラメータ説明

- `source`: 文字置換を行う文字列で、サポートされるデータ型は VARCHAR です。`from_string` に含まれない文字は、結果の文字列でそのまま出力されます。

- `from_string`: サポートされるデータ型は VARCHAR です。文字が `from_string` 内に複数回出現する場合、最初に出現したものだけが有効です（6番目の例を参照）。`from_string` 内の文字は以下のルールに従います：
  - `to_string` に対応する位置の文字がある場合、その文字に置換されます。
  - `to_string` に対応する位置の文字がない場合（つまり `to_string` の文字長が `from_string` より短い場合）、その文字は結果の文字列から削除されます（例2、3、4を参照）。

- `to_string`: `from_string` の対応する位置の文字を置換するために使用されます。サポートされるデータ型は VARCHAR です。`to_string` の文字長が `from_string` より長い場合、その余分な文字は無視され、何の影響もありません（例5を参照）。

## 戻り値の説明

戻り値のデータ型は VARCHAR です。

`NULL` が返されるシナリオ：

- いずれかの入力パラメータが `NULL` の場合。

- 置換後の文字列の長さが VARCHAR 型の最大長（1048576）を超える場合。

## 例

```plaintext
-- 元の文字列の 'ab' を '12' に順番に置換します。
MySQL > select translate('abcabc', 'ab', '12') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 元の文字列の 'mf1' を 'to' に順番に置換します。'to' の長さが 'mf1' より短いため、結果の文字列から '1' が削除されます。
MySQL > select translate('s1m1a1rrfcks','mf1','to') as test;
+-----------+
| test      |
+-----------+
| starrocks |
+-----------+

-- 元の文字列の 'テストa無視' を 'CS1' に順番に置換します。'CS1' の長さが 'テストa無視' より短いため、結果の文字列から '無視' が削除されます。
MySQL > select translate('テストabc無視', 'テストa無視', 'CS1') as test;
+-------+
| test  |
+-------+
| C1bcS |
+-------+

-- 元の文字列の 'ab' を '1' に順番に置換します。'1' の長さが 'ab' より短いため、結果の文字列から 'b' が削除されます。
MySQL > select translate('abcabc', 'ab', '1') as test;
+------+
| test |
+------+
| 1c1c |
+------+

-- 元の文字列の 'ab' を '123' に順番に置換します。'123' の長さが 'ab' より長いため、余分な文字 '3' は無視されます。
MySQL > select translate('abcabc', 'ab', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- 元の文字列の 'aba' を '123' に順番に置換します。'a' が繰り返し出現するため、最初に出現した 'a' のみが '1' に置換されます。
MySQL > select translate('abcabc', 'aba', '123') as test;
+--------+
| test   |
+--------+
| 12c12c |
+--------+

-- この関数は repeat() および concat() と組み合わせて使用されます。置換後の文字列の長さが VARCHAR 型の最大長を超えるため、NULL が返されます。
MySQL > select translate(concat('b', repeat('a', 1024*1024-3)), 'a', '膨張') as test;
+--------+
| test   |
+--------+
| NULL   |
+--------+

-- この関数は length()、repeat()、および concat() と組み合わせて使用され、置換後の文字列の長さを計算します。
MySQL > select length(translate(concat('b', repeat('a', 1024*1024-3)), 'b', '膨張')) as test
+---------+
| test    |
+---------+
| 1048576 |
+---------+
```

## 関連関数

- [concat](./concat.md)
- [length](./length.md)
- [repeat](./repeat.md)
