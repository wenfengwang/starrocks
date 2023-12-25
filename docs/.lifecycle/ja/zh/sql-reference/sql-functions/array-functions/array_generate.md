---
displayed_sidebar: Chinese
---

# array_generate

## 機能

`start` と `end` の間の数値要素を含む配列を生成し、ステップ幅は `step` です。

この関数はバージョン 3.1 からサポートされています。

## 文法

```Haskell
ARRAY array_generate([start,] end [, step])
```

## パラメータ説明

- `start`：オプション。TINYINT、SMALLINT、INT、BIGINT、LARGEINT のデータ型を持つ定数またはカラムをサポートします。指定されていない場合のデフォルト値は 1 です。
- `end`：必須。TINYINT、SMALLINT、INT、BIGINT、LARGEINT のデータ型を持つ定数またはカラムをサポートします。
- `step`：オプション。TINYINT、SMALLINT、INT、BIGINT、LARGEINT のデータ型を持つ定数またはカラムをサポートします。`start` < `end` の場合、指定されていない場合のデフォルト値は 1 です。`start` > `end` の場合、指定されていない場合のデフォルト値は -1 です。

## 戻り値の説明

戻り値のデータ型は ARRAY です。配列内の要素の型は入力パラメータの型と同じです。

## 注意事項

- 任意のパラメータがカラムの場合、そのカラムが属するテーブルを指定する必要があります。
- 任意のパラメータがカラムの場合、他のパラメータは指定されている必要があり、デフォルト値は使用できません。
- 任意のパラメータが NULL の場合、結果は NULL となります。
- `step` が 0 の場合、空の配列を返します。
- `start` と `end` が等しい場合、その値を返します。

## 例

### 入力パラメータが定数の場合

```Plain Text
mysql> select array_generate(9);
+---------------------+
| array_generate(9)   |
+---------------------+
| [1,2,3,4,5,6,7,8,9] |
+---------------------+

mysql> select array_generate(9,12);
+-----------------------+
| array_generate(9, 12) |
+-----------------------+
| [9,10,11,12]          |
+-----------------------+

mysql> select array_generate(9,6);
+----------------------+
| array_generate(9, 6) |
+----------------------+
| [9,8,7,6]            |
+----------------------+

mysql> select array_generate(9,6,-1);
+--------------------------+
| array_generate(9, 6, -1) |
+--------------------------+
| [9,8,7,6]                |
+--------------------------+

mysql> select array_generate(3,3);
+----------------------+
| array_generate(3, 3) |
+----------------------+
| [3]                  |
+----------------------+
```

### 入力パラメータがカラムの場合

```sql
CREATE TABLE `array_generate`
(
  `c1` TINYINT,
  `c2` SMALLINT,
  `c3` INT
)
ENGINE = OLAP
DUPLICATE KEY(`c1`)
DISTRIBUTED BY HASH(`c1`);

INSERT INTO `array_generate` VALUES
(1, 6, 3),
(2, 9, 4);
```

```Plain Text
mysql> select array_generate(1,c2,2) from `array_generate`;
+--------------------------+
| array_generate(1, c2, 2) |
+--------------------------+
| [1,3,5]                  |
| [1,3,5,7,9]              |
+--------------------------+
```
