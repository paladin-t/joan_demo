The Joan NLP system.

Copyright (C) 2016 Wang Renxin. All rights reserved.

[Email](mailto:hellotony521@qq.com)

# 一、概述

本软件是自然语言处理（Natural Language Processing, NLP）的一个演示程序，实现了一套自然语言问答系统。

# 二、使用方法

## 1. 运行方法

本 demo 暂时只提供 Win32 平台二进制文件。

运行需 VC++ 2015 运行时库，如需安装请执行 [/redist/vc_redist.x86.exe](https://github.com/paladin-t/joan_demo/raw/master/redist/vc_redist.x86.exe)。

请执行以下批处理文件启动 demo：

~~~~~~~~~~
/bin/x86/start.cmd
~~~~~~~~~~

注意操作系统可能会弹出对话框询问是否允许网络访问，请选择允许。demo 程序将启动一个 web 服务器：

![](https://github.com/paladin-t/joan_demo/blob/master/img/1.png)

服务器已配置成在启动后打开一个本机的浏览器，并转向服务页面。如果你看到如下页面说明服务器工作正常。你也可以使用能连接到 demo 服务器的终端浏览器手动输入其地址来访问，服务器地址会在如下页面中显示。注意 demo 页面使用 HTML5 技术，请确保你的浏览器不是过早的版本。

在对应浏览器前台窗口的文件输入框输入对话文本并点击 "Send" 按钮来和服务器交互：

![](https://github.com/paladin-t/joan_demo/blob/master/img/2.png)

## 2. 例句

因 demo 的目的是小规模演示，故只提供受限功能、脚本、DSL、数据等。可尝试输入以下例句：

惯用短语

~~~~~~~~~~
你好！
~~~~~~~~~~

事实、定义型问题

~~~~~~~~~~
你是谁？
你是干什么的？
苹果是什么？
~~~~~~~~~~

共指消歧义

~~~~~~~~~~
你的开发者是谁？
他的工作是什么，他喜欢吃什么？
~~~~~~~~~~

列表型问题

~~~~~~~~~~
苹果手机由什么组成的？
~~~~~~~~~~

关系、观点问题

~~~~~~~~~~
Siri是谁？
你和Siri谁更聪明？
~~~~~~~~~~

祈使句

~~~~~~~~~~
你给我唱个歌吧。
给我讲个笑话呢？
~~~~~~~~~~

# 三、技术说明

## 1. 知识建模

作为**知识表达**和**机器推理**数据库，系统将内置提供诸如涵盖但不限于以下领域特定和知识数据：

~~~~~~~~~~
地理，动物，植物，饮食，职业，日常用品
服饰，交通工具，起居，天气
时间概念，空间概念，经济概念
~~~~~~~~~~

知识采用一种对象化表示的数据库结构，例如：

![](https://github.com/paladin-t/joan_demo/blob/master/img/3.png)

又如：

![](https://github.com/paladin-t/joan_demo/blob/master/img/4.png)

## 2. 分词，句法分析

完成**线性文本**到**结构化**的转换。

分词和句法分析流程如下：

![](https://github.com/paladin-t/joan_demo/blob/master/img/5.png)

系统会先将文本字符串切分出可行的分词列表。并做必要处理。如量词、时间识别，动词时态识别。然后会交给句法分析器匹配句法，并对每条匹配评分。最终得到按评分排序的所有可能句法列表作为下一个阶段的输入数据。

句法分析中采用类似编译原理中自顶向下的分析方法，汉语作为分析语，其语序相对固定，其文法声明书写为内部 DSL：

![](https://github.com/paladin-t/joan_demo/blob/master/img/6.png)

如上面代码第二个 `PatternRule`：

~~~~~~~~~~
attributive.front()
	.add(WP_ADJECTIVE).add(WP_AUXILIARY).add(WP_COMPOSITED)
	.finish();
~~~~~~~~~~

表示产生式：

~~~~~~~~~~
Attributive := adj. Auxiliary; <Composited;>
~~~~~~~~~~

句法分析器会利用句法声明对带属性的线性节点做匹配：

![](https://github.com/paladin-t/joan_demo/blob/master/img/7.png)

## 3. 编译生成 AST

完成**结构化**到**波兰表达式**的转换。

此 NLP 系统内建一套“谓词-论元”规则，并以与自然语言无关的波兰表达式组织。

例如如下代码展示某一个按编译模板生成句式的流程：

![](https://github.com/paladin-t/joan_demo/blob/master/img/8.png)

其中读入结构化节点的过程分别由可选（`optional`）和必选（`forward`）两种操作抽取。在遇到第一个不匹配的节点时当前模式的匹配过程就会马上结束并返回上层，以进行和其他规则的匹配。

## 4. AST 语义理解，知识查询

处理**波兰表达式**，“思考”并生成输出**波兰表达式**。

作为思考前的必要处理，会将谓词、论元等结构进一步结构特化：

![](https://github.com/paladin-t/joan_demo/blob/master/img/9.png)

例如谓词的特化过程如下：

![](https://github.com/paladin-t/joan_demo/blob/master/img/10.png)

在理解过程中会生成查询 `Query` 从知识数据库中查询必要知识，并做加工处理。而每条 `Query` 可以对应一组查询元语的组合。这里仅举例一部分实践中用到的查询元语：

* `SELECT`：后跟一串对象关系路径参数，以此在数据库中查询对象细节
* `COMPARE`：用于多个对象的比较查询和关系查询，如“iOS 和 Android 哪个好用”，“成都到北京有多远”
* `DECLARE`：对于用户输入的陈述句用声明（declare）处理，实践中会维护一个和会话用户对应的数据库，此元语对一条声明做简单的真假判断，并存储到会话数据库中
* `IMPETRATE`：供祈使句处理使用，一些情况中此元语仅充当其他元语的前置语气，并无实际逻辑作用，如“告诉我月亮离地球有多远”，实际是对地月距离的 `COMPARE`；另一些则以祈使句为主体，如“给我唱首歌”，“讲个笑话”
* `EXTRACT`：在元语搭配中，`EXTRACT` 以其他元语查询的结果为中间数据，进一步抽取出细节作为结果

如果将问答系统用于实用助手类应用场景，则可扩展查询元语，类似编程语言中的多态性，扩展出更多功能，例如天气查询、航班、股市、休闲消费的查询等。

在思考最前面会做必要的共指消歧等处理，然后以 `queryXXX` 的形式查询知识数据库，并用 `emitXXX` 做出应答。一个典型的“固化”在代码里的查询应答流程如下：

![](https://github.com/paladin-t/joan_demo/blob/master/img/11.png)

除此之外，此 demo 亦支持以外部 DSL 数据的方式“描述”出语义理解和知识查询应答的流程，如：

![](https://github.com/paladin-t/joan_demo/blob/master/img/12.png)

即先 `match` （匹配），再 `query` （查询），最后 `say` （应答）。一个对形如祈使句“给我讲个笑话”的语义理解外部 DSL 描述如下：

![](https://github.com/paladin-t/joan_demo/blob/master/img/13.png)

另外，`Query` 元语也使用数据驱动的方式描述。

![](https://github.com/paladin-t/joan_demo/blob/master/img/14.png)

系统还支持其他语义理解描述方式，如 Lua 脚本等。暂略。

## 5. 应答 AST 生成

完成**波兰表达式**到**结构化**再到**线性文本**的序列化过程。

首先，在**波兰表达式**到**结构化**的环节会按句式模板生成结构化 AST，注意此过程相当于编译生成 AST 的逆过程，此过程的输出和自然语言相关，例如一个典型的汉语“主-谓-宾”句式转换过程如下：

![](https://github.com/paladin-t/joan_demo/blob/master/img/15.png)

需要注意自然语言无关的 AST 表示方法是一种便于计算机处理的“精简”表示，在生成最终输出时需对具体自然语言的惯用法做一些处理。如量词惯用法：

![](https://github.com/paladin-t/joan_demo/blob/master/img/16.png)

表示“鸟”及其子类的“后肢”需要被输出为“腿”，并且单指用“条”，双指用“双”；“前肢”被输出为“翅膀”，单指用“只”，双指用“对”。

以及介词惯用法：

![](https://github.com/paladin-t/joan_demo/blob/master/img/17.png)

表示 `<from>` 词义在搭配“组成”和“决定”时需要输出成“由 ... 组成”，“由 ... 决定”，而不输出成“从”。
