---
displayed_sidebar: "Japanese"
---

# TPC-DSベンチマーク

TPC-DSは、Transaction Processing Performance Council（TPC）によって開発された意思決定支援ベンチマークです。TPC-Hよりも包括的なテストデータセットと複雑なSQLクエリを使用しています。

TPC-DSは、意思決定支援システムのいくつかの一般的に適用可能な側面をモデル化しており、クエリとデータのメンテナンスを含んでいます。TPC-DSは、小売業の環境でデータベースシステムのパフォーマンスをテストおよび評価するための包括的で現実的なワークロードを提供することを目指しています。TPC-DSベンチマークは、小売企業の3つの販売チャネル（店舗、インターネット、カタログ）の販売および返品データをシミュレートしています。販売および返品データモデルのテーブルを作成するだけでなく、簡単な在庫システムとプロモーションシステムも含まれています。

このベンチマークでは、データサイズが1GBから3GBまでの24のテーブルに対して合計99の複雑なSQLクエリをテストします。主なパフォーマンス指標は、各クエリの応答時間であり、クエリが送信されてから結果が返されるまでの時間です。

## 1. テスト結論

TPC-DS 100GBのデータセットに対して99のクエリをテストしました。以下の図は、テスト結果を示しています。
![tpc-ds](../assets/tpc-ds.png)

テストでは、StarRocksはネイティブストレージとHive外部テーブルの両方からデータをクエリします。StarRocksとTrinoは、Hive外部テーブルから同じデータコピーをクエリします。データはLZ4圧縮され、Parquet形式で保存されています。

StarRocksがネイティブストレージからデータをクエリするレイテンシは**174秒**、Hive外部テーブルからデータをクエリするレイテンシは**239秒**、データキャッシュ機能を有効にした状態でHive外部テーブルからデータをクエリするレイテンシは**176秒**、TrinoがHive外部テーブルからデータをクエリするレイテンシは**892秒**です。

## 2. テストの準備

### 2.1 ハードウェア環境

| マシン           | クラウドホスト4台                            |
| ----------------- | ---------------------------------------------- |
| CPU               | 8コア Intel(R) Xeon(R) Platinum 8269CY CPU @ 2.50GHz |
| メモリ            | 32 GB                                          |
| ネットワーク帯域幅 | 5 Gbit/s                                       |
| ディスク          | ESSD                                           |

### 2.2 ソフトウェア環境

StarRocksとTrinoは、同じ構成のマシンに展開されています。StarRocksには1つのFEと3つのBEが展開されています。Trinoには1つのコーディネーターと3つのワーカーが展開されています。

- カーネルバージョン：Linux 3.10.0-1127.13.1.el7.x86_64
- OSバージョン：CentOS Linux released 7.8.2003
- ソフトウェアバージョン：StarRocks Community Edition 3.1、Trino-419、Hive-3.1.2

> StarRocks FEは、別々に展開するか、BEとハイブリッド展開することができますが、テスト結果には影響しません。

## 3. テストデータと結果

### 3.1 テストデータ

| テーブル                  | レコード数   |
| -------------          | --------- |
| call_center            | 30        |
| catalog_page           | 20400     |
| catalog_returns        | 14404374  |
| catalog_sales          | 143997065 |
| customer_address       | 1000000   |
| customer_demographics  | 1920800   |
| customer               | 2000000   |
| date_dim               | 73049     |
| household_demographics | 7200      |
| income_band            | 20        |
| inventory              | 399330000 |
| item                   | 204000    |
| promotion              | 1000      |
| reason                 | 55        |
| ship_mode              | 20        |
| store                  | 402       |
| store_returns          | 28795080  |
| store_sales            | 287997024 |
| time_dim               | 86400     |
| warehouse              | 15        |
| web_page               | 2040      |
| web_returns            | 7197670   |
| web_sales              | 72001237  |
| web_site               | 24        |

### 3.2 テスト結果

> **注意**
>
> - 以下の表でクエリのレイテンシの単位はミリ秒です。
> - `StarRocks-3.0.5-native`はStarRocksのネイティブストレージを示し、`StarRocks-3.0-Hive external`はStarRocksがHive外部テーブルをHiveカタログを介してクエリすることを示し、`StarRocks-3.0-Hive external-Cache`はStarRocksがHive外部テーブルをHiveカタログを介してデータキャッシュを有効にしてクエリすることを示します。
> - StarRocksでは集約プッシュダウンが有効になっています（`SET global cbo_push_down_aggregate_mode = 0`）。

