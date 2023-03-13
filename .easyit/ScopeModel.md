# ScopeModel

## ScopeModel 是什么
todo
ScopeModel 一共分为三种:
* [FrameworkModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L74)
* [ApplicationModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L100)
* [ModuleModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L54)

FrameworkModel 包含多个 ApplicationModel, ApplicationModel 包含多个 ModuleModel.    

## ScopeModel 的构造
在 [获取 DubboBootstrap 实例](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L48) 的时候, 会构造 FrameworkModel 实例.进而构造 ApplicationModel 和 ModuleModel. 
FrameworkModel 构造流程: 
1. 调用父类构造函数
2. 将自身注册到 allInstances 中. 
3. [进行初始化动作](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L84). 
    1. [构造 extensionDirector](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L100) : 详见 [ExtensionDirector](ExtensionDirector.md)
    2. [为 extensionDirector 注册 PostProcessor](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L101)
    3. [构造 beanFactory](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L102): 详见 [ScopeBeanFactory](ScopeBeanFactory.md)
    4. [保存保存当前的 ScopeModel.class 的 classLoader](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L105): 这里缓存了一份加载 Dubbo 的 classloader 
4. 初始化 TypeDefinitionBuilder todo
5. 构造 serviceRepository todo 
6. 初始化相关 ExtensionLoader 
7. 构造 internalApplicationModel

为什么这里是 internal 的? 什么时候会出现 非 internal 的? 两者又有什么区别?
 
> 查看了下代码, internalApplicationModel 并没有地方用到, 不确定有什么用 </br> 
  区别: internal 的[不会被加到 pubApplicationModels 中](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L247)

AplicationModel 的构造
todo 

ModuleModel 的构造
todo


 
## 问题
1 [globalLock](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L76) 和 [instLock](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L77) 分别是保护什么资源的? 
