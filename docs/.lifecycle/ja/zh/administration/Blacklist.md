---
displayed_sidebar: Chinese
---

# ブラックリストの管理

本文では、SQL ブラックリスト（SQL Blacklist）の管理方法について説明します。

StarRocks では、特定のシナリオで特定のタイプの SQL を禁止するために、SQL ブラックリストを維持することができます。これにより、そのような SQL がクラスタのダウンや予期せぬ挙動を引き起こすのを防ぎます。

> **注意**
>
> BLACKLIST 権限を持つユーザーのみがブラックリスト機能を使用できます。

## ブラックリスト機能の有効化

以下のコマンドでブラックリスト機能を有効にします。

```sql
ADMIN SET FRONTEND CONFIG ("enable_sql_blacklist" = "true");
```

## ブラックリストの追加

以下のコマンドで SQL ブラックリストを追加します。

```sql
ADD SQLBLACKLIST "sql";
```

**"sql"**：特定の SQL の正規表現です。SQL には `(`、`)`、`*`、`.` などの文字が含まれており、これらは正規表現の意味と混同する可能性があるため、ブラックリストを設定する際にはエスケープ文字を使用して区別する必要があります。`(` と `)` は SQL で頻繁に使用されるため、内部で処理を行い、設定時にエスケープする必要はありませんが、他の特殊文字にはエスケープ文字 "\\" をプレフィックスとして使用する必要があります。

例:

* `count(\*)` を禁止する。

    ```sql
    ADD SQLBLACKLIST "select count\\(\\*\\) from .+";
    ```

* `count(distinct )` を禁止する。

    ```sql
    ADD SQLBLACKLIST "select count\\(distinct .+\\) from .+";
    ```

* `order by limit x, y` で `1 <= x <=7`, `5 <=y <=7` を禁止する。

    ```sql
    ADD SQLBLACKLIST "select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]";
    ```

* 複雑な SQL を禁止する（`*` と `-` のエスケープ方法を示す例）。

    ```sql
    ADD SQLBLACKLIST "select id_int \\* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable";
    ```

## ブラックリストの表示

```sql
SHOW SQLBLACKLIST;
```

例：

```plain text
mysql> show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
| 3     | select id_int from test_all_type_select1 order by id_int limit [1-7], [5-7]                                                                                                                                                                                                                            |
| 4     | select count\(distinct .+\) from .+                                                                                                                                                                                                                                                                    |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

結果には `Index` と `Forbidden SQL` が含まれます。`Index` は禁止された SQL のブラックリスト番号で、`Forbidden SQL` は禁止された SQL を表示し、すべての SQL の意味を持つ文字はエスケープ処理されています。

## ブラックリストの削除

以下のコマンドで SQL ブラックリストを削除できます。

```sql
DELETE SQLBLACKLIST index_no;
```

`index_no`：禁止された SQL のブラックリスト番号で、`SHOW SQLBLACKLIST;` コマンドで確認できます。複数の `index_no` は `,` で区切ります。

例：

```plain text
mysql> delete sqlblacklist  3, 4;

show sqlblacklist;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Index | Forbidden SQL                                                                                                                                                                                                                                                                                          |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1     | select count\(\*\) from .+                                                                                                                                                                                                                                                                             |
| 2     | select id_int \* 4, id_tinyint, id_varchar from test_all_type_nullable except select id_int, id_tinyint, id_varchar from test_basic except select \(id_int \* 9 \- 8\) \/ 2, id_tinyint, id_varchar from test_all_type_nullable2 except select id_int, id_tinyint, id_varchar from test_basic_nullable |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```
