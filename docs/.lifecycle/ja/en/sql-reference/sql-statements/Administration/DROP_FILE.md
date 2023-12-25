---
displayed_sidebar: English
---

# DROP FILE

## 説明

`DROP FILE` ステートメントを実行してファイルを削除できます。このステートメントでファイルを削除すると、フロントエンド（FE）のメモリと Berkeley DB Java Edition（BDBJE）の両方からファイルが削除されます。

:::tip

この操作にはSYSTEMレベルのFILE権限が必要です。[GRANT](../account-management/GRANT.md)の指示に従ってこの権限を付与することができます。

:::

## 構文

```SQL
DROP FILE "file_name" [FROM database]
[properties]
```

## パラメータ

| **パラメータ** | **必須** | **説明**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| file_name     | はい          | 削除するファイルの名前です。                                        |
| database      | いいえ           | ファイルが属するデータベースです。                        |
| properties    | はい          | ファイルのプロパティです。以下の表は`properties`の設定項目を説明しています。 |

**`properties`の設定項目**

| **設定項目** | **必須** | **説明**                       |
| ----------------------- | ------------ | ------------------------------------- |
| catalog                 | はい          | ファイルが属するカテゴリです。 |

## 例

**ca.pem** という名前のファイルを削除します。

```SQL
DROP FILE "ca.pem" properties("catalog" = "kafka");
```
