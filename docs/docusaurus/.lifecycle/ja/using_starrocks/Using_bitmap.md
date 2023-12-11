---
displayed_sidebar: "Japanese"
---

# 正確なCount Distinctのためのビットマップの使用

本トピックでは、StarRocksでの異なる値の数を計算するためにビットマップを使用する方法について説明します。

ビットマップは配列内の異なる値の数を計算するための有用なツールです。この方法は、従来のCount Distinctと比較して、より少ないストレージスペースを使用し、計算を加速させることができます。[0, n)の値範囲を持つ配列Aがあると仮定します。(n+7)/8バイトのビットマップを使用することで、配列内の異なる要素の数を計算できます。これには、すべてのビットを0に初期化し、要素の値をビットの添字として設定し、その後すべてのビットを1に設定します。ビットマップ内の1の数が配列内の異なる要素の数です。

## 従来のCount Distinct

StarRocksはMPPアーキテクチャを使用しており、Count Distinctを使用する際に詳細なデータを維持できます。ただし、Count Distinct機能はクエリ処理中に複数のデータシャッフルを必要とし、データ量が増加すると性能が線形に低下するため、より多くのリソースを消費します。

以下のシナリオでは、テーブル（dt、page、user_id）の詳細データを基にUVを計算しています。

|  dt   |   page  | user_id |
| :---: | :---: | :---:|
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 102 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |

StarRocksは、データを以下の図のように処理します。まず、データを`page`および`user_id`の列でグループ化し、その後処理結果を数えます。

![alter](../assets/6.1.2-2.png)

* 注: 図は2つのBEノードで計算された6行のデータの概略を示しています。

大容量のデータを処理する際に複数のシャッフル操作が必要な場合、必要な計算リソースが大幅に増加する可能性があります。これによりクエリの遅延が発生します。しかし、ビットマップ技術を使用すると、この問題を解決し、このようなシナリオでクエリのパフォーマンスを向上させることができます。

`page`をグループ化して`uv`を数える：

```sql
select page, count(distinct user_id) as uv from table group by page;

|  page   |   uv  |
| :---: | :---: |
|   game  |  1   |
|   shopping  |   2  |
```

## ビットマップを使用したCount Distinctの利点

COUNT(DISTINCT expr)と比較して、ビットマップを使用することで以下のような利点が得られます：

* ストレージスペースを節約: INT32データの異なる値の数をビットマップで計算する場合、必要なストレージスペースはCOUNT(DISTINCT expr)の1/32しか必要ありません。StarRocksでは、圧縮されたローリングビットマップを使用して計算を実行し、従来のビットマップと比較してストレージスペースの利用がさらに削減されています。
* 高速な計算: ビットマップはビット演算を使用するため、COUNT(DISTINCT expr)と比較してより高速な計算が可能です。StarRocksでは、異なる値の数の計算を並列処理でき、クエリのパフォーマンスがさらに向上します。

Roaring Bitmapの実装については、[特定の論文と実装](https://github.com/RoaringBitmap/RoaringBitmap)を参照してください。

## 使用上の注意

* ビットマップインデックスおよびビットマップCount Distinctの両方がビットマップ技術を使用しています。ただし、それらを導入する目的と解決する問題は完全に異なります。前者は低基数の列をフィルタリングするために使用され、後者はデータ行の値列内の異なる要素の数を計算するために使用されます。
* StarRocks 2.3以降のバージョンでは、テーブルの種類（アグリゲートテーブル、デュプリケートキーテーブル、プライマリキーテーブル、またはユニークキーテーブル）に関係なく、値列をBITMAPタイプとして定義できます。ただし、テーブルの[ソートキー](../table_design/Sort_key.md)はBITMAPタイプにできません。
* テーブルを作成する際、値列をBITMAPとして定義し、集約関数を[BITMAP_UNION](../sql-reference/sql-functions/bitmap-functions/bitmap_union.md)にすることができます。
* ビットマップを使用して異なる値の数を計算するためには、TINYINT、SMALLINT、INT、BIGINTのデータのみを使用できます。他のタイプのデータについては、[グローバル辞書を構築](#global-dictionary)する必要があります。

## ビットマップを使用したCount Distinct

ページUVの計算を例に挙げます。

1. BITMAP_UNIONを使用するBITMAP列`visit_users`を持つアグリゲートテーブルを作成します。

    ```sql
    CREATE TABLE `page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT 'アクセス時間',
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

    データをロードした後：

    * `page_id = 1, visit_date = '2020-06-23 01:30:30'`の行では、`visit_users`フィールドに3つのビットマップ要素（13、23、33）が含まれています。
    * `page_id = 1, visit_date = '2020-06-23 02:30:30'`の行では、`visit_users`フィールドに1つのビットマップ要素（13）が含まれています。
    * `page_id = 2, visit_date = '2020-06-23 01:30:30'`の行では、`visit_users`フィールドに1つのビットマップ要素（23）が含まれています。

   ローカルファイルからデータをロードする：

    ```shell
    echo -e '1,2020-06-23 01:30:30,130\n1,2020-06-23 01:30:30,230\n1,2020-06-23 01:30:30,120\n1,2020-06-23 02:30:30,133\n2,2020-06-23 01:30:30,234' > tmp.csv | 
    curl --location-trusted -u <username>:<password> -H "label:label_1600960288798" \
        -H "column_separator:," \
        -H "columns:page_id,visit_date,visit_users, visit_users=to_bitmap(visit_users)" -T tmp.csv \
        http://StarRocks_be0:8040/api/db0/page_uv/_stream_load
    ```

3. ページUVを計算します。

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

現在、ビットマップベースのCount Distinctメカニズムでは、入力を整数として必要とします。ユーザーがビットマップに他のデータ型を使用する必要がある場合、ユーザーは他のデータ（文字列型など）を整数型にマップするためのグローバル辞書を構築する必要があります。グローバル辞書を構築するためのいくつかのアイデアがあります。

### Hiveテーブルベースのグローバル辞書

このスキームのグローバル辞書自体はHiveテーブルであり、2つの列を持ちます。1つは元の値、もう1つはエンコードされた整数値用です。グローバル辞書を生成する手順は次のとおりです：

1. ファクトテーブルの辞書の列を重複排除して一時テーブルを生成します
2. 一時テーブルとグローバル辞書を左結合し、`新しい値`を一時テーブルに追加します。
3. `新しい値`をエンコードし、それをグローバル辞書に挿入します。
4. ファクトテーブルと更新されたグローバル辞書を左結合し、IDで辞書項目を置換します。

この方法で、グローバル辞書が更新され、ファクトテーブルの値列がSparkまたはMRを使用して置換されることができます。トライ木ベースのグローバル辞書と比較して、この方法は分散可能であり、グローバル辞書を再利用できます。

ただし、注意する点がいくつかあります。元のファクトテーブルが複数回読み取られ、グローバル辞書の計算中に多くの余分なリソースを消費する2つの結合があるためです。

### トライ木に基づくグローバル辞書を構築
```
Users can also build their own global dictionaries using trie trees (aka prefix trees or dictionary trees). The trie tree has common prefixes for the descendants of nodes, which can be used to reduce query time and minimize string comparisons, and therefore is well suited for implementing dictionary encoding. However, the implementation of trie tree is not easy to distribute and can create performance bottlenecks when the data volume is relatively large.

By building a global dictionary and converting other types of data to integer data, you can use Bitmap to perform accurate Count Distinct analysis of non-integer data columns.
```