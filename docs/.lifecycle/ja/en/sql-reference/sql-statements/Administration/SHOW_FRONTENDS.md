---
displayed_sidebar: English
---

# SHOW FRONTENDS

## 説明

このステートメントは、FEノードを表示するために使用されます。

:::tip

この操作は、SYSTEMレベルのOPERATE権限を持つユーザー、または`cluster_admin`ロールを持つユーザーのみが実行できます。

:::

## 構文

```sql
SHOW FRONTENDS
```

注記：

1. NameはBDBJE内のFEノードの名前を表します。
2. Joinがtrueである場合、このノードは既にクラスターに参加していることを意味します。しかし、ノードがクラスター内にまだ存在しているとは限らないため、ノードが欠落している可能性があります。
3. Aliveはノードが生存しているかどうかを示します。
4. ReplayedJournalIdは、ノードが現在リプレイしているメタデータログの最大IDを表します。
5. LastHeartbeatは最新のハートビートを示します。
6. IsHelperはノードがBDBJEのヘルパーノードであるかどうかを示します。
7. ErrMsgはハートビートに失敗した際のエラーメッセージを表示するために使用されます。
