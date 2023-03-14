# ExtensionDirector

## 什么是 ExtensionDirector
Dubbo
将扩展分为不同的 [ExtensionScope](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionScope.java#L28):
* FRAMEWORK
* APPLICATION
* MODULE
* SELF

TODO: 扩展这块找个地缝整体说一下



## ExtensionDirector 的构造
[ExtensionDirector](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionDirector.java#L34)
会在构造 [ScopeModel](ScopeModel.md)
的时候[被创建](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L100).


## 获取 ExtensionLoader 流程
1. [判断自身是否已销毁](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionDirector.java#L68)
2. [从缓存中获取](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionDirector.java#L81)
3. [获取 Extension 接口的 Scope 参数](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionDirector.java#L83)
4. [如果 Scope 参数是 SELF ,直接创建](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionDirector.java#L92)
5. 如果上一步没创建成功, [交给父 ExtensionDirector 创建](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionDirector.java#L98)
6. [如果上一步没创建成功 判断 Scope 符合之后创建](../dubbo-common/src/main/java/org/apache/dubbo/common/extension/ExtensionDirector.java#L104)









