---
displayed_sidebar: "Japanese"
---

# プロパティの設定

## 説明

### 構文

```SQL
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

ユーザーの属性を設定します。ここでのユーザープロパティは、ユーザーの属性を指します。つまり、CREATE USER ステートメントで作成された2つのユーザー、'jack'@'%' と 'jack'@'192.%' に対しては、SET PROPERTY ステートメントは 'jack' ユーザーにのみ使用できます。

key:

スーパーユーザー権限:

```plain text
max_user_connections: 最大接続数
resource.cpu_share: CPU リソースの割り当て
```

一般ユーザー権限:

```plain text
quota.normal: 通常レベルのリソース割り当て
quota.high: 高レベルのリソース割り当て
quota.low: 低レベルのリソース割り当て
```

## 例

1. ユーザー jack の最大接続数を 1000 に変更する

    ```SQL
    SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';
    ```

2. ユーザー jack の cpu_share を 1000 に変更する

    ```SQL
    SET PROPERTY FOR 'jack' 'resource.cpu_share' = '1000';
    ```

3. ユーザー jack の通常レベルのウェイトを変更する

    ```SQL
    SET PROPERTY FOR 'jack' 'quota.normal' = '400';
    ```
