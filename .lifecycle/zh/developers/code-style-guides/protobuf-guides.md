---
displayed_sidebar: English
---

# 规则

## 绝不使用“必需”

随着项目的发展，任何字段都有可能变为可选。但如果一个字段被定义为“必需”，那么它就不能被移除。

因此，“必需”一词不应被使用。

## 永远不要更改顺序

为了保持向后兼容性，字段的顺序编号不应该被更改。

# 命名规则

## 文件名

消息的名称应全部使用小写字母，单词之间用下划线连接。文件名应以.proto为后缀。

```
my_message.proto            // Good
mymessage.proto             // Bad
my_message.pb                // Bad
```

## 消息名称

消息的名称应以大写字母开头，每个新单词的首字母大写，不使用下划线，并且以PB作为后缀，例如：MyMessagePB

```protobuf
message MyMessagePB       // Good
message MyMessage         // Bad
message My_Message_PB     // Bad
message myMessagePB       // Bad
```

## 字段名称

字段的名称应全部使用小写字母，单词之间用下划线连接。

```
optional int64 my_field = 3;        // Good
optional int64 myField = 3;         // Bad
```
