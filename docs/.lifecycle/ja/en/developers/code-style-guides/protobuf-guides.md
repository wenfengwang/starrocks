---
displayed_sidebar: English
---

# ルール

## requiredは使用しない

プロジェクトの進行に伴い、どのフィールドもオプショナルになる可能性があります。しかし、一度requiredとして定義されると、それを取り除くことはできません。

従って、`required`は使用しないでください。

## 順序を変更しない

後方互換性を保つために、フィールドの順序は変更してはいけません(SHOULD NOT)。

# 命名規則

## ファイル名

メッセージの名前は全て小文字で、単語間にはアンダースコアを使用します。
ファイルは `.proto` で終わるべきです。

```
my_message.proto            // Good
mymessage.proto             // Bad
my_message.pb               // Bad
```

## メッセージ名

メッセージ名は大文字で始まり、新しい単語のそれぞれに大文字を使用し、アンダースコアは含まず、`PB` を接尾辞として使用します: MyMessagePB

```protobuf
message MyMessagePB       // Good
message MyMessage         // Bad
message My_Message_PB     // Bad
message myMessagePB       // Bad
```

## フィールド名

フィールド名は全て小文字で、単語間にはアンダースコアを使用します。

```
optional int64 my_field = 3;        // Good
optional int64 myField = 3;         // Bad
```