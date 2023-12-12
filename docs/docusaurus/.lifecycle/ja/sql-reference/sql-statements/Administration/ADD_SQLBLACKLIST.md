---
displayed_sidebar: "Japanese"
---

# SQLBLACKLISTの追加

## 説明

SQLのブラックリストに正規表現を追加して、特定のSQLパターンを禁止します。SQLブラックリスト機能が有効になっているとき、StarRocksは実行されるすべてのSQLステートメントをブラックリスト内のSQL正規表現と比較します。そして、ブラックリスト内の正規表現にマッチするSQLは実行されず、エラーが返されます。これにより、特定のSQLがクラスタのクラッシュや予期しない動作を引き起こすことを防ぎます。

SQLブラックリストについて詳しくは、[SQLブラックリストの管理](../../../administration/Blacklist.md)を参照してください。

> **注意**
>
> ADMIN権限を持つユーザーのみがSQL正規表現をSQLブラックリストに追加できます。

## 構文

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## パラメータ

`sql_reg_expr`: 特定のSQLパターンを指定するために使用される正規表現。SQLステートメント内の特殊文字と正規表現内の特殊文字を区別するために、SQLステートメント内の特殊文字の前にエスケープ文字 `\` を使用する必要があります。例えば、`(`、`)`、`+`などの特殊文字に対してはエスケープ文字を使用する必要があります。ただし `(` および `)` はSQLステートメントでよく使用されるため、StarRocksはSQLステートメント内の `(` および `)` を直接識別できます。したがって、`(` および `)` に対してはエスケープ文字を使用する必要はありません。

## 例

例1: `count(\*)`をSQLブラックリストに追加する。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

例2: `count(distinct )`をSQLブラックリストに追加する。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

例3: `order by limit x, y, 1 <= x <=7, 5 <=y <=7`をSQLブラックリストに追加する。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

例4: 複雑なSQL正規表現をSQLブラックリストに追加する。この例は、SQLステートメント内の `*` と `-` にエスケープ文字を使用する方法を示しています。

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