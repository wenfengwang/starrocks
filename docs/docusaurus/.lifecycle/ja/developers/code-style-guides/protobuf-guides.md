---
displayed_sidebar: "Japanese"
---

# ルール

## 必要ないことは絶対に使用しない

プロジェクトが進行するにつれて、任意のフィールドが必須でなくなる可能性があります。しかし、それが必須として定義されている場合、削除できません。

したがって、 `required` は使用すべきではありません。

## 順序を変更しないこと

後方互換性を保つために、フィールドの順序を変更しないようにします。

# 命名規則

## ファイル名

メッセージの名前はすべて小文字で、単語間にはアンダースコアを使用します。
ファイルは `.proto` で終わらなければなりません。

```
my_message.proto            // 良い
mymessage.proto             // 悪い
my_message.pb                // 悪い
```

## メッセージ名

メッセージ名は大文字で始め、単語ごとに大文字で記述し、アンダースコアは使用せず、`PB` で終わるようにします: MyMessagePB

```protobuf
message MyMessagePB       // 良い
message MyMessage         // 悪い
message My_Message_PB     // 悪い
message myMessagePB       // 悪い
```

## フィールド名

メッセージの名前はすべて小文字で、単語間にはアンダースコアを使用します。

```
optional int64 my_field = 3;        // 良い
optional int64 myField = 3;         // 悪い
```