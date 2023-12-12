---
displayed_sidebar: "Japanese"
---

# 正確なCount Distinctのためのビットマップの使用
このトピックでは、StarRocksでのビットマップの使用方法を説明します。

ビットマップは、配列内の異なる値の数を計算するための便利なツールです。このメソッドはストレージ容量を節約し、従来のCount Distinctと比較して計算を加速することができます。0からn未満の値範囲を持つ配列Aがあるとします。これに対して(n+7)/8バイトのビットマップを使用することで、配列内の異なる要素の数を計算できます。これを行うには、すべてのビットを0で初期化し、要素の値をビットの添字として設定し、その後すべてのビットを1に設定します。ビットマップ内の1の数が配列内の異なる要素の数です。

## 通常のCount Distinct

StarRocksはMPPアーキテクチャを使用しており、Count Distinctを使用するときに詳細なデータを保持できます。ただし、Count Distinct機能を使用するとクエリ処理中に複数のデータシャッフルが必要となり、データ量が増加するにつれて性能が線形に低下します。

以下のシナリオでは、テーブル（dt、page、user_id）の詳細データに基づいてUV（Unique Visitors）を計算します。

|  dt   |   page  | user_id |
| :---: | :---: | :---:|
|   20191206  |   ゲーム  | 101 |
|   20191206  |   ショッピング  | 102 |
|   20191206  |   ゲーム  | 101 |
|   20191206  |   ショッピング  | 101 |
|   20191206  |   ゲーム  | 101 |
|   20191206  |   ショッピング  | 101 |

StarRocksはデータを以下の図に従って計算します。まず、データを`page`と`user_id`の列でグループ化し、その後処理結果をカウントします。

![alter](../assets/6.1.2-2.png)

* 注意：この図は2つのBEノードで計算された6行のデータの模式図です。

大量のデータを処理する場合、複数のシャッフル操作が必要となり、必要な計算リソースが大幅に増加します。これによりクエリが遅くなります。しかし、ビットマップ技術を使用することで、これらの問題を解決し、クエリのパフォーマンスを向上させることができます。

`page`でグループ化したUVを数える：

```sql
select page, count(distinct user_id) as uv from table group by page;

|  page   |   uv  |
| :---: | :---: |
|   ゲーム  |  1   |
|   ショッピング  |   2  |
```

## ビットマップを使用したCount Distinctの利点

COUNT(DISTINCT expr)と比較して、ビットマップを使用することで以下のような利点を得ることができます：

* ストレージ領域の節約：INT32データの異なる値の数を計算する場合、必要なストレージ領域はCOUNT(DISTINCT expr)の1/32だけです。StarRocksでは、圧縮されたroaringビットマップを使用して計算を実行し、従来のビットマップと比較してストレージ領域の使用量をさらに削減しています。
* 高速な計算：ビットマップはビット演算を使用するため、COUNT(DISTINCT expr)と比較してより高速な計算が可能です。StarRocksでは、異なる値の数の計算を並列で処理できるため、クエリパフォーマンスがさらに向上します。

