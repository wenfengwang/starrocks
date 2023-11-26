---
displayed_sidebar: "Japanese"
---

# ルール

## 必須は使用しない

プロジェクトが進行するにつれて、任意のフィールドがオプションになる可能性があります。しかし、必須と定義されている場合は削除できません。

したがって、`required`は使用しないでください。

## 順序を変更しない

後方互換性を保つために、フィールドの順序は変更してはいけません。

# 命名規則

## ファイル名

メッセージの名前はすべて小文字で、単語間にアンダースコアを使用します。
ファイルの拡張子は `.proto` で終わる必要があります。

```
my_message.proto            // 良い例
mymessage.proto             // 悪い例
my_message.pb                // 悪い例
```

## メッセージ名

メッセージ名は大文字で始まり、新しい単語ごとに大文字を使用し、アンダースコアは使用せず、`PB` を接尾辞として付けます: MyMessagePB

```protobuf
message MyMessagePB       // 良い例
message MyMessage         // 悪い例
message My_Message_PB     // 悪い例
message myMessagePB       // 悪い例
```

## フィールド名

フィールドの名前はすべて小文字で、単語間にアンダースコアを使用します。

```
optional int64 my_field = 3;        // 良い例
optional int64 myField = 3;         // 悪い例
```
