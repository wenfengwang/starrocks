```yaml
---
displayed_sidebar: "Japanese"
---

# プロパティの設定

## 説明

### 構文

```SQL
SET PROPERTY [FOR 'ユーザー'] 'キー' = '値' [, 'キー' = '値']
```

ユーザーのリソースなどを含むユーザー属性を設定します。ここでのユーザープロパティは、ユーザー識別属性ではなく、ユーザー自身の属性を指します。つまり、'jack'@'%' と 'jack'@'192.%' の2つのユーザーが CREATE USER ステートメントを通じて作成された場合、SET PROPERTY ステートメントは 'jack'@'%' や 'jack'@'192.%' ではなく、ユーザー jack にのみ使用できます。

キー:

スーパーユーザー権限:

```plain text
max_user_connections: 接続の最大数
resource.cpu_share: CPU リソースの割り当て
```

一般ユーザー権限:

```plain text
quota.normal: 通常レベルでのリソース割り当て
quota.high: 高レベルでのリソース割り当て
quota.low: 低レベルでのリソース割り当て
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
```