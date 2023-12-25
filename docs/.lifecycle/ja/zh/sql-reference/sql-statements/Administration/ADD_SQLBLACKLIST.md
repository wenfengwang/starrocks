---
displayed_sidebar: Chinese
---

# ADD SQLBLACKLIST

## 機能

SQL 正規表現を SQL ブラックリストに追加します。SQL ブラックリスト機能が有効になっている場合、StarRocks は実行する必要があるすべての SQL ステートメントをブラックリスト内の SQL 正規表現と比較します。StarRocks は、ブラックリスト内のいずれかの正規表現に一致する SQL を実行せず、エラーを返します。

SQL ブラックリストの詳細については、[SQL ブラックリストの管理](../../../administration/Blacklist.md)を参照してください。

:::tip

この操作には SYSTEM レベルの BLACKLIST 権限が必要です。ユーザーに権限を付与するには [GRANT](../account-management/GRANT.md) を参照してください。

:::

## 文法

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## パラメータ説明

`sql_reg_expr`：SQL ステートメントを指定する正規表現です。SQL ステートメント内の特殊文字と正規表現内の特殊文字を区別するために、特殊文字の前にバックスラッシュ（\）を使用してエスケープする必要があります。例えば `(`、`)`、および `+` です。`(` と `)` は SQL ステートメントで頻繁に使用されるため、StarRocks は SQL ステートメント内の `(` と `)` を直接認識できます。そのため、`(` と `)` にエスケープ文字を追加する必要はありません。

## 例

例 1：`count(\*)` を SQL ブラックリストに追加します。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

例 2：`count(distinct )` を SQL ブラックリストに追加します。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

例 3：`order by limit x, y，1 <= x <=7, 5 <=y <=7` を SQL ブラックリストに追加します。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

例 4：複雑な SQL 正規表現を SQL ブラックリストに追加します。この例では、SQL ステートメント内の `*` と `-` を表すためにエスケープ文字を使用する方法を示しています。

```Plain
mysql> ADD SQLBLACKLIST 
    "select id_int \\* 4, id_tinyint, id_varchar 
        from test_all_type_nullable 
    except select id_int, id_tinyint, id_varchar 
        from test_basic 
    except select (id_int \\* 9 \\- 8) \\/ 2, id_tinyint, id_varchar 
        from test_all_type_nullable2 
    except select id_int, id_tinyint, id_varchar 
        from test_basic_nullable";
```
