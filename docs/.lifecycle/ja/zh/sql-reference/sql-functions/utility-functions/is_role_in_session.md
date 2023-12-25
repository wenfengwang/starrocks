---
displayed_sidebar: Chinese
---

# is_role_in_session

## 機能

指定されたロール（ネストされたロールを含む）が現在のセッションでアクティブかどうかをチェックするために使用されます。

この関数はバージョン3.1.4からサポートされています。

## 文法

```Haskell
BOOLEAN is_role_in_session(VARCHAR role_name);
```

## パラメータ説明

`role_name`: チェックするロールで、ネストされたロールも可能です。サポートされるデータタイプはVARCHARです。

## 戻り値の説明

戻り値のデータタイプはBOOLEANです。`0`は非アクティブを意味し、`1`はロールがアクティブであることを意味します。

## 例

1. ロールとユーザーを作成します。

   ```sql
   -- 3つのロールを作成します。
   create role r1;
   create role r2;
   create role r3;

   -- ユーザーu1を作成します。
   create user u1;

   -- ロールを段階的にr1に渡し、その後ユーザーu1にr1を付与します。これにより、ユーザーはr1、r2、r3の3つのロールを持つことになります。
   grant r3 to role r2;
   grant r2 to role r1;
   grant r1 to user u1;

   -- ユーザーu1に切り替え、u1の権限で操作を実行します。
   execute as u1 with no revert;
   ```

2. ロール`r1`がアクティブかどうかをチェックします。結果は非アクティブであることを示しています。

   ```plaintext
   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        0 |
   +--------------------------+
   ```

3. [SET ROLE](../../sql-statements/account-management/SET_ROLE.md) コマンドを使用してロール`r1`をアクティブにし、再度チェックします。結果はロール`r1`がアクティブになっており、そのネストされたロール`r2`と`r3`も同時にアクティブになっていることを示しています。

   ```sql
   set role "r1";

   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        1 |
   +--------------------------+

   select is_role_in_session("r2");
   +--------------------------+
   | is_role_in_session('r2') |
   +--------------------------+
   |                        1 |
   +--------------------------+

   select is_role_in_session("r3");
   +--------------------------+
   | is_role_in_session('r3') |
   +--------------------------+
   |                        1 |
   +--------------------------+
   ```
