---
displayed_sidebar: "Japanese"
---

# データの復旧

StarRocksは間違って削除されたデータベース/テーブル/パーティションのデータ復旧をサポートしています。`drop table`または`drop database`の後、StarRocksはデータを直ちに物理的に削除せず、一定期間（デフォルトでは1日）Trashに保持します。管理者は`RECOVER`コマンドを使用して、間違って削除されたデータを回復することができます。

## 関連コマンド

構文:

~~~sql
-- 1) データベースの復旧
RECOVER DATABASE db_name;
-- 2) テーブルの復旧
RECOVER TABLE [db_name.]table_name;
-- 3) パーティションの復旧
RECOVER PARTITION partition_name FROM [db_name.]table_name;
~~~

## 注意事項

1. この操作は削除されたメタ情報のみを復元できます。デフォルトの時間は1日であり、`fe.conf`の`catalog_trash_expire_second`パラメータで構成することができます。
2. メタ情報が削除された後に同じ名前とタイプの新しいメタ情報が作成された場合、以前に削除されたメタ情報を復元することはできません。

## 例

1. `example_db`という名前のデータベースを回復する

    ~~~sql
    RECOVER DATABASE example_db;
    ~~~

2. `example_tbl`という名前のテーブルを回復する

    ~~~sql
    RECOVER TABLE example_db.example_tbl;
    ~~~

3. `example_tbl`内の`p1`という名前のパーティションを回復する

    ~~~sql
    RECOVER PARTITION p1 FROM example_tbl;
    ~~~