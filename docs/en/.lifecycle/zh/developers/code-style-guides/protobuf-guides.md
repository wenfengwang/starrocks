---
displayed_sidebar: English
---

# 规则

## 绝不使用 required

随着项目的发展，任何字段都有可能变成可选的。但如果一个字段被定义为 required，那么它就不能被移除。

因此，不应该使用 `required`。

## 永远不要改变顺序值

为了保持向后兼容性，字段的顺序值不应该被改变。

# 命名

## 文件名

消息的名称应全部为小写，并且单词之间用下划线分隔。文件应以 `.proto` 结尾。

```
my_message.proto            // Good
mymessage.proto             // Bad
my_message.pb                // Bad
```

## 消息名称

消息名称应以大写字母开头，每个新单词的首字母大写，不使用下划线，并以 `PB` 作为后缀：MyMessagePB

```protobuf
message MyMessagePB       // Good
message MyMessage         // Bad
message My_Message_PB     // Bad
message myMessagePB       // Bad
```

## 字段名称

字段名称应全部为小写，并且单词之间用下划线分隔。

```
optional int64 my_field = 3;        // Good
optional int64 myField = 3;         // Bad
```