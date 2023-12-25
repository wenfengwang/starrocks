---
displayed_sidebar: Chinese
---

# Bitmapを使用した正確な重複排除

この記事では、Bitmapを使用して正確な重複排除（Exact Count Distinct）を実現する方法について説明します。

Bitmapによる重複排除は、データセット内の重複しない要素の数を正確に計算でき、従来のCount Distinctと比較して、ストレージスペースを節約し、計算を高速化できます。例えば、配列Aがあり、その値の範囲が[0, n)である場合、(n+7)/8バイト長のbitmapを使用して配列Aの重複を排除できます。つまり、すべてのbitを0に初期化し、次に配列Aの要素の値をbitのインデックスとして使用し、そのbitを1に設定します。すると、bitmap内の1の数が配列Aの異なる要素（Count Distinct）の数になります。

## 伝統的なCount distinct

StarRocksはMPPアーキテクチャに基づいて実装されており、count distinctを使用して正確な重複排除を行う場合、詳細データを保持し、柔軟性が高いです。しかし、クエリ実行中に複数回のデータシャッフル（異なるノード間でのデータ転送、重複排除の計算）が必要になるため、データ量が増加するにつれてパフォーマンスが直線的に低下します。

例えば、以下の詳細データを使用してテーブル（dt, page, user_id）の各ページのUVを計算します。

|  dt   |   page  | user_id |
| :---: | :---: | :---:|
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 102 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |
|   20191206  |   game  | 101 |
|   20191206  |   shopping  | 101 |

StarRocksは計算時に以下の図のように計算を行い、まずpage列とuser_id列でgroup byを行い、最後にcountします。

![変更](../assets/6.1.2-2.png)

> 注：図は2つのBEノード上で6行のデータを計算する概念図です。

上記の計算方法では、データが複数回シャッフルされる必要があるため、データ量が増加すると必要な計算リソースが増加し、クエリも遅くなります。Bitmapを使用した重複排除は、大量のデータシナリオでの伝統的なcount distinctのパフォーマンス問題を解決するためにあります。

```sql
select page, count(distinct user_id) as uv from table group by page;

|  page      |   uv  |
| :---:      | :---: |
|  game      |  1    |
|  shopping  |   2   |
```

## Bitmapによる重複排除の利点

伝統的な[count distinct](#伝統的なCount-distinct)方式と比較して、Bitmapの利点は主に以下の2点に表れます：

- ストレージスペースの節約：Bitmapの1ビットを使用して対応するインデックスの存在を表すことで、大量のストレージスペースを節約できます。例えば、INT32型のデータを重複排除する場合、通常のbitmapを使用すれば、COUNT(DISTINCT expr)に必要なストレージスペースの1/32しか占めません。StarRocksは非常に巧妙に設計されたbitmap、Roaring Bitmapを使用しており、通常のBitmapと比較してさらにメモリ使用量を削減できます。
- 計算の高速化：Bitmapによる重複排除はビット演算を使用するため、COUNT(DISTINCT expr)よりも計算速度が速く、さらにStarRocksのMPP実行エンジンでは並列加速処理が可能で、計算速度を向上させることができます。

Roaring Bitmapの実装についての詳細は、[具体的な論文と実装](https://github.com/RoaringBitmap/RoaringBitmap)を参照してください。

## 使用説明

- Bitmap indexとBitmapによる重複排除はどちらもBitmap技術を使用していますが、導入理由と解決する問題が全く異なります。前者は低基数の列挙型列の等価条件フィルタリングに使用され、後者はデータ行のグループの指標列の重複しない要素の数を計算するために使用されます。
- StarRocksのバージョン2.3から、すべてのデータモデルテーブルの指標列はBITMAP型として設定することができますが、すべてのデータモデルテーブルで[ソートキー](../table_design/Sort_key.md)をBITMAP型として設定することはできません。
- テーブル作成時に、指標列の型をBITMAPとして指定し、[BITMAP_UNION](../sql-reference/sql-functions/bitmap-functions/bitmap_union.md)関数を使用してデータを集約します。
- StarRocksのBitmapによる重複排除はRoaring Bitmapに基づいて実装されており、Roaring BitmapはTINYINT、SMALLINT、INT、BIGINT型のデータのみを重複排除できます。他のタイプのデータをRoaring Bitmapで重複排除する場合は、グローバル辞書を構築する必要があります。

## Bitmapによる重複排除の使用例

あるページのユニークビジター数（UV）を統計する例を以下に示します：

1. 集約テーブル`page_uv`を作成します。ここでの`visit_users`列は訪問ユーザーのIDを表し、集約列であり、列のタイプはBITMAPで、集約関数BITMAP_UNIONを使用してデータを集約します。

    ```sql
    CREATE TABLE `page_uv` (
      `page_id` INT NOT NULL COMMENT 'ページID',
      `visit_date` datetime NOT NULL COMMENT '訪問日時',
      `visit_users` BITMAP BITMAP_UNION NOT NULL COMMENT '訪問ユーザーID'
    ) ENGINE=OLAP
    AGGREGATE KEY(`page_id`, `visit_date`)
    DISTRIBUTED BY HASH(`page_id`)
    PROPERTIES (
      "replication_num" = "3",
      "storage_format" = "DEFAULT"
    );
    ```

2. テーブルにデータをインポートします。

    INSERT INTO文を使用してインポートします：

    ```sql
    INSERT INTO page_uv VALUES
    (1, '2020-06-23 01:30:30', to_bitmap(13)),
    (1, '2020-06-23 01:30:30', to_bitmap(23)),
    (1, '2020-06-23 01:30:30', to_bitmap(33)),
    (1, '2020-06-23 02:30:30', to_bitmap(13)),
    (2, '2020-06-23 01:30:30', to_bitmap(23));
    ```

    データインポート後：

    - page_id = 1、visit_date = '2020-06-23 01:30:30'のデータ行では、`visit_users`フィールドに3つのbitmap要素（13、23、33）が含まれています。
    - page_id = 1、visit_date = '2020-06-23 02:30:30'のデータ行では、`visit_users`フィールドに1つのbitmap要素（13）が含まれています。
    - page_id = 2、visit_date = '2020-06-23 01:30:30'のデータ行では、`visit_users`フィールドに1つのbitmap要素（23）が含まれています。

    ローカルファイルからのインポートを使用します：

    ```shell
    echo -e '1,2020-06-23 01:30:30,130\n1,2020-06-23 01:30:30,230\n1,2020-06-23 01:30:30,120\n1,2020-06-23 02:30:30,133\n2,2020-06-23 01:30:30,234' > tmp.csv | 
    curl --location-trusted -u <username>:<password> -H "label:label_1600960288798" \
        -H "column_separator:," \
        -H "columns:page_id,visit_date,visit_users, visit_users=to_bitmap(visit_users)" -T tmp.csv \
        http://StarRocks_be0:8040/api/db0/page_uv/_stream_load
    ```

3. 各ページのUVを統計します。

    ```sql
    select page_id, count(distinct visit_users) from page_uv group by page_id;

    +-----------+------------------------------+
    |  page_id  | count(DISTINCT `visit_users`) |
    +-----------+------------------------------+
    |         1 |                            3 |
    +-----------+------------------------------+
    |         2 |                            1 |
    +-----------+------------------------------+
    2 rows in set (0.00 sec)

    ```

## Bitmapグローバル辞書

現在、Bitmap型に基づく重複排除メカニズムには一定の制限があり、Bitmapは整数型のデータを入力として使用する必要があります。他のデータタイプをBitmapの入力として使用したい場合は、グローバル辞書を構築して、他のタイプのデータ（例えば文字列タイプ）を整数タイプにマッピングする必要があります。グローバル辞書を構築するには以下の方法があります：

### Hiveテーブルに基づくグローバル辞書

この方法では、グローバル辞書としてHiveテーブルを作成する必要があります。Hiveテーブルには2つの列があり、1つは元の値、もう1つはエンコードされたInt値です。以下はグローバル辞書を生成する手順です：

1. ファクトテーブルの辞書列を重複排除して一時テーブルを生成します。
2. 一時テーブルとグローバル辞書をleft joinし、新しい値を新しいvalueとして使用します。
3. 新しいvalueにエンコードを施し、グローバル辞書に挿入します。
4. ファクトテーブルと更新されたグローバル辞書をleft joinし、辞書項目をIDに置き換えます。

この方法でグローバル辞書を構築すると、SparkまたはMapReduceを使用してグローバル辞書の更新とファクトテーブルのValue列の置き換えを実現できます。Trieツリーに基づくグローバル辞書と比較して、この方法は分散化が可能で、グローバル辞書の再利用も実現できます。

ただし、この方法でグローバル辞書を構築する際には、ファクトテーブルが複数回読み込まれ、プロセス中に2回のJoin操作が発生するため、グローバル辞書の計算に多くの追加リソースが必要になることに注意が必要です。

### Trieツリーに基づくグローバル辞書

Trieツリーは、プレフィックスツリーまたは辞書ツリーとも呼ばれます。Trieツリーのノードの子孫には共通のプレフィックスが存在し、システムは文字列の共通プレフィックスを利用して検索時間を短縮し、文字列比較を最小限に抑えることができます。したがって、Trieツリーに基づいてグローバル辞書を構築する方法は、辞書エンコーディングを実現するのに適しています。しかし、Trieツリーに基づくグローバル辞書の実装は分散化が難しく、データ量が多い場合にはパフォーマンスのボトルネックが発生する可能性があります。