| **クエリ** | **StarRocks-3.0.5-native** | **StarRocks-3.0-Hive external** | **StarRocks-3.0-Hive external-Cache** | **Trino-419** |
| --------- | -------------------------- | ------------------------------- | ------------------------------------- | ------------- |
| SUM       | 174157                     | 238590                          | 175970                                | 891892        |
| Q1        | 274                        | 780                             | 254                                   | 1681          |
| Q2        | 338                        | 676                             | 397                                   | 10200         |
| Q3        | 455                        | 1156                            | 628                                   | 3156          |
| Q4        | 16180                      | 13229                           | 12623                                 | 48176         |
| Q5        | 1162                       | 773                             | 506                                   | 4490          |
| Q6        | 397                        | 606                             | 165                                   | 1349          |
| Q7        | 898                        | 1707                            | 724                                   | 2300          |
| Q8        | 532                        | 447                             | 141                                   | 2330          |
| Q9        | 2113                       | 7998                            | 6336                                  | 17734         |
| Q10       | 588                        | 847                             | 285                                   | 2498          |
| Q11       | 6465                       | 5086                            | 4665                                  | 31333         |
| Q12       | 149                        | 302                             | 135                                   | 728           |
| Q13       | 1573                       | 2661                            | 1349                                  | 4370          |
| Q14       | 7928                       | 7811                            | 5955                                  | 69729         |
| Q15       | 323                        | 461                             | 199                                   | 1522          |
| Q16       | 639                        | 1278                            | 661                                   | 3282          |
| Q17       | 1157                       | 898                             | 682                                   | 4102          |
| Q18       | 540                        | 1746                            | 521                                   | 2471          |
| Q19       | 667                        | 639                             | 230                                   | 1701          |
| Q20       | 209                        | 369                             | 144                                   | 849           |
| Q21       | 466                        | 586                             | 306                                   | 1591          |
| Q22       | 3876                       | 4704                            | 4536                                  | 17422         |
| Q23       | 24500                      | 24746                           | 21707                                 | 145850        |
| Q24       | 1256                       | 5220                            | 3219                                  | 21234         |
| Q25       | 1037                       | 792                             | 542                                   | 3702          |
| Q26       | 393                        | 834                             | 360                                   | 1737          |
| Q27       | 742                        | 1303                            | 696                                   | 2396          |
| Q28       | 1864                       | 8600                            | 6564                                  | 15837         |
| Q29       | 1097                       | 1134                            | 888                                   | 4024          |
| Q30       | 194                        | 669                             | 242                                   | 1922          |
| Q31       | 1149                       | 1070                            | 834                                   | 5431          |
| Q32       | 222                        | 718                             | 104                                   | 1706          |
| Q33       | 922                        | 735                             | 327                                   | 2048          |
| Q34       | 544                        | 1392                            | 576                                   | 3185          |
| Q35       | 974                        | 897                             | 574                                   | 3050          |
| Q36       | 630                        | 1009                            | 464                                   | 3056          |
| Q37       | 246                        | 791                             | 273                                   | 3258          |
| Q38       | 2831                       | 2017                            | 1695                                  | 10913         |
| Q39       | 1057                       | 2312                            | 1324                                  | 10665         |
| Q40       | 331                        | 560                             | 209                                   | 2678          |
| Q41       | 57                         | 148                             | 79                                    | 776           |
| Q42       | 463                        | 559                             | 106                                   | 1213          |
| Q43       | 885                        | 602                             | 342                                   | 2914          |
| Q44       | 506                        | 3783                            | 2306                                  | 9705          |
| Q45       | 439                        | 777                             | 309                                   | 1012          |
| Q46       | 868                        | 1746                            | 1037                                  | 4766          |
| Q47       | 1816                       | 2979                            | 2684                                  | 19111         |
| Q48       | 635                        | 2038                            | 1202                                  | 3635          |
| Q49       | 1440                       | 2754                            | 1168                                  | 3435          |
| Q50       | 836                        | 2053                            | 1305                                  | 4375          |
| Q51       | 3966                       | 5258                            | 4466                                  | 14283         |
| Q52       | 483                        | 436                             | 100                                   | 1126          |
| Q53       | 698                        | 802                             | 391                                   | 1648          |
| Q54       | 794                        | 970                             | 534                                   | 5146          |
| Q55       | 463                        | 540                             | 97                                    | 963           |
| Q56       | 874                        | 695                             | 240                                   | 2110          |
| Q57       | 1717                       | 2723                            | 2372                                  | 10203         |
| Q58       | 554                        | 727                             | 242                                   | 2053          |
| Q59       | 2764                       | 1581                            | 1368                                  | 15697         |
| Q60       | 1053                       | 557                             | 389                                   | 2421          |
| Q61       | 1353                       | 1026                            | 439                                   | 2334          |
| Q62       | 453                        | 659                             | 427                                   | 2422          |
| Q63       | 709                        | 943                             | 374                                   | 1624          |
| Q64       | 3209                       | 6968                            | 6175                                  | 31994         |
| Q65       | 2147                       | 3043                            | 2451                                  | 9334          |
| Q66       | 688                        | 805                             | 437                                   | 2598          |
| Q67       | 15486                      | 23743                           | 21975                                 | 58091         |
| Q68       | 965                        | 1702                            | 776                                   | 2710          |
| Q69       | 600                        | 703                             | 263                                   | 2872          |
| Q70       | 2376                       | 2217                            | 1588                                  | 10272         |
| Q71       | 702                        | 691                             | 348                                   | 3074          |
| Q72       | 1764                       | 2733                            | 2305                                  | 13973         |
| Q73       | 576                        | 1145                            | 484                                   | 1899          |
| Q74       | 4615                       | 3884                            | 3776                                  | 18749         |
| Q75       | 2661                       | 3479                            | 3137                                  | 10858         |
| Q76       | 450                        | 2001                            | 1014                                  | 5297          |
| Q77       | 1109                       | 743                             | 317                                   | 2810          |
| Q78       | 6540                       | 7198                            | 5890                                  | 19671         |
| Q79       | 1116                       | 1953                            | 1121                                  | 4406          |
| Q80       | 2290                       | 1973                            | 1480                                  | 5865          |
| Q81       | 247                        | 1024                            | 317                                   | 1729          |
| Q82       | 392                        | 929                             | 407                                   | 3605          |
| Q83       | 134                        | 313                             | 158                                   | 1209          |
| Q84       | 107                        | 820                             | 228                                   | 2448          |
| Q85       | 460                        | 2045                            | 621                                   | 4311          |
| Q86       | 433                        | 999                             | 387                                   | 1693          |
| Q87       | 2873                       | 2159                            | 1779                                  | 10709         |
| Q88       | 3616                       | 7076                            | 5432                                  | 26002         |
| Q89       | 735                        | 785                             | 454                                   | 1997          |
| Q90       | 174                        | 898                             | 232                                   | 2585          |
| Q91       | 113                        | 495                             | 139                                   | 1745          |
| Q92       | 203                        | 627                             | 91                                    | 1016          |
| Q93       | 529                        | 2508                            | 1422                                  | 12265         |
| Q94       | 475                        | 811                             | 598                                   | 2153          |
| Q95       | 1059                       | 1993                            | 1526                                  | 8058          |
| Q96       | 395                        | 1197                            | 681                                   | 3976          |
| Q97       | 3000                       | 3459                            | 2860                                  | 6818          |
| Q98       | 419                        | 486                             | 344                                   | 2090          |
| Q99       | 755                        | 1070                            | 740                                   | 4332          |

## 4. テスト手順

### 4.1 StarRocksネイティブテーブルのクエリ

#### 4.1.1 データの生成

tpcds-pocツールキットをダウンロードし、標準のTPC-DSテストデータセット `スケールファクター=100` を生成します。

```Bash
wget https://starrocks-public.oss-cn-zhangjiakou.aliyuncs.com/tpcds-poc-1.0.zip
unzip tpcds-poc-1.0
cd tpcds-poc-1.0

sh bin/gen_data/gen-tpcds.sh 100 data_100
```

#### 4.1.2 テーブルスキーマの作成

構成ファイル `conf/starrocks.conf` を変更し、クラスターアドレスを指定します。`mysql_host` と `mysql_port` に注意してください。

```Bash
sh bin/create_db_table.sh ddl_100
```

#### 4.1.3 データのロード

```Bash
sh bin/stream_load.sh data_100
```

#### 4.1.4 データのクエリ

```Bash
sh bin/benchmark.sh -p -d tpcds
```

### 4.2 Hive外部テーブルのクエリ

#### 4.2.1 テーブルスキーマの作成

Hiveで外部テーブルを作成し、ストレージ形式をParquet、圧縮形式をLZ4に指定します。詳細なCREATE TABLEステートメントについては、[Hive外部テーブルの作成（Parquet）](#53-create-a-hive-external-tableparquet)を参照してください。StarRocksとTrinoは、これらの外部テーブルからデータをクエリします。

#### 4.2.2 データのロード

[4.1.1](#411-generate-data)で生成されたCSVデータをHiveの指定したHDFSパスにロードします。この例では、HDFSパスとして`/user/tmp/csv/`を使用します。HiveでCSV形式の外部テーブルを作成し、テーブルを`/user/tmp/csv/`に保存します。詳細なCREATE TABLEステートメントについては、[Hive外部テーブルの作成（CSV）](#54-create-a-hive-external-tablecsv)を参照してください。

CSV形式の外部テーブルからParquet形式の外部テーブルにデータをロードします。これにより、LZ4圧縮されたParquetデータが生成されます。

```SQL
use tpcds_100g_parquet_lz4;

insert into call_center select * from tpcds_100g_csv.call_center order by cc_call_center_sk;
insert into catalog_page select * from tpcds_100g_csv.catalog_page order by cp_catalog_page_sk;
insert into catalog_returns select * from tpcds_100g_csv.catalog_returns order by cr_returned_date_sk, cr_item_sk;
insert into date_dim select * from tpcds_100g_csv.date_dim order by D_DATE_SK;
insert into household_demographics select * from tpcds_100g_csv.household_demographics order by HD_DEMO_SK;
insert into income_band select * from tpcds_100g_csv.income_band order by IB_INCOME_BAND_SK;
insert into item select * from tpcds_100g_csv.item order by I_ITEM_SK;
insert into promotion select * from tpcds_100g_csv.promotion  order by P_PROMO_SK;
insert into reason select * from tpcds_100g_csv.reason a order by a.R_REASON_SK;
insert into ship_mode select * from tpcds_100g_csv.ship_mode order by SM_SHIP_MODE_SK;
insert into store select * from tpcds_100g_csv.store order by S_STORE_SK;
insert into time_dim select * from tpcds_100g_csv.time_dim order by T_TIME_SK;
insert into warehouse select * from tpcds_100g_csv.warehouse order by W_WAREHOUSE_SK;
insert into web_page select * from tpcds_100g_csv.web_page order by WP_WEB_PAGE_SK;
insert into web_site select * from tpcds_100g_csv.web_site order by WEB_SITE_SK;
insert into customer_demographics select * from tpcds_100g_csv.customer_demographics order by CD_DEMO_SK;
insert into customer select * from tpcds_100g_csv.customer order by C_CUSTOMER_SK;
insert into customer_address select * from tpcds_100g_csv.customer_address order by CA_ADDRESS_SK;
insert into web_sales select * from tpcds_100g_csv.web_sales order by WS_SOLD_DATE_SK, WS_ITEM_SK;
insert into web_returns select * from tpcds_100g_csv.web_returns order by WR_RETURNED_DATE_SK, WR_ITEM_SK;
insert into inventory select * from tpcds_100g_csv.inventory order by INV_DATE_SK, INV_ITEM_SK;
insert into catalog_sales select * from tpcds_100g_csv.catalog_sales order by CS_SOLD_DATE_SK, CS_ITEM_SK;
insert into store_returns select * from tpcds_100g_csv.store_returns order by SR_RETURNED_DATE_SK, SR_ITEM_SK;
insert into store_sales select * from tpcds_100g_csv.store_sales order by SS_SOLD_DATE_SK, SS_ITEM_SK;
```

#### 4.2.3 統計情報の収集

StarRocks v3.0では、外部テーブルからの統計情報の収集はサポートされていません。そのため、列レベルの統計情報を収集するためにHiveを使用します。

```SQL
use tpcds_100g_parquet_lz4;

analyze table call_center compute statistics FOR COLUMNS;
analyze table catalog_page compute statistics FOR COLUMNS;
analyze table catalog_returns compute statistics FOR COLUMNS;
analyze table catalog_sales compute statistics FOR COLUMNS;
analyze table customer compute statistics FOR COLUMNS;
analyze table customer_address compute statistics FOR COLUMNS;
analyze table customer_demographics compute statistics FOR COLUMNS;
analyze table date_dim compute statistics FOR COLUMNS;
analyze table household_demographics compute statistics FOR COLUMNS;
analyze table income_band compute statistics FOR COLUMNS;
analyze table inventory compute statistics FOR COLUMNS;
analyze table item compute statistics FOR COLUMNS;
analyze table promotion compute statistics FOR COLUMNS;
analyze table reason compute statistics FOR COLUMNS;
analyze table ship_mode compute statistics FOR COLUMNS;
analyze table store compute statistics FOR COLUMNS;
analyze table store_returns compute statistics FOR COLUMNS;
analyze table store_sales compute statistics FOR COLUMNS;
analyze table time_dim compute statistics FOR COLUMNS;
analyze table warehouse compute statistics FOR COLUMNS;
analyze table web_page compute statistics FOR COLUMNS;
analyze table web_returns compute statistics FOR COLUMNS;
analyze table web_sales compute statistics FOR COLUMNS;
analyze table web_site compute statistics FOR COLUMNS;
```

#### 4.2.4 データのクエリ

StarRocksは[Hiveカタログ](../data_source/catalog/hive_catalog.md)を使用してHive外部テーブルをクエリします。

テスト中にStarRocksで[データキャッシュ](../data_source/data_cache.md)を有効にする場合、データキャッシュ機能に対して以下の設定が推奨されます。

```SQL
block_cache_mem_size = 5368709120
block_cache_disk_size = 193273528320
```

## 5. クエリステートメントとCREATE TABLEステートメント

### 5.1 TPC-DS SQLクエリ

99のSQLクエリの詳細については、[TPC-DS SQL](./tpc_ds_99_sql.md)を参照してください。

### 5.2 StarRocksネイティブテーブルの作成

```SQL
create table call_center
(
    cc_call_center_sk         integer               not null,
    cc_call_center_id         char(16)              not null,
    cc_rec_start_date         date                          ,
    cc_rec_end_date           date                          ,
    cc_closed_date_sk         integer                       ,
    cc_open_date_sk           integer                       ,
    cc_name                   varchar(50)                   ,
    cc_class                  varchar(50)                   ,
    cc_employees              integer                       ,
    cc_sq_ft                  integer                       ,
    cc_hours                  char(20)                      ,
    cc_manager                varchar(40)                   ,
    cc_mkt_id                 integer                       ,
    cc_mkt_class              char(50)                      ,
    cc_mkt_desc               varchar(100)                  ,
    cc_market_manager         varchar(40)                   ,
    cc_division               integer                       ,
    cc_division_name          varchar(50)                   ,
    cc_company                integer                       ,
    cc_company_name           char(50)                      ,
    cc_street_number          char(10)                      ,
    cc_street_name            varchar(60)                   ,
    cc_street_type            char(15)                      ,
    cc_suite_number           char(10)                      ,
    cc_city                   varchar(60)                   ,
    cc_county                 varchar(30)                   ,
    cc_state                  char(2)                       ,
    cc_zip                    char(10)                      ,
    cc_country                varchar(20)                   ,
    cc_gmt_offset             decimal(5,2)                  ,
    cc_tax_percentage         decimal(5,2)
)
duplicate key (cc_call_center_sk)
distributed by hash(cc_call_center_sk) buckets 1
properties(
    "replication_num" = "1"
);

create table catalog_page
(
    cp_catalog_page_sk        integer               not null,
    cp_catalog_page_id        char(16)              not null,
    cp_start_date_sk          integer                       ,
    cp_end_date_sk            integer                       ,
    cp_department             varchar(50)                   ,
    cp_catalog_number         integer                       ,
    cp_catalog_page_number    integer                       ,
    cp_description            varchar(100)                  ,
    cp_type                   varchar(100)
)
duplicate key (cp_catalog_page_sk)
distributed by hash(cp_catalog_page_sk) buckets 1
properties(
    "replication_num" = "1"
);

create table catalog_returns
(
    cr_item_sk                integer               not null,
    cr_order_number           integer               not null,
    cr_returned_date_sk       integer                       ,
    cr_returned_time_sk       integer                       ,
    cr_refunded_customer_sk   integer                       ,
    cr_refunded_cdemo_sk      integer                       ,
    cr_refunded_hdemo_sk      integer                       ,
    cr_refunded_addr_sk       integer                       ,
    cr_returning_customer_sk  integer                       ,
    cr_returning_cdemo_sk     integer                       ,
    cr_returning_hdemo_sk     integer                       ,
    cr_returning_addr_sk      integer                       ,
    cr_call_center_sk         integer                       ,
    cr_catalog_page_sk        integer                       ,
    cr_ship_mode_sk           integer                       ,
    cr_warehouse_sk           integer                       ,
    cr_reason_sk              integer                       ,
    cr_return_quantity        integer                       ,
    cr_return_amount          decimal(7,2)                  ,
    cr_return_tax             decimal(7,2)                  ,
    cr_return_amt_inc_tax     decimal(7,2)                  ,
    cr_fee                    decimal(7,2)                  ,
    cr_return_ship_cost       decimal(7,2)                  ,
    cr_refunded_cash          decimal(7,2)                  ,
    cr_reversed_charge        decimal(7,2)                  ,
    cr_store_credit           decimal(7,2)                  ,
    cr_net_loss               decimal(7,2)
)
duplicate key (cr_item_sk, cr_order_number)
distributed by hash(cr_item_sk, cr_order_number) buckets 16
properties(
    "replication_num" = "1"
);

create table catalog_sales
(
    cs_item_sk                integer               not null,
    cs_order_number           integer               not null,
    cs_sold_date_sk           integer                       ,
    cs_sold_time_sk           integer                       ,
    cs_ship_date_sk           integer                       ,
    cs_bill_customer_sk       integer                       ,
    cs_bill_cdemo_sk          integer                       ,
    cs_bill_hdemo_sk          integer                       ,
    cs_bill_addr_sk           integer                       ,
    cs_ship_customer_sk       integer                       ,
    cs_ship_cdemo_sk          integer                       ,
    cs_ship_hdemo_sk          integer                       ,
    cs_ship_addr_sk           integer                       ,
    cs_call_center_sk         integer                       ,
    cs_catalog_page_sk        integer                       ,
    cs_ship_mode_sk           integer                       ,
    cs_warehouse_sk           integer                       ,
    cs_promo_sk               integer                       ,
    cs_quantity               integer                       ,
    cs_wholesale_cost         decimal(7,2)                  ,
    cs_list_price             decimal(7,2)                  ,
    cs_sales_price            decimal(7,2)                  ,
    cs_ext_discount_amt       decimal(7,2)                  ,
    cs_ext_sales_price        decimal(7,2)                  ,
    cs_ext_wholesale_cost     decimal(7,2)                  ,
    cs_ext_list_price         decimal(7,2)                  ,
    cs_ext_tax                decimal(7,2)                  ,
    cs_coupon_amt             decimal(7,2)                  ,
    cs_ext_ship_cost          decimal(7,2)                  ,
    cs_net_paid               decimal(7,2)                  ,
    cs_net_paid_inc_tax       decimal(7,2)                  ,
    cs_net_paid_inc_ship      decimal(7,2)                  ,
    cs_net_paid_inc_ship_tax  decimal(7,2)                  ,
    cs_net_profit             decimal(7,2)
)
duplicate key (cs_item_sk, cs_order_number)
distributed by hash(cs_item_sk) buckets 192
properties(
    "replication_num" = "1"
);

create table customer_address
(
    ca_address_sk             integer               not null,
    ca_address_id             char(16)              not null,
    ca_street_number          char(10)                      ,
    ca_street_name            varchar(60)                   ,
    ca_street_type            char(15)                      ,
    ca_suite_number           char(10)                      ,
    ca_city                   varchar(60)                   ,
    ca_county                 varchar(30)                   ,
    ca_state                  char(2)                       ,
    ca_zip                    char(10)                      ,
    ca_country                varchar(20)                   ,
    ca_gmt_offset             decimal(5,2)                  ,
    ca_location_type          char(20)
)
duplicate key(ca_address_sk)
distributed by hash(ca_address_sk) buckets 10
properties(
    "replication_num" = "1"
);

create table customer_demographics
(
    cd_demo_sk                integer  not null,
    cd_gender                 char(1)  not null,
    cd_marital_status         char(1)  not null,
    cd_education_status       char(20) not null,
    cd_purchase_estimate      integer  not null,
    cd_credit_rating          char(10) not null,
    cd_dep_count              integer  not null,
    cd_dep_employed_count     integer  not null,
    cd_dep_college_count      integer  not null
)
duplicate key (cd_demo_sk)
use tpcds_100g_parquet_lz4;

CREATE EXTERNAL TABLE IF NOT EXISTS customer_address
 (
  ca_address_sk int
  ,ca_address_id varchar(16)
  ,ca_street_number varchar(10)
  ,ca_street_name varchar(60)
  ,ca_street_type varchar(15)
  ,ca_suite_number varchar(10)
  ,ca_city varchar(60)
  ,ca_county varchar(30)
  ,ca_state varchar(2)
  ,ca_zip varchar(10)
  ,ca_country varchar(20)
  ,ca_gmt_offset decimal(5,2)
  ,ca_location_type varchar(20)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/customer_address/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS customer_demographics
 (
  cd_demo_sk int
 ,cd_gender varchar(1)
 ,cd_marital_status varchar(1)
 ,cd_education_status varchar(20)
 ,cd_purchase_estimate   int
 ,cd_credit_rating varchar(10)
 ,cd_dep_count int
 ,cd_dep_employed_count int
 ,cd_dep_college_count int
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/customer_demographics/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS date_dim
 (
  d_date_sk int
  ,d_date_id varchar(16)
  ,d_date date
  ,d_month_seq int
  ,d_week_seq int
  ,d_quarter_seq int
  ,d_year int
  ,d_dow int
  ,d_moy int
  ,d_dom int
  ,d_qoy int
  ,d_fy_year int
  ,d_fy_quarter_seq int
  ,d_fy_week_seq int
  ,d_day_name varchar(9)
  ,d_quarter_name varchar(6)
  ,d_holiday varchar(1)
  ,d_weekend varchar(1)
  ,d_following_holiday varchar(1)
  ,d_first_dom int
  ,d_last_dom int
  ,d_same_day_ly int
  ,d_same_day_lq int
  ,d_current_day varchar(1)
  ,d_current_week varchar(1)
  ,d_current_month varchar(1)
  ,d_current_quarter varchar(1)
  ,d_current_year varchar(1)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/date_dim/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS warehouse
 (
  w_warehouse_sk int
  ,w_warehouse_id varchar(16)
  ,w_warehouse_name varchar(20)
  ,w_warehouse_sq_ft int
  ,w_street_number varchar(10)
  ,w_street_name varchar(60)
  ,w_street_type varchar(15)
  ,w_suite_number varchar(10)
  ,w_city varchar(60)
  ,w_county varchar(30)
  ,w_state varchar(2)
  ,w_zip varchar(10)
  ,w_country varchar(20)
  ,w_gmt_offset decimal(5,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/warehouse/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS ship_mode
 (
  sm_ship_mode_sk int
  ,sm_ship_mode_id varchar(16)
  ,sm_type varchar(30)
  ,sm_code varchar(10)
  ,sm_carrier varchar(20)
  ,sm_contract varchar(20)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/ship_mode/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS time_dim
 (
  t_time_sk int
  ,t_time_id varchar(16)
  ,t_time int
  ,t_hour int
  ,t_minute int
  ,t_second int
  ,t_am_pm varchar(2)
  ,t_shift varchar(20)
  ,t_sub_shift varchar(20)
  ,t_meal_time varchar(20)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/time_dim/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS reason
 (
  r_reason_sk int
  ,r_reason_id varchar(16)
  ,r_reason_desc varchar(100)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/reason/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS income_band
 (
  ib_income_band_sk integer ,
  ib_lower_bound integer ,
  ib_upper_bound integer
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/income_band/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS item
 (
  i_item_sk int
  ,i_item_id varchar(16)
  ,i_rec_start_date date
  ,i_rec_end_date date
  ,i_item_desc varchar(200)
  ,i_current_price decimal(7,2)
  ,i_wholesale_cost decimal(7,2)
  ,i_brand_id int
  ,i_brand varchar(50)
  ,i_class_id int
  ,i_class varchar(50)
  ,i_category_id int
  ,i_category varchar(50)
  ,i_manufact_id int
  ,i_manufact varchar(50)
  ,i_size varchar(20)
  ,i_formulation varchar(20)
  ,i_color varchar(20)
  ,i_units varchar(10)
  ,i_container varchar(10)
  ,i_manager_id int
  ,i_product_name varchar(50)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/item/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS store
 (
  s_store_sk int
  ,s_store_id varchar(16)
  ,s_rec_start_date date
  ,s_rec_end_date date
  ,s_closed_date_sk int
  ,s_store_name varchar(50)
  ,s_number_employees int
  ,s_floor_space int
  ,s_hours varchar(20)
  ,s_manager varchar(40)
  ,s_market_id int
  ,s_geography_class varchar(100)
  ,s_market_desc varchar(100)
  ,s_market_manager varchar(40)
  ,s_division_id int
  ,s_division_name varchar(50)
  ,s_company_id int
  ,s_company_name varchar(50)
  ,s_street_number varchar(10)
  ,s_street_name varchar(60)
  ,s_street_type varchar(15)
  ,s_suite_number varchar(10)
  ,s_city varchar(60)
  ,s_county varchar(30)
  ,s_state varchar(2)
  ,s_zip varchar(10)
  ,s_country varchar(20)
  ,s_gmt_offset decimal(5,2)
  ,s_tax_precentage decimal(5,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/store/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS call_center
 (
  cc_call_center_sk int
  ,cc_call_center_id varchar(16)
  ,cc_rec_start_date date
  ,cc_rec_end_date date
  ,cc_closed_date_sk int
  ,cc_open_date_sk int
  ,cc_name varchar(50)
  ,cc_class varchar(50)
  ,cc_employees int
  ,cc_sq_ft int
  ,cc_hours varchar(20)
  ,cc_manager varchar(40)
  ,cc_mkt_id int
  ,cc_mkt_class varchar(50)
  ,cc_mkt_desc varchar(100)
  ,cc_market_manager varchar(40)
  ,cc_division int
  ,cc_division_name varchar(50)
  ,cc_company int
  ,cc_company_name varchar(50)
  ,cc_street_number varchar(10)
  ,cc_street_name varchar(60)
  ,cc_street_type varchar(15)
  ,cc_suite_number varchar(10)
  ,cc_city varchar(60)
  ,cc_county varchar(30)
  ,cc_state varchar(2)
  ,cc_zip varchar(10)
  ,cc_country varchar(20)
  ,cc_gmt_offset decimal(5,2)
  ,cc_tax_percentage decimal(5,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/call_center/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS customer
 (
  c_customer_sk int
  ,c_customer_id varchar(16)
  ,c_current_cdemo_sk int
  ,c_current_hdemo_sk int
  ,c_current_addr_sk int
  ,c_first_shipto_date_sk    int
  ,c_first_sales_date_sk     int
  ,c_salutation varchar(10)
  ,c_first_name varchar(20)
  ,c_last_name varchar(30)
  ,c_preferred_cust_flag     varchar(1)
  ,c_birth_day int
  ,c_birth_month int
  ,c_birth_year int
  ,c_birth_country varchar(20)
  ,c_login varchar(13)
  ,c_email_address varchar(50)
  ,c_last_review_date varchar(10)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/customer/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS web_site
 (
  web_site_sk int
  ,web_site_id varchar(16)
  ,web_rec_start_date date
  ,web_rec_end_date date
  ,web_name varchar(50)
  ,web_open_date_sk int
  ,web_close_date_sk int
  ,web_class varchar(50)
  ,web_manager varchar(40)
  ,web_mkt_id int
  ,web_mkt_class varchar(50)
  ,web_mkt_desc varchar(100)
  ,web_market_manager varchar(40)
  ,web_company_id int
  ,web_company_name varchar(50)
  ,web_street_number varchar(10)
  ,web_street_name varchar(60)
  ,web_street_type varchar(15)
  ,web_suite_number varchar(10)
  ,web_city varchar(60)
  ,web_county varchar(30)
  ,web_state varchar(2)
  ,web_zip varchar(10)
  ,web_country varchar(20)
  ,web_gmt_offset decimal(5,2)
  ,web_tax_percentage decimal(5,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/web_site/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS store_returns
 (
  sr_item_sk int
  ,sr_ticket_number int
  ,sr_returned_date_sk int
  ,sr_return_time_sk int
  ,sr_customer_sk int
  ,sr_cdemo_sk int
  ,sr_hdemo_sk int
  ,sr_addr_sk int
  ,sr_store_sk int
  ,sr_reason_sk int
  ,sr_return_quantity int
  ,sr_return_amt decimal(7,2)
  ,sr_return_tax decimal(7,2)
  ,sr_return_amt_inc_tax decimal(7,2)
  ,sr_fee decimal(7,2)
  ,sr_return_ship_cost decimal(7,2)
  ,sr_refunded_cash decimal(7,2)
  ,sr_reversed_charge decimal(7,2)
  ,sr_store_credit decimal(7,2)
  ,sr_net_loss decimal(7,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/store_returns/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS household_demographics
 (
  hd_demo_sk int
  ,hd_income_band_sk int
  ,hd_buy_potential varchar(15)
  ,hd_dep_count int
  ,hd_vehicle_count int
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/household_demographics/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS web_page
 (
  wp_web_page_sk int
  ,wp_web_page_id varchar(16)
  ,wp_rec_start_date date
  ,wp_rec_end_date date
  ,wp_creation_date_sk int
  ,wp_access_date_sk int
  ,wp_autogen_flag varchar(1)
  ,wp_customer_sk int
  ,wp_url varchar(100)
  ,wp_type varchar(50)
  ,wp_char_count int
  ,wp_link_count int
  ,wp_image_count int
  ,wp_max_ad_count int
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/web_page/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS promotion
 (
  p_promo_sk int
  ,p_promo_id varchar(16)
  ,p_start_date_sk int
  ,p_end_date_sk int
  ,p_item_sk int
  ,p_cost decimal(15,2)
  ,p_response_target int
  ,p_promo_name varchar(50)
  ,p_channel_dmail varchar(1)
  ,p_channel_email varchar(1)
  ,p_channel_catalog varchar(1)
  ,p_channel_tv varchar(1)
  ,p_channel_radio varchar(1)
  ,p_channel_press varchar(1)
  ,p_channel_event varchar(1)
  ,p_channel_demo varchar(1)
  ,p_channel_details varchar(100)
  ,p_purpose varchar(15)
  ,p_discount_active varchar(1)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/promotion/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS catalog_page
 (
  cp_catalog_page_sk int
  ,cp_catalog_page_id varchar(16)
  ,cp_start_date_sk int
  ,cp_end_date_sk int
  ,cp_department varchar(50)
  ,cp_catalog_number int
  ,cp_catalog_page_number    int
  ,cp_description varchar(100)
  ,cp_type varchar(100)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/catalog_page/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS inventory
 (
  inv_date_sk integer ,
  inv_item_sk integer ,
  inv_warehouse_sk integer ,
  inv_quantity_on_hand integer
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/inventory/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS catalog_returns
 (
  cr_item_sk int
  ,cr_order_number int
  ,cr_returned_date_sk int
  ,cr_returned_time_sk int
  ,cr_refunded_customer_sk int
  ,cr_refunded_cdemo_sk  int
  ,cr_refunded_hdemo_sk  int
  ,cr_refunded_addr_sk int
  ,cr_returning_customer_sk int
  ,cr_returning_cdemo_sk     int
  ,cr_returning_hdemo_sk     int
  ,cr_returning_addr_sk  int
  ,cr_call_center_sk int
  ,cr_catalog_page_sk int
  ,cr_ship_mode_sk int
  ,cr_warehouse_sk int
  ,cr_reason_sk int
  ,cr_return_quantity int
  ,cr_return_amount decimal(7,2)
  ,cr_return_tax decimal(7,2)
  ,cr_return_amt_inc_tax     decimal(7,2)
  ,cr_fee decimal(7,2)
  ,cr_return_ship_cost  decimal(7,2)
  ,cr_refunded_cash decimal(7,2)
  ,cr_reversed_charge decimal(7,2)
  ,cr_store_credit decimal(7,2)
  ,cr_net_loss decimal(7,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/catalog_returns/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS web_returns
 (
  wr_item_sk int
  ,wr_order_number int
  ,wr_returned_date_sk int
  ,wr_returned_time_sk int
  ,wr_refunded_customer_sk   int
  ,wr_refunded_cdemo_sk  int
  ,wr_refunded_hdemo_sk int
  ,wr_refunded_addr_sk int
  ,wr_returning_customer_sk  int
  ,wr_returning_cdemo_sk     int
  ,wr_returning_hdemo_sk     int
  ,wr_returning_addr_sk  int
  ,wr_web_page_sk int
  ,wr_reason_sk int
  ,wr_return_quantity int
  ,wr_return_amt decimal(7,2)
  ,wr_return_tax decimal(7,2)
  ,wr_return_amt_inc_tax     decimal(7,2)
  ,wr_fee decimal(7,2)
  ,wr_return_ship_cost decimal(7,2)
  ,wr_refunded_cash decimal(7,2)
  ,wr_reversed_charge decimal(7,2)
  ,wr_account_credit decimal(7,2)
  ,wr_net_loss decimal(7,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/web_returns/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS web_sales
 (
  ws_item_sk int
  ,ws_order_number int
  ,ws_sold_date_sk int
  ,ws_sold_time_sk int
  ,ws_ship_date_sk int
  ,ws_bill_customer_sk int
  ,ws_bill_cdemo_sk int
  ,ws_bill_hdemo_sk int
  ,ws_bill_addr_sk int
  ,ws_ship_customer_sk int
  ,ws_ship_cdemo_sk int
  ,ws_ship_hdemo_sk int
  ,ws_ship_addr_sk int
  ,ws_web_page_sk int
  ,ws_web_site_sk int
  ,ws_ship_mode_sk int
  ,ws_warehouse_sk int
  ,ws_promo_sk int
  ,ws_quantity int
  ,ws_wholesale_cost decimal(7,2)
  ,ws_list_price decimal(7,2)
  ,ws_sales_price decimal(7,2)
  ,ws_ext_discount_amt decimal(7,2)
  ,ws_ext_sales_price decimal(7,2)
  ,ws_ext_wholesale_cost decimal(7,2)
  ,ws_ext_list_price decimal(7,2)
  ,ws_ext_tax decimal(7,2)
  ,ws_coupon_amt decimal(7,2)
  ,ws_ext_ship_cost decimal(7,2)
  ,ws_net_paid decimal(7,2)
  ,ws_net_paid_inc_tax  decimal(7,2)
  ,ws_net_paid_inc_ship  decimal(7,2)
  ,ws_net_paid_inc_ship_tax  decimal(7,2)
  ,ws_net_profit decimal(7,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/web_sales/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS catalog_sales
 (
  cs_item_sk int
  ,cs_order_number int
  ,cs_sold_date_sk int
  ,cs_sold_time_sk int
  ,cs_ship_date_sk int
  ,cs_bill_customer_sk int
  ,cs_bill_cdemo_sk int
  ,cs_bill_hdemo_sk int
  ,cs_bill_addr_sk int
  ,cs_ship_customer_sk int
  ,cs_ship_cdemo_sk int
  ,cs_ship_hdemo_sk int
  ,cs_ship_addr_sk int
  ,cs_call_center_sk int
  ,cs_catalog_page_sk int
  ,cs_ship_mode_sk int
  ,cs_warehouse_sk int
  ,cs_promo_sk int
  ,cs_quantity int
  ,cs_wholesale_cost decimal(7,2)
  ,cs_list_price decimal(7,2)
  ,cs_sales_price decimal(7,2)
  ,cs_ext_discount_amt decimal(7,2)
  ,cs_ext_sales_price decimal(7,2)
  ,cs_ext_wholesale_cost decimal(7,2)
  ,cs_ext_list_price decimal(7,2)
  ,cs_ext_tax decimal(7,2)
  ,cs_coupon_amt decimal(7,2)
  ,cs_ext_ship_cost decimal(7,2)
  ,cs_net_paid decimal(7,2)
  ,cs_net_paid_inc_tax  decimal(7,2)
  ,cs_net_paid_inc_ship  decimal(7,2)
  ,cs_net_paid_inc_ship_tax decimal(7,2)
  ,cs_net_profit decimal(7,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/catalog_sales/'
tblproperties("parquet.compression"="Lz4");

CREATE EXTERNAL TABLE IF NOT EXISTS store_sales
 (
  ss_item_sk int
  ,ss_ticket_number int
  ,ss_sold_date_sk int
  ,ss_sold_time_sk int
  ,ss_customer_sk int
  ,ss_cdemo_sk int
  ,ss_hdemo_sk int
  ,ss_addr_sk int
  ,ss_store_sk int
  ,ss_promo_sk int
  ,ss_quantity int
  ,ss_wholesale_cost decimal(7,2)
  ,ss_list_price decimal(7,2)
  ,ss_sales_price decimal(7,2)
  ,ss_ext_discount_amt decimal(7,2)
  ,ss_ext_sales_price decimal(7,2)
  ,ss_ext_wholesale_cost decimal(7,2)
  ,ss_ext_list_price decimal(7,2)
  ,ss_ext_tax decimal(7,2)
  ,ss_coupon_amt decimal(7,2)
  ,ss_net_paid decimal(7,2)
  ,ss_net_paid_inc_tax decimal(7,2)
  ,ss_net_profit decimal(7,2)
 )
stored as PARQUET
LOCATION '/user/tmp/parquet/store_sales/'
tblproperties("parquet.compression"="Lz4");
```

### 5.4 Create Hive external tables（CSV）

```SQL
use tpcds_100g_csv;

CREATE EXTERNAL TABLE IF NOT EXISTS customer_address
 (
  ca_address_sk int
  ,ca_address_id varchar(16)
  ,ca_street_number varchar(10)
  ,ca_street_name varchar(60)
  ,ca_street_type varchar(15)
  ,ca_suite_number varchar(10)
  ,ca_city varchar(60)
  ,ca_county varchar(30)
  ,ca_state varchar(2)
  ,ca_zip varchar(10)
  ,ca_country varchar(20)
  ,ca_gmt_offset decimal(5,2)
  ,ca_location_type varchar(20)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/customer_address/';

CREATE EXTERNAL TABLE IF NOT EXISTS customer_demographics
 (
  cd_demo_sk int
 ,cd_gender varchar(1)
 ,cd_marital_status varchar(1)
 ,cd_education_status varchar(20)
 ,cd_purchase_estimate   int
 ,cd_credit_rating varchar(10)
 ,cd_dep_count int
 ,cd_dep_employed_count int
 ,cd_dep_college_count int
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/customer_demographics/';

CREATE EXTERNAL TABLE IF NOT EXISTS date_dim
 (
  d_date_sk int
  ,d_date_id varchar(16)
  ,d_date date
  ,d_month_seq int
  ,d_week_seq int
  ,d_quarter_seq int
  ,d_year int
  ,d_dow int
  ,d_moy int
  ,d_dom int
  ,d_qoy int
  ,d_fy_year int
  ,d_fy_quarter_seq int
  ,d_fy_week_seq int
  ,d_day_name varchar(9)
  ,d_quarter_name varchar(6)
  ,d_holiday varchar(1)
  ,d_weekend varchar(1)
  ,d_following_holiday varchar(1)
  ,d_first_dom int
  ,d_last_dom int
  ,d_same_day_ly int
  ,d_same_day_lq int
  ,d_current_day varchar(1)
  ,d_current_week varchar(1)
  ,d_current_month varchar(1)
  ,d_current_quarter varchar(1)
  ,d_current_year varchar(1)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/date_dim/';

CREATE EXTERNAL TABLE IF NOT EXISTS warehouse
 (
  w_warehouse_sk int
  ,w_warehouse_id varchar(16)
  ,w_warehouse_name varchar(20)
  ,w_warehouse_sq_ft int
  ,w_street_number varchar(10)
  ,w_street_name varchar(60)
  ,w_street_type varchar(15)
  ,w_suite_number varchar(10)
  ,w_city varchar(60)
  ,w_county varchar(30)
  ,w_state varchar(2)
  ,w_zip varchar(10)
  ,w_country varchar(20)
  ,w_gmt_offset decimal(5,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/warehouse/';

CREATE EXTERNAL TABLE IF NOT EXISTS ship_mode
 (
  sm_ship_mode_sk int
  ,sm_ship_mode_id varchar(16)
  ,sm_type varchar(30)
  ,sm_code varchar(10)
  ,sm_carrier varchar(20)
  ,sm_contract varchar(20)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/ship_mode/';

CREATE EXTERNAL TABLE IF NOT EXISTS time_dim
 (
  t_time_sk int
  ,t_time_id varchar(16)
  ,t_time int
  ,t_hour int
  ,t_minute int
  ,t_second int
  ,t_am_pm varchar(2)
  ,t_shift varchar(20)
  ,t_sub_shift varchar(20)
  ,t_meal_time varchar(20)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/time_dim/';

CREATE EXTERNAL TABLE IF NOT EXISTS reason
 (
  r_reason_sk int
  ,r_reason_id varchar(16)
  ,r_reason_desc varchar(100)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/reason/';

CREATE EXTERNAL TABLE IF NOT EXISTS income_band
 (
  ib_income_band_sk integer ,
  ib_lower_bound integer ,
  ib_upper_bound integer
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/income_band/';

CREATE EXTERNAL TABLE IF NOT EXISTS item
 (
  i_item_sk int
  ,i_item_id varchar(16)
  ,i_rec_start_date date
  ,i_rec_end_date date
  ,i_item_desc varchar(200)
  ,i_current_price decimal(7,2)
  ,i_wholesale_cost decimal(7,2)
  ,i_brand_id int
  ,i_brand varchar(50)
  ,i_class_id int
  ,i_class varchar(50)
  ,i_category_id int
  ,i_category varchar(50)
  ,i_manufact_id int
  ,i_manufact varchar(50)
  ,i_size varchar(20)
  ,i_formulation varchar(20)
  ,i_color varchar(20)
  ,i_units varchar(10)
  ,i_container varchar(10)
  ,i_manager_id int
  ,i_product_name varchar(50)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/item/';

CREATE EXTERNAL TABLE IF NOT EXISTS store
 (
  s_store_sk int
  ,s_store_id varchar(16)
  ,s_rec_start_date date
  ,s_rec_end_date date
  ,s_closed_date_sk int
  ,s_store_name varchar(50)
  ,s_number_employees int
  ,s_floor_space int
  ,s_hours varchar(20)
  ,s_manager varchar(40)
  ,s_market_id int
  ,s_geography_class varchar(100)
  ,s_market_desc varchar(100)
  ,s_market_manager varchar(40)
  ,s_division_id int
  ,s_division_name varchar(50)
  ,s_company_id int
  ,s_company_name varchar(50)
  ,s_street_number varchar(10)
  ,s_street_name varchar(60)
  ,s_street_type varchar(15)
  ,s_suite_number varchar(10)
  ,s_city varchar(60)
  ,s_county varchar(30)
  ,s_state varchar(2)
  ,s_zip varchar(10)
  ,s_country varchar(20)
  ,s_gmt_offset decimal(5,2)
  ,s_tax_precentage decimal(5,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/store/';

CREATE EXTERNAL TABLE IF NOT EXISTS call_center
 (
  cc_call_center_sk int
  ,cc_call_center_id varchar(16)
  ,cc_rec_start_date date
  ,cc_rec_end_date date
  ,cc_closed_date_sk int
  ,cc_open_date_sk int
  ,cc_name varchar(50)
  ,cc_class varchar(50)
  ,cc_employees int
  ,cc_sq_ft int
  ,cc_hours varchar(20)
  ,cc_manager varchar(40)
  ,cc_mkt_id int
  ,cc_mkt_class varchar(50)
  ,cc_mkt_desc varchar(100)
  ,cc_market_manager varchar(40)
  ,cc_division int
  ,cc_division_name varchar(50)
  ,cc_company int
  ,cc_company_name varchar(50)
  ,cc_street_number varchar(10)
  ,cc_street_name varchar(60)
  ,cc_street_type varchar(15)
  ,cc_suite_number varchar(10)
  ,cc_city varchar(60)
  ,cc_county varchar(30)
  ,cc_state varchar(2)
  ,cc_zip varchar(10)
  ,cc_country varchar(20)
  ,cc_gmt_offset decimal(5,2)
  ,cc_tax_percentage decimal(5,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/call_center/';

CREATE EXTERNAL TABLE IF NOT EXISTS customer
 (
  c_customer_sk int
  ,c_customer_id varchar(16)
  ,c_current_cdemo_sk int
  ,c_current_hdemo_sk int
  ,c_current_addr_sk int
  ,c_first_shipto_date_sk    int
  ,c_first_sales_date_sk     int
  ,c_salutation varchar(10)
  ,c_first_name varchar(20)
  ,c_last_name varchar(30)
  ,c_preferred_cust_flag     varchar(1)
  ,c_birth_day int
  ,c_birth_month int
  ,c_birth_year int
  ,c_birth_country varchar(20)
  ,c_login varchar(13)
  ,c_email_address varchar(50)
  ,c_last_review_date varchar(10)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/customer/';

CREATE EXTERNAL TABLE IF NOT EXISTS web_site
 (
  web_site_sk int
  ,web_site_id varchar(16)
  ,web_rec_start_date date
  ,web_rec_end_date date
  ,web_name varchar(50)
  ,web_open_date_sk int
  ,web_close_date_sk int
  ,web_class varchar(50)
  ,web_manager varchar(40)
  ,web_mkt_id int
  ,web_mkt_class varchar(50)
  ,web_mkt_desc varchar(100)
  ,web_market_manager varchar(40)
  ,web_company_id int
  ,web_company_name varchar(50)
  ,web_street_number varchar(10)
  ,web_street_name varchar(60)
  ,web_street_type varchar(15)
  ,web_suite_number varchar(10)
  ,web_city varchar(60)
  ,web_county varchar(30)
  ,web_state varchar(2)
  ,web_zip varchar(10)
  ,web_country varchar(20)
  ,web_gmt_offset decimal(5,2)
  ,web_tax_percentage decimal(5,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/web_site/';

CREATE EXTERNAL TABLE IF NOT EXISTS store_returns
 (
  sr_item_sk int
  ,sr_ticket_number int
  ,sr_returned_date_sk int
  ,sr_return_time_sk int
  ,sr_customer_sk int
  ,sr_cdemo_sk int
  ,sr_hdemo_sk int
  ,sr_addr_sk int
  ,sr_store_sk int
  ,sr_reason_sk int
  ,sr_return_quantity int
  ,sr_return_amt decimal(7,2)
  ,sr_return_tax decimal(7,2)
  ,sr_return_amt_inc_tax decimal(7,2)
  ,sr_fee decimal(7,2)
  ,sr_return_ship_cost decimal(7,2)
  ,sr_refunded_cash decimal(7,2)
  ,sr_reversed_charge decimal(7,2)
  ,sr_store_credit decimal(7,2)
  ,sr_net_loss decimal(7,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/store_returns/';

CREATE EXTERNAL TABLE IF NOT EXISTS household_demographics
 (
  hd_demo_sk int
  ,hd_income_band_sk int
  ,hd_buy_potential varchar(15)
  ,hd_dep_count int
  ,hd_vehicle_count int
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/household_demographics/';

CREATE EXTERNAL TABLE IF NOT EXISTS web_page
 (
  wp_web_page_sk int
  ,wp_web_page_id varchar(16)
  ,wp_rec_start_date date
  ,wp_rec_end_date date
  ,wp_creation_date_sk int
  ,wp_access_date_sk int
  ,wp_autogen_flag varchar(1)
  ,wp_customer_sk int
  ,wp_url varchar(100)
  ,wp_type varchar(50)
  ,wp_char_count int
  ,wp_link_count int
  ,wp_image_count int
  ,wp_max_ad_count int
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/web_page/';

CREATE EXTERNAL TABLE IF NOT EXISTS promotion
 (
  p_promo_sk int
  ,p_promo_id varchar(16)
  ,p_start_date_sk int
  ,p_end_date_sk int
  ,p_item_sk int
  ,p_cost decimal(15,2)
  ,p_response_target int
  ,p_promo_name varchar(50)
  ,p_channel_dmail varchar(1)
  ,p_channel_email varchar(1)
  ,p_channel_catalog varchar(1)
  ,p_channel_tv varchar(1)
  ,p_channel_radio varchar(1)
  ,p_channel_press varchar(1)
  ,p_channel_event varchar(1)
  ,p_channel_demo varchar(1)
  ,p_channel_details varchar(100)
  ,p_purpose varchar(15)
  ,p_discount_active varchar(1)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/promotion/';

CREATE EXTERNAL TABLE IF NOT EXISTS catalog_page
 (
  cp_catalog_page_sk int
  ,cp_catalog_page_id varchar(16)
  ,cp_start_date_sk int
  ,cp_end_date_sk int
  ,cp_department varchar(50)
  ,cp_catalog_number int
  ,cp_catalog_page_number    int
  ,cp_description varchar(100)
  ,cp_type varchar(100)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/catalog_page/';

CREATE EXTERNAL TABLE IF NOT EXISTS inventory
 (
  inv_date_sk integer ,
  inv_item_sk integer ,
  inv_warehouse_sk integer ,
  inv_quantity_on_hand integer
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/inventory/';

CREATE EXTERNAL TABLE IF NOT EXISTS catalog_returns
 (
  cr_item_sk int
  ,cr_order_number int
  ,cr_returned_date_sk int
  ,cr_returned_time_sk int
  ,cr_refunded_customer_sk int
  ,cr_refunded_cdemo_sk  int
  ,cr_refunded_hdemo_sk  int
  ,cr_refunded_addr_sk int
  ,cr_returning_customer_sk int
  ,cr_returning_cdemo_sk     int
  ,cr_returning_hdemo_sk     int
  ,cr_returning_addr_sk  int
  ,cr_call_center_sk int
  ,cr_catalog_page_sk int
  ,cr_ship_mode_sk int
  ,cr_warehouse_sk int
  ,cr_reason_sk int
  ,cr_return_quantity int
  ,cr_return_amount decimal(7,2)
  ,cr_return_tax decimal(7,2)
  ,cr_return_amt_inc_tax     decimal(7,2)
  ,cr_fee decimal(7,2)
  ,cr_return_ship_cost  decimal(7,2)
  ,cr_refunded_cash decimal(7,2)
  ,cr_reversed_charge decimal(7,2)
  ,cr_store_credit decimal(7,2)
  ,cr_net_loss decimal(7,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/catalog_returns/';

CREATE EXTERNAL TABLE IF NOT EXISTS web_returns
 (
  wr_item_sk int
  ,wr_order_number int
  ,wr_returned_date_sk int
  ,wr_returned_time_sk int
  ,wr_refunded_customer_sk   int
  ,wr_refunded_cdemo_sk  int
  ,wr_refunded_hdemo_sk int
  ,wr_refunded_addr_sk int
  ,wr_returning_customer_sk  int
  ,wr_returning_cdemo_sk     int
  ,wr_returning_hdemo_sk     int
  ,wr_returning_addr_sk  int
  ,wr_web_page_sk int
  ,wr_reason_sk int
  ,wr_return_quantity int
  ,wr_return_amt decimal(7,2)
  ,wr_return_tax decimal(7,2)
  ,wr_return_amt_inc_tax     decimal(7,2)
  ,wr_fee decimal(7,2)
  ,wr_return_ship_cost decimal(7,2)
  ,wr_refunded_cash decimal(7,2)
  ,wr_reversed_charge decimal(7,2)
  ,wr_account_credit decimal(7,2)
  ,wr_net_loss decimal(7,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/web_returns/';

CREATE EXTERNAL TABLE IF NOT EXISTS web_sales
 (
  ws_item_sk int
  ,ws_order_number int
  ,ws_sold_date_sk int
  ,ws_sold_time_sk int
  ,ws_ship_date_sk int
  ,ws_bill_customer_sk int
  ,ws_bill_cdemo_sk int
  ,ws_bill_hdemo_sk int
  ,ws_bill_addr_sk int
  ,ws_ship_customer_sk int
  ,ws_ship_cdemo_sk int
  ,ws_ship_hdemo_sk int
  ,ws_ship_addr_sk int
  ,ws_web_page_sk int
  ,ws_web_site_sk int
  ,ws_ship_mode_sk int
  ,ws_warehouse_sk int
  ,ws_promo_sk int
  ,ws_quantity int
  ,ws_wholesale_cost decimal(7,2)
  ,ws_list_price decimal(7,2)
  ,ws_sales_price decimal(7,2)
  ,ws_ext_discount_amt decimal(7,2)
  ,ws_ext_sales_price decimal(7,2)
  ,ws_ext_wholesale_cost decimal(7,2)
  ,ws_ext_list_price decimal(7,2)
  ,ws_ext_tax decimal(7,2)
  ,ws_coupon_amt decimal(7,2)
  ,ws_ext_ship_cost decimal(7,2)
  ,ws_net_paid decimal(7,2)
  ,ws_net_paid_inc_tax  decimal(7,2)
  ,ws_net_paid_inc_ship  decimal(7,2)
  ,ws_net_paid_inc_ship_tax  decimal(7,2)
  ,ws_net_profit decimal(7,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/web_sales/';

CREATE EXTERNAL TABLE IF NOT EXISTS catalog_sales
 (
  cs_item_sk int
  ,cs_order_number int
  ,cs_sold_date_sk int
  ,cs_sold_time_sk int
  ,cs_ship_date_sk int
  ,cs_bill_customer_sk int
  ,cs_bill_cdemo_sk int
  ,cs_bill_hdemo_sk int
  ,cs_bill_addr_sk int
  ,cs_ship_customer_sk int
  ,cs_ship_cdemo_sk int
  ,cs_ship_hdemo_sk int
  ,cs_ship_addr_sk int
  ,cs_call_center_sk int
  ,cs_catalog_page_sk int
  ,cs_ship_mode_sk int
  ,cs_warehouse_sk int
  ,cs_promo_sk int
  ,cs_quantity int
  ,cs_wholesale_cost decimal(7,2)
  ,cs_list_price decimal(7,2)
  ,cs_sales_price decimal(7,2)
  ,cs_ext_discount_amt decimal(7,2)
  ,cs_ext_sales_price decimal(7,2)
  ,cs_ext_wholesale_cost decimal(7,2)
  ,cs_ext_list_price decimal(7,2)
  ,cs_ext_tax decimal(7,2)
  ,cs_coupon_amt decimal(7,2)
  ,cs_ext_ship_cost decimal(7,2)
  ,cs_net_paid decimal(7,2)
  ,cs_net_paid_inc_tax  decimal(7,2)
  ,cs_net_paid_inc_ship  decimal(7,2)
  ,cs_net_paid_inc_ship_tax decimal(7,2)
  ,cs_net_profit decimal(7,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/catalog_sales/';
  ,cs_ext_ship_cost decimal(7,2)
  ,cs_net_paid decimal(7,2)
  ,cs_net_paid_inc_tax  decimal(7,2)
  ,cs_net_paid_inc_ship  decimal(7,2)
  ,cs_net_paid_inc_ship_tax decimal(7,2)
  ,cs_net_profit decimal(7,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/catalog_sales/';

CREATE EXTERNAL TABLE IF NOT EXISTS store_sales
 (
  ss_item_sk int
  ,ss_ticket_number int
  ,ss_sold_date_sk int
  ,ss_sold_time_sk int
  ,ss_customer_sk int
  ,ss_cdemo_sk int
  ,ss_hdemo_sk int
  ,ss_addr_sk int
  ,ss_store_sk int
  ,ss_promo_sk int
  ,ss_quantity int
  ,ss_wholesale_cost decimal(7,2)
  ,ss_list_price decimal(7,2)
  ,ss_sales_price decimal(7,2)
  ,ss_ext_discount_amt decimal(7,2)
  ,ss_ext_sales_price decimal(7,2)
  ,ss_ext_wholesale_cost decimal(7,2)
  ,ss_ext_list_price decimal(7,2)
  ,ss_ext_tax decimal(7,2)
  ,ss_coupon_amt decimal(7,2)
  ,ss_net_paid decimal(7,2)
  ,ss_net_paid_inc_tax decimal(7,2)
  ,ss_net_profit decimal(7,2)
 )
row format delimited fields terminated by '|'
LOCATION '/user/tmp/csv/store_sales/';
