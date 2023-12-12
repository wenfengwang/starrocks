---
displayed_sidebar: "Japanese"
---

# データの回復

StarRocksは誤って削除されたデータベース/テーブル/パーティションのデータ回復をサポートしています。 `drop table`または`drop database`の後、StarRocksはデータを直ちに物理的に削除せず、ある期間（デフォルトで1日）にわたってゴミ箱に保持します。管理者は`RECOVER`コマンドを使用して誤って削除されたデータを回復できます。

## 関連コマンド

構文:

~~~sql
-- 1) データベースを回復
RECOVER DATABASE db_name;
-- 2) テーブルを復元
RECOVER TABLE [db_name.]table_name;
-- 3) パーティションを回復
RECOVER PARTITION partition_name FROM [db_name.]table_name;
~~~

## 注意事項

1. この操作は削除されたメタ情報のみを復元できます。デフォルトの時間は1日であり、`fe.conf`の`catalog_trash_expire_second`パラメータで設定できます。
2. メタ情報が削除された後に同じ名前とタイプの新しいメタ情報が作成された場合、以前に削除されたメタ情報を回復することはできません。

## 例

1. `example_db`という名前のデータベースを回復します

    ~~~sql
    RECOVER DATABASE example_db;
    ~~~ 2.

2. `example_tbl`という名前のテーブルを回復します

    ~~~sql
    RECOVER TABLE example_db.example_tbl;
    ~~~ 3.

3. `example_tbl`というテーブルで`p1`という名前のパーティションを回復します

    ~~~sql
    RECOVER PARTITION p1 FROM example_tbl;
    ~~~