---
displayed_sidebar: "Japanese"
---

# データの復旧

StarRocksは、誤って削除されたデータベース/テーブル/パーティションのデータの復旧をサポートしています。`drop table`または`drop database`の後、StarRocksはデータを直ちに物理的に削除せず、一定期間（デフォルトでは1日）Trashに保持します。管理者は`RECOVER`コマンドを使用して、誤って削除されたデータを復元することができます。

## 関連コマンド

構文:

~~~sql
-- 1) データベースの復元
RECOVER DATABASE データベース名;
-- 2) テーブルの復元
RECOVER TABLE [データベース名.]テーブル名;
-- 3) パーティションの復元
RECOVER PARTITION パーティション名 FROM [データベース名.]テーブル名;
~~~

## 注意事項

1. この操作は削除されたメタ情報のみを復元することができます。デフォルトの時間は1日であり、`fe.conf`の`catalog_trash_expire_second`パラメータで設定することができます。
2. メタ情報が削除された後に同じ名前とタイプの新しいメタ情報が作成された場合、以前に削除されたメタ情報は復元できません。

## 例

1. `example_db`という名前のデータベースを復元する場合

    ~~~sql
    RECOVER DATABASE example_db;
    ~~~ 2.

2. `example_tbl`という名前のテーブルを復元する場合

    ~~~sql
    RECOVER TABLE example_db.example_tbl;
    ~~~ 3.

3. `example_tbl`の中の`p1`という名前のパーティションを復元する場合

    ~~~sql
    RECOVER PARTITION p1 FROM example_tbl;
    ~~~
