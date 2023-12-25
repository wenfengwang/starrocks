---
displayed_sidebar: English
---

# bitmap

ここでは、Bitmapでのいくつかの集約関数の使用例を簡単に説明します。詳細な関数定義やその他のBitmap関数については、bitmap-functionsを参照してください。

## テーブル作成

テーブルを作成する際には集約モデルが必要です。データ型はbitmapで、集約関数はbitmap_unionです。

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

注: 大量のデータがある場合は、頻繁にbitmap_unionを使用するロールアップテーブルを作成することをお勧めします。

```SQL
ALTER TABLE pv_bitmap ADD ROLLUP pv (page, user_id);
```

## データロード

`TO_BITMAP(expr)`: 0〜18446744073709551615の符号なしbigintをbitmapに変換します。

`BITMAP_EMPTY()`: 挿入時や入力時にデフォルト値として埋めるために使用される空のbitmap列を生成します。

`BITMAP_HASH(expr)`: 任意の型の列をハッシュを使用してbitmapに変換します。

### ストリームロード

Stream Loadを使用してデータを入力する場合、以下のようにデータをBitmapフィールドに変換できます：

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

Insert Intoを使用してデータを入力する場合、ソーステーブルの列の型に基づいて対応するモードを選択する必要があります。

* ソーステーブルのid2の列の型はbitmapです

```SQL
insert into bitmap_table1
select id, id2 from bitmap_table2;
```

* ターゲットテーブルのid2の列の型はbitmapです

```SQL
insert into bitmap_table1 (id, id2)
values (1001, to_bitmap(1000))
, (1001, to_bitmap(2000));
```

* ソーステーブルのid2の列の型はbitmapで、bit_map_union()を使用した集約の結果です。

```SQL
insert into bitmap_table1
select id, bitmap_union(id2) from bitmap_table2 group by id;
```

* ソーステーブルのid2の列の型はINTで、bitmap型はto_bitmap()によって生成されます。

```SQL
insert into bitmap_table1
select id, to_bitmap(id2) from table;
```

* ソーステーブルのid2の列の型はSTRINGで、bitmap型はbitmap_hash()によって生成されます。

```SQL
insert into bitmap_table1
select id, bitmap_hash(id2) from table;
```

## データクエリ

### 構文

`BITMAP_UNION(expr)`: 入力されたBitmapの和集合を計算し、新しいBitmapを返します。

`BITMAP_UNION_COUNT(expr)`: 入力されたBitmapの和集合を計算し、その基数を返します。これは`BITMAP_COUNT(BITMAP_UNION(expr))`と同等です。`BITMAP_UNION_COUNT`関数は`BITMAP_COUNT(BITMAP_UNION(expr))`よりもパフォーマンスが優れているため、最初に使用することを推奨します。

`BITMAP_UNION_INT(expr)`: TINYINT、SMALLINT、INT型の列の異なる値の数を計算し、`COUNT(DISTINCT expr)`と同じ値を返します。

`INTERSECT_COUNT(bitmap_column_to_count, filter_column, filter_values...)`: filter_columnの条件を満たす複数のbitmapの交差部分の基数を計算します。bitmap_column_to_countはbitmap型の列、filter_columnは様々な次元の列、filter_valuesは次元値のリストです。

`BITMAP_INTERSECT(expr)`: このグループのBitmap値の交差部分を計算し、新しいBitmapを返します。

### 例

以下のSQLは、上記の`pv_bitmap`テーブルを例に使用しています：

`user_id`の重複排除された値を計算します：

```SQL
select bitmap_union_count(user_id)
from pv_bitmap;

select bitmap_count(bitmap_union(user_id))
from pv_bitmap;
```

`id`の重複排除された値を計算します：

```SQL
select bitmap_union_int(id)
from pv_bitmap;
```

`user_id`のリテンションを計算します：

```SQL
select intersect_count(user_id, page, 'game') as game_uv,
    intersect_count(user_id, page, 'shopping') as shopping_uv,
    intersect_count(user_id, page, 'game', 'shopping') as retention -- 'game'と'shopping'の両方のページにアクセスするユーザー数
from pv_bitmap
where page in ('game', 'shopping');
```

## キーワード

BITMAP,BITMAP_COUNT,BITMAP_EMPTY,BITMAP_UNION,BITMAP_UNION_INT,TO_BITMAP,BITMAP_UNION_COUNT,INTERSECT_COUNT,BITMAP_INTERSECT
