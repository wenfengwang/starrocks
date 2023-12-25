---
displayed_sidebar: English
---

# SET PROPERTY

## 説明

### 構文

```SQL
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

ユーザー属性を設定します。これには、ユーザーに割り当てられたリソースなどが含まれます。ここでのユーザープロパティは、user_identityの属性ではなく、ユーザー自身の属性を指します。つまり、CREATE USER文で'jack'@'%'と'jack'@'192.%'という2つのユーザーが作成された場合、SET PROPERTY文はユーザーjackにのみ使用でき、'jack'@'%'や'jack'@'192.%'には使用できません。

キー：

スーパーユーザー権限:

```plain text
max_user_connections: 最大接続数
resource.cpu_share: CPUリソースの割り当て
```

一般ユーザー権限:

```plain text
quota.normal: 通常レベルでのリソース割り当て
quota.high: 高レベルでのリソース割り当て
quota.low: 低レベルでのリソース割り当て
```

## 例

1. ユーザーjackの最大接続数を1000に変更

    ```SQL
    SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';
    ```

2. ユーザーjackのCPUリソース割り当てを1000に変更

    ```SQL
    SET PROPERTY FOR 'jack' 'resource.cpu_share' = '1000';
    ```

3. ユーザーjackの通常レベルのリソース割り当てを400に変更

    ```SQL
    SET PROPERTY FOR 'jack' 'quota.normal' = '400';
    ```
