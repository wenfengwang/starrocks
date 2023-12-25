---
displayed_sidebar: English
---

# ADD SQLBLACKLIST

## 説明

SQLブラックリストに正規表現を追加して、特定のSQLパターンを禁止します。SQLブラックリスト機能が有効の場合、StarRocksは実行される全てのSQL文をブラックリスト内のSQL正規表現と比較します。ブラックリスト内の正規表現に一致するSQLは実行されず、エラーが返されます。これにより、特定のSQLがクラスターのクラッシュや予期せぬ挙動を引き起こすのを防ぎます。

SQLブラックリストの詳細については、[SQLブラックリストの管理](../../../administration/Blacklist.md)を参照してください。

:::tip

この操作にはSYSTEMレベルのBLACKLIST権限が必要です。[GRANT](../account-management/GRANT.md)の指示に従ってこの権限を付与することができます。

:::

## 構文

```SQL
ADD SQLBLACKLIST "<sql_reg_expr>"
```

## パラメータ

`sql_reg_expr`: 特定のSQLパターンを指定するために使用される正規表現です。SQL文の特殊文字と正規表現の特殊文字を区別するために、SQL文の特殊文字（`(`、`)`、`+`など）の前にエスケープ文字`\`を使用する必要があります。ただし、`(`と`)`はSQL文で頻繁に使用されるため、StarRocksはSQL文内の`(`と`)`を直接識別できます。`(`と`)`にはエスケープ文字を使用する必要はありません。

## 例

例1: `count(\*)`をSQLブラックリストに追加します。

```Plain
mysql> ADD SQLBLACKLIST "select count(\\*) from .+";
```

例2: `count(distinct )`をSQLブラックリストに追加します。

```Plain
mysql> ADD SQLBLACKLIST "select count(distinct .+) from .+";
```

例3: `order by limit x, y, 1 <= x <=7, 5 <=y <=7`をSQLブラックリストに追加します。

```Plain
mysql> ADD SQLBLACKLIST "select id_int from test_all_type_select1 
    order by id_int 
    limit [1-7], [5-7]";
```

例4: 複雑なSQL正規表現をSQLブラックリストに追加します。この例は、SQL文内の`*`と`-`にエスケープ文字を使用する方法を示しています。

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
