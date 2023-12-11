---
displayed_sidebar: "Japanese"
---

# ビットマップ

以下は、ビットマップで複数の集計関数の使用例を示す簡単な例です。関数の詳細な定義や他のビットマップ関数については、bitmap-functions を参照してください。

## テーブルの作成

テーブルを作成する際には、集計モデルが必要です。データ型はビットマップであり、集計関数は `bitmap_union` です。

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

注：データ量が多い場合、頻繁に使用される `bitmap_union` に対応したロールアップテーブルを作成すると良いでしょう。

```SQL
ALTER TABLE pv_bitmap ADD ROLLUP pv (page, user_id);
```

## データのロード

`TO_BITMAP (expr)`: 0 〜 18446744073709551615 の符号なしbigintをビットマップに変換します。

`BITMAP_EMPTY ()`: 空のビットマップ列を生成し、挿入や入力時にデフォルト値として使用されます。

`BITMAP_HASH (expr)`: 任意の型の列をハッシュ化してビットマップに変換します。

### ストリームロード

Stream Load を使用してデータを入力する際には、以下のようにデータをビットマップフィールドに変換できます：

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

Insert Into を使用してデータを入力する際には、ソーステーブルの列のタイプに応じて対応するモードを選択する必要があります。

* ソーステーブルのid2列のタイプがビットマップの場合

```SQL
insert into bitmap_table1
select id, id2 from bitmap_table2;
```

* ターゲットテーブルのid2列のタイプがビットマップの場合

```SQL
insert into bitmap_table1 (id, id2)
values (1001, to_bitmap(1000))
, (1001, to_bitmap(2000));
```

* ソーステーブルのid2列のタイプがビットマップであり、 `bit_map_union()` を使用して集計された結果である場合

```SQL
insert into bitmap_table1
select id, bitmap_union(id2) from bitmap_table2 group by id;
```

* ソーステーブルのid2列のタイプがINTであり、ビットマップタイプが `to_bitmap()` によって生成された場合

```SQL
insert into bitmap_table1
select id, to_bitmap(id2) from table;
```

* ソーステーブルのid2列のタイプがSTRINGであり、ビットマップタイプが `bitmap_hash()` によって生成された場合

```SQL
insert into bitmap_table1
select id, bitmap_hash(id2) from table;
```

## データのクエリ

### 構文

``BITMAP_UNION (expr)`: 入力ビットマップの合わせを計算し、新しいビットマップを返します。

`BITMAP_UNION_COUNT (expr)`: 入力ビットマップの合わせを計算し、その枚数を返します。これは、BITMAP_COUNT (BITMAP_UNION (expr)) と同等であり、性能が BITMAP_COUNT (BITMAP_UNION (expr)) よりも優れているため、まず BITMAP_UNION_COUNT 関数を使用することをお勧めします。

`BITMAP_UNION_INT (expr)`: TINYINT、SMALLINT、INT形式の列の異なる値の数を計算し、COUNT (DISTINCT expr) と同じ値を返します。

`INTERSECT_COUNT (bitmap_column_to_count, filter_column, filter_values ...)`: filter_column 条件を満たす複数のビットマップの共通部分の枚数を計算します。bitmap_column_to_count はビットマップ型の列であり、filter_column は異なる次元の列であり、filter_values は次元値のリストです。

`BITMAP_INTERSECT(expr)`: このグループのビットマップ値の共通部分を計算し、新しいビットマップを返します。

### 例

以下のSQLは上記の `pv_bitmap` テーブルを使用しています：

`user_id` の重複値を計算します：

```SQL
select bitmap_union_count(user_id)
from pv_bitmap;

select bitmap_count(bitmap_union(user_id))
from pv_bitmap;
```

`id` の重複値を計算します：

```SQL
select bitmap_union_int(id)
from pv_bitmap;
```

`user_id` のリテンションを計算します：

```SQL
select intersect_count(user_id, page, 'game') as game_uv,
    intersect_count(user_id, page, 'shopping') as shopping_uv,
    intersect_count(user_id, page, 'game', 'shopping') as retention -- 'game' と 'shopping' ページの両方にアクセスしたユーザーの数
from pv_bitmap
where page in ('game', 'shopping');
```

## キーワード

ビットマップ, ビットマップカウント, 空のビットマップ, ビットマップの合わせ, ビットマップの合わせの枚数, TO_BITMAP, ビットマップの共通部分の枚数, ビットマップの共通部分