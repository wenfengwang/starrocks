---
displayed_sidebar: Chinese
---

# ビットマップ

ここでは、Bitmap のいくつかの集約関数の使用例を簡単な例で紹介します。具体的な関数の定義やその他の Bitmap 関数については [bitmap-functions](../bitmap-functions/bitmap_and.md) を参照してください。

## テーブル作成

テーブルを作成する際には、集約モデルを使用し、データ型は bitmap、集約関数は bitmap_union を使用します。

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

>データ量が多い場合は、頻繁に bitmap_union クエリを行うための rollup テーブルを作成することをお勧めします。例えば:

```SQL
ALTER TABLE pv_bitmap ADD ROLLUP pv (page, user_id);
```

## データのロード

`TO_BITMAP(expr)`: 0 から 18446744073709551615 までの unsigned bigint を bitmap に変換します。

`BITMAP_EMPTY()`: 空の bitmap 列を生成し、insert やインポート時にデフォルト値として使用します。

`BITMAP_HASH(expr)`: 任意の型の列を Hash を使って bitmap に変換します。

### ストリームロード

Stream Load を使ってデータをインポートする場合、以下のように Bitmap フィールドに変換できます:

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

Insert Into を使ってデータをインポートする場合、source テーブルの列の型に応じて適切な方法を選択する必要があります。

* source テーブルの id2 の列の型が bitmap の場合

```SQL
insert into bitmap_table1
select id, id2 from bitmap_table2;
```

* target テーブルの id2 の列の型が bitmap の場合

```SQL
insert into bitmap_table1 (id, id2)
values (1001, to_bitmap(1000))
, (1001, to_bitmap(2000));
```

* source テーブルの id2 の列の型が bitmap で、bitmap_union() を使って集約する場合

```SQL
insert into bitmap_table1
select id, bitmap_union(id2) from bitmap_table2 group by id;
```

* source テーブルの id2 の列の型が int で、to_bitmap() を使って bitmap 型に変換する場合

```SQL
insert into bitmap_table1
select id, to_bitmap(id2) from table;
```

* source テーブルの id2 の列の型が String で、bitmap_hash() を使って bitmap 型に変換する場合

```SQL
insert into bitmap_table1
select id, bitmap_hash(id2) from table;
```

## 構文と対応する機能

`BITMAP_UNION(expr)`: 入力された Bitmap の和集合を計算し、新しい Bitmap を返します。

`BITMAP_UNION_COUNT(expr)`: 入力された Bitmap の和集合を計算し、その基数を返します。`BITMAP_COUNT(BITMAP_UNION(expr))` と同等であり、現在は `BITMAP_UNION_COUNT()` の使用が推奨されており、そのパフォーマンスは `BITMAP_COUNT(BITMAP_UNION(expr))` よりも優れています。

`BITMAP_UNION_INT(expr)`: TINYINT、SMALLINT、INT 型の列で異なる値の数を計算し、COUNT(DISTINCT expr) と同じ値を返します。

`INTERSECT_COUNT(bitmap_column_to_count, filter_column, filter_values ...)`: filter_column のフィルター条件を満たす複数の bitmap の交差点の基数を計算します。bitmap_column_to_count は bitmap 型の列、filter_column は変動する次元列、filter_values は次元の値のリストです。

`BITMAP_INTERSECT(expr)`: このグループの bitmap 値の交差点を計算し、新しい bitmap を返します。

## 例

以下の SQL は、上記の pv_bitmap テーブルを例にしています。

user_id の重複を除いた値を計算します:

```SQL
select bitmap_union_count(user_id)
from pv_bitmap;

select bitmap_count(bitmap_union(user_id))
from pv_bitmap;
```

ID の重複を除いた値を計算します:

```SQL
select bitmap_union_int(id)
from pv_bitmap;
```

user_id のリテンションを計算します:

```SQL
select intersect_count(user_id, page, 'meituan') as meituan_uv,
    intersect_count(user_id, page, 'waimai') as waimai_uv,
    intersect_count(user_id, page, 'meituan', 'waimai') as retention -- 'meituan' と 'waimai' の両方のページに現れたユーザー数
from pv_bitmap
where page in ('meituan', 'waimai');
```