Roaring Bitmapの実装については、[特定の論文と実装](https://github.com/RoaringBitmap/RoaringBitmap)を参照してください。

## 使用上の注意

* ビットマップインデックスとビットマップCount Distinctの両方がビットマップ技術を使用しています。ただし、導入の目的と解決する問題は完全に異なります。前者は低カーディナリティを持つ列をフィルタリングするために使用され、後者はデータ行の値列内の異なる要素の数を計算するために使用されます。
* StarRocksの2.3以降のバージョンでは、テーブルの種類（結合テーブル、重複キー付きテーブル、プライマリキーテーブル、またはユニークキーテーブル）に関係なく、値列をBITMAPタイプに定義できます。ただし、テーブルの[ソートキー](../table_design/Sort_key.md)はBITMAPタイプにすることはできません。
* テーブルを作成する際に、値列をBITMAPとして定義し、集約関数を[BITMAP_UNION](../sql-reference/sql-functions/bitmap-functions/bitmap_union.md)にすることができます。
* TINYINT、SMALLINT、INT、BIGINTのデータの異なる値の数を計算する場合、roaringビットマップのみを使用できます。他のタイプのデータの場合は、[グローバル辞書を構築](#global-dictionary)する必要があります。

## ビットマップを使用したCount Distinct

ページUVの計算を例に取ります。

1. BITMAP_UNIONを利用したBITMAP列`visit_users`を持つ集約テーブルを作成します。

    ```sql
    CREATE TABLE `page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT 'アクセス時刻',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT 'ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`)
    PROPERTIES (
      "replication_num" = "3",
      "storage_format" = "DEFAULT"
    );
    ```

2. このテーブルにデータをロードします。

    INSET INTOを使用してデータをロードします：

    ```sql
    INSERT INTO page_uv VALUES
    (1, '2020-06-23 01:30:30', to_bitmap(13)),
    (1, '2020-06-23 01:30:30', to_bitmap(23)),
    (1, '2020-06-23 01:30:30', to_bitmap(33)),
    (1, '2020-06-23 02:30:30', to_bitmap(13)),
    (2, '2020-06-23 01:30:30', to_bitmap(23));
    ```

    データがロードされると：

    * 行 `page_id = 1, visit_date = '2020-06-23 01:30:30'` では、`visit_users`のフィールドに3つのビットマップ要素（13、23、33）が含まれています。
    * 行 `page_id = 1, visit_date = '2020-06-23 02:30:30'` では、`visit_users`のフィールドに1つのビットマップ要素（13）が含まれています。
    * 行 `page_id = 2, visit_date = '2020-06-23 01:30:30'` では、`visit_users`のフィールドに1つのビットマップ要素（23）が含まれています。

   ローカルファイルからデータをロードします：

    ```shell
    echo -e '1,2020-06-23 01:30:30,130\n1,2020-06-23 01:30:30,230\n1,2020-06-23 01:30:30,120\n1,2020-06-23 02:30:30,133\n2,2020-06-23 01:30:30,234' > tmp.csv | 
    curl --location-trusted -u <username>:<password> -H "label:label_1600960288798" \
        -H "column_separator:," \
        -H "columns:page_id,visit_date,visit_users, visit_users=to_bitmap(visit_users)" -T tmp.csv \
        http://StarRocks_be0:8040/api/db0/page_uv/_stream_load
    ```

3. ページUV数を計算します。

    ```sql
    SELECT page_id, count(distinct visit_users) FROM page_uv GROUP BY page_id;
    +-----------+------------------------------+
    |  page_id  | count(DISTINCT `visit_users`)|
    +-----------+------------------------------+
    |         1 |                            3 |
    |         2 |                            1 |
    +-----------+------------------------------+
    2 row in set (0.00 sec)
    ```

## グローバル辞書

現在、ビットマップベースのCount Distinctメカニズムは入力を整数型である必要があります。ユーザーがビットマップに他のデータ型を使用する必要がある場合、他の種類のデータ（文字列型など）を整数型にマッピングするために、ユーザー自身がグローバル辞書を構築する必要があります。グローバル辞書を構築するためのいくつかのアイデアがあります。

### Hiveテーブルベースのグローバル辞書

このスキームのグローバル辞書自体はHiveテーブルであり、2つの列を持ちます。1つは元の値、もう1つはエンコードされた整数値です。グローバル辞書を生成する手順は以下のとおりです。

1. 辞書の列を重複削除して一時テーブルを生成します。
2. 一時テーブルとグローバル辞書を左結合し、一時テーブルに`新しい値`を追加します。
3. `新しい値`をエンコードしてグローバル辞書に挿入します。
4. ファクトテーブルと更新されたグローバル辞書を左結合し、IDで辞書の項目を置換します。

この方法では、グローバル辞書は更新され、ファクトテーブル内の値列がSparkまたはMRを使用して置換されます。トライ木に基づくグローバル辞書に比べて、このアプローチは分散でき、グローバル辞書を再利用できます。

ただし、注意すべき点がいくつかあります。元のファクトテーブルが複数回読み込まれ、グローバル辞書の計算中に多くの余分なリソースを消費する2つの結合があります。

### トライ木に基づくグローバル辞書の構築
```
Users can also build their own global dictionaries using trie trees (aka prefix trees or dictionary trees). The trie tree has common prefixes for the descendants of nodes, which can be used to reduce query time and minimize string comparisons, and therefore is well suited for implementing dictionary encoding. However, the implementation of trie tree is not easy to distribute and can create performance bottlenecks when the data volume is relatively large.

By building a global dictionary and converting other types of data to integer data, you can use Bitmap to perform accurate Count Distinct analysis of non-integer data columns.
```