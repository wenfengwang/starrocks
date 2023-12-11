---
displayed_sidebar: "Japanese"
---

# プロパティを設定する

## 説明

### 構文

```SQL
SET PROPERTY [FOR 'ユーザー'] 'キー' = '値' [, 'キー' = '値']
```

ユーザーに割り当てられるリソースなどを含むユーザー属性を設定します。ここでのユーザープロパティとは、`user_identity` の属性ではなく、ユーザーの属性を指します。つまり、CREATE USER ステートメントを使用して 'jack'@'%' および 'jack'@'192.%' の 2 つのユーザーを作成した場合、SET PROPERTY ステートメントは 'jack' ユーザーにのみ使用でき、'jack'@'%' や 'jack'@'192.%' には使用できません。

キー:

スーパーユーザー権限:

```plain text
max_user_connections: 最大接続数
resource.cpu_share: CPU リソース割り当て
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

3. ユーザー jack の通常レベルの重みを変更する

    ```SQL
    SET PROPERTY FOR 'jack' 'quota.normal' = '400';
    ```