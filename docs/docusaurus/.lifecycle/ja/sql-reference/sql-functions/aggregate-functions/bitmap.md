---
displayed_sidebar: "Japanese"
---

# ビットマップ

これは、ビットマップ内で複数の集計関数の使用方法を説明するシンプルな例です。詳細な機能の定義や他のビットマップ関数については、bitmap-functionsを参照してください。

## テーブルの作成

テーブルを作成する際には集計モデルが必要です。データ型はビットマップであり、集計関数はbitmap_unionです。

```SQL
CREATE TABLE `pv_bitmap` (
  `dt` int(11) NULL COMMENT "",
  `page` varchar(10) NULL COMMENT "",
  `user_id` bitmap BITMAP_UNION NULL COMMENT ""
) ENGINE=OLAP
AGGREGATE KEY(`dt`, `page`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`dt`);
```

注意：大量のデータがある場合は、高頻度のbitmap_unionに対応するロールアップテーブルを作成することをお勧めします。

```SQL
ALTER TABLE pv_bitmap ADD ROLLUP pv (page, user_id);
```

## データのロード

`TO_BITMAP (expr)`: 0 〜 18446744073709551615の符号なしbigintをビットマップに変換します。

`BITMAP_EMPTY ()`: 空のビットマップ列を生成し、挿入または入力時のデフォルト値に使用されます。

`BITMAP_HASH (expr)`: 任意の型の列をハッシングしてビットマップに変換します。

### ストリームのロード

ストリームロードを使用してデータを入力する場合、次のようにデータをビットマップフィールドに変換できます。

``` bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=to_bitmap(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

``` bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=bitmap_hash(user_id)" \
    http://host:8410/api/test/testDb/_stream_load
```

``` bash
cat data | curl --location-trusted -u user:passwd -T - \
    -H "columns: dt,page,user_id, user_id=bitmap_empty()" \
    http://host:8410/api/test/testDb/_stream_load
```

### Insert Into

Insert Intoを使用してデータを入力する場合、ソーステーブルの列の種類に基づいて対応するモードを選択する必要があります。

* ソーステーブルのid2の列の種類はビットマップです

```SQL
insert into bitmap_table1
select id, id2 from bitmap_table2;
```

* ターゲットテーブルのid2の列の種類はビットマップです

```SQL
insert into bitmap_table1 (id, id2)
values (1001, to_bitmap(1000))
, (1001, to_bitmap(2000));
```

* ソーステーブルのid2の列の種類はビットマップであり、bit_map_union()を使用した集計の結果です。

```SQL
insert into bitmap_table1
select id, bitmap_union(id2) from bitmap_table2 group by id;
```

* ソーステーブルのid2の列の種類はINTであり、ビットマップ型はto_bitmap()で生成されます。

```SQL
insert into bitmap_table1
select id, to_bitmap(id2) from table;
```

* ソーステーブルのid2の列の種類はSTRINGであり、ビットマップ型はbitmap_hash()で生成されます。

```SQL
insert into bitmap_table1
select id, bitmap_hash(id2) from table;
```

## データのクエリ

### 構文

``BITMAP_UNION (expr)`: 入力ビットマップの和集合を計算し、新しいビットマップを返します。

`BITMAP_UNION_COUNT (expr)`: 入力ビットマップの和集合を計算し、その要素数を返し、BITMAP_COUNT (BITMAP_UNION (expr))と同じです。性能がBITMAP_COUNT (BITMAP_UNION (expr))よりも優れているため、最初にBITMAP_UNION_COUNT関数を使用することが推奨されます。

`BITMAP_UNION_INT (expr)`: TINYINT、SMALLINT、INTの列の異なる値の数を計算し、COUNT (DISTINCT expr)と同じ値を返します。

`INTERSECT_COUNT (bitmap_column_to_count, filter_column, filter_values ...)`: filter_column条件を満たす複数のビットマップの積集合の要素数を計算します。bitmap_column_to_countはビットマップ型の列であり、filter_columnは可変次元の列で、filter_valuesは次元値のリストです。

`BITMAP_INTERSECT(expr)`: このグループのビットマップ値の積集合を計算し、新しいビットマップを返します。

### 例

以下のSQLは、前述の`pv_bitmap`テーブルを例にしています。

`user_id`の重複値を計算します：

```SQL
select bitmap_union_count(user_id)
from pv_bitmap;

select bitmap_count(bitmap_union(user_id))
from pv_bitmap;
```

`id`の重複値を計算します：

```SQL
select bitmap_union_int(id)
from pv_bitmap;
```

`user_id`の保持率を計算します：

```SQL
select intersect_count(user_id, page, 'game') as game_uv,
    intersect_count(user_id, page, 'shopping') as shopping_uv,
    intersect_count(user_id, page, 'game', 'shopping') as retention -- 'game'と'shopping'ページの両方にアクセスするユーザーの数
from pv_bitmap
where page in ('game', 'shopping');
```

## キーワード

ビットマップ、BITMAP_COUNT、BITMAP_EMPTY、BITMAP_UNION、BITMAP_UNION_INT、TO_BITMAP、BITMAP_UNION_COUNT、INTERSECT_COUNT、BITMAP_INTERSECT