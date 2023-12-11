---
displayed_sidebar: "Japanese"
---

# SQLBLACKLISTの追加

## 説明

SQL blacklistに正規表現を追加して、特定のSQLパターンを禁止します。SQL Blacklist機能が有効になっていると、StarRocksは実行されるすべてのSQLステートメントをSQL blacklist内の正規表現と比較します。SQL blacklist内のいずれかの正規表現に一致するSQLは実行されず、エラーが返されます。これにより、特定のSQLがクラスタのクラッシュや予期しない動作を引き起こすのを防ぎます。

SQL Blacklistについて詳しくは、[SQL Blacklistの管理](../../../administration/Blacklist.md)を参照してください。

> **注意**
>
> ADMIN権限を持つユーザーのみがSQL blacklistにSQL正規表現を追加できます。

## 構文

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## パラメータ

`sql_reg_expr`：特定のSQLパターンを指定するために使用される正規表現。SQLステートメント内の特殊文字と正規表現内の特殊文字を区別するために、SQLステートメント内の特殊文字の前にエスケープ文字 `\` を使用する必要があります。例えば、`(`、`)`、`+`などの特殊文字に対してエスケープ文字を使用する必要があります。ただし、`(`と`)`はSQLステートメントでよく使用されるため、StarRocksはSQLステートメント内の`(`と`)`を直接識別できます。`(`と`)`に対してはエスケープ文字は必要ありません。

## 例

例1：`count(\*)`をSQL blacklistに追加。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

例2：`count(distinct )`をSQL blacklistに追加。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

例3：`order by limit x, y, 1 <= x <=7, 5 <=y <=7`をSQL blacklistに追加。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

例4：複雑なSQL正規表現をSQL blacklistに追加。この例は、SQLステートメントで`*`と`-`のエスケープ文字の使用方法を示しています。

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