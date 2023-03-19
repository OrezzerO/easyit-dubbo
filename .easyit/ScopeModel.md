# ScopeModel

## ScopeModel 是什么
todo: ScopeModel 是什么

ScopeModel 一共分为三种:

* [FrameworkModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L74)
* [ApplicationModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L100)
* [ModuleModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L54)

FrameworkModel 包含多个 ApplicationModel, ApplicationModel 包含多个 ModuleModel.

## ScopeModel 的构造

### FrameworkModel 的构造

FrameworkModel 并不是按照单例来设计的, 但在常规使用中, 可以将 FrameworkModel 看成一个单例.
通过 FrameworkModel.defaultModel() 方法 [获取 FrameworkModel 实例](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L171)
获取.

在 [获取 DubboBootstrap 实例](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L48)
的时候,
会调用 [FrameworkModel.defaultModel() 方法](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L171)
构造 FrameworkModel 实例.

#### FrameworkModel 构造流程:

1. [调用父类构造函数](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L75)
2. [将自身注册到 allInstances 中.](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L80)
3. [进行初始化动作](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L84).
    1. [构造 extensionDirector](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L100) :
       详见 [ExtensionDirector](ExtensionDirector.md)
    2. [为 extensionDirector 注册 PostProcessor](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L101)
    3. [构造 beanFactory](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L102):
       详见 [ScopeBeanFactory](ScopeBeanFactory.md)
    4. [保存保存当前的 ScopeModel.class 的 classLoader](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ScopeModel.java#L105):
       这里缓存了一份加载 Dubbo 的 classloader
4. [初始化 TypeDefinitionBuilder](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L86):
   详见 [TypeBuilder](../dubbo-common/src/main/java/org/apache/dubbo/metadata/definition/builder/TypeBuilder.java#L31)
5. [构造 serviceRepository](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L88): todo
6. [初始化 ScopeModelInitializer](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L90):
   todo
7. [构造 internalApplicationModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L96):
   暂时没有看到 internalApplicationModel 有什么用, 是否考虑删掉?

为什么这里是 internal 的? 什么时候会出现 非 internal 的? 两者又有什么区别?

> 查看了下代码, internalApplicationModel 并没有地方用到, 不确定有什么用 </br>
> 区别: internal 的[不会被加到 pubApplicationModels 中](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L247)

### ApplicationModel 的构造

FrameworkModel 内部会构造一个 internalApplicationModel 对象. 而在构造 DubboBootstrap 的时候,
会[通过 FrameworkModel.defaultModel().newApplication() 构造一个新的 ApplicationModel](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L123).

#### ApplicationModel 构造流程

1. [调用父类构造函数](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L101)
2. [设置 frameworkModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L104)
3. [初始化](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L109): 调用 ScopeModel 中的 initialize 方法, 和 FrameworkModel 逻辑一致.
4. [构造 internalModule](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L111)
5. [构造 serviceRepository](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L112)
6. [初始化 ApplicationInitListener](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L114)
7. [初始化 ApplicationExt](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L136)
8. [初始化 ScopeModelInitializer](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ApplicationModel.java#L122)

### ModuleModel 的构造
ApplicationModel 内部会构造一个 internalModule. 在[创建 builtinServices 的时候会被用到](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ServiceRepository.java#L45).
在启动 Dubbo 的时候, 启动业务 Model 之前, 会先 [prepareInternalModule](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L635).

在[设置 service 信息](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L52)的时候, 会[懒加载一个 DefaultModel](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L537)

#### ModuleModel 的构造流程
1. [调用父类构造函数](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L55)
2. [设置 applicationModel](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L58)
3. [初始化](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L64)
4. [构造 serviceRepository ](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L66)
5. [初始化 ModuleExt](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L68)
6. [初始化 ScopeModelInitializer](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L70)
7. [触发 applicationDeployer 的 ModuleChanged 事件](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/ModuleModel.java#L80)


## 问题

1. [globalLock](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L76)
   和 [instLock](../dubbo-common/src/main/java/org/apache/dubbo/rpc/model/FrameworkModel.java#L77) 分别是保护什么资源的?
2. FrameworkModel 并不是单例, 但是 TypeDefinitionBuilder 却是单例.
   初始化时 [initBuilders](../dubbo-common/src/main/java/org/apache/dubbo/metadata/definition/TypeDefinitionBuilder.java#L42)
   却依赖一个非单例, 这里需要进一步看看.
    1. 是不是 initBuilders 内部,又获取了一次单例? 需要看看 ExtensionLoader 的实现.
    2. 如果说多次初始化, 会发生什么? 是否加个标记位, 只初始化一次? 
