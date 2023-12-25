---
displayed_sidebar: English
---

# データ復旧

StarRocksは、誤って削除されたデータベース/テーブル/パーティションのデータ復旧をサポートしています。`drop table`または`drop database`を実行した後、StarRocksはデータをすぐに物理的に削除するのではなく、一定期間（デフォルトでは1日）Trashに保管します。管理者は`RECOVER`コマンドを使用して、誤って削除されたデータを復旧することができます。

## 関連コマンド

構文：

~~~sql
-- 1) データベースの復旧
RECOVER DATABASE db_name;
-- 2) テーブルの復旧
RECOVER TABLE [db_name.]table_name;
-- 3) パーティションの復旧
RECOVER PARTITION partition_name FROM [db_name.]table_name;
~~~

## 注意事項

1. この操作は削除されたメタデータのみを復元できます。デフォルトの保持期間は1日で、`fe.conf`の`catalog_trash_expire_second`パラメータで設定可能です。
2. メタデータが削除された後に、同じ名前とタイプの新しいメタデータが作成された場合、以前に削除されたメタデータは復元できません。

## 例

1. `example_db`という名前のデータベースを復旧する

    ~~~sql
    RECOVER DATABASE example_db;
    ~~~

2. `example_tbl`という名前のテーブルを復旧する

    ~~~sql
    RECOVER TABLE example_db.example_tbl;
    ~~~

3. `example_tbl`テーブルの`p1`というパーティションを復旧する

    ~~~sql
    RECOVER PARTITION p1 FROM example_tbl;
    ~~~
