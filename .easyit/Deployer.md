# Deployer 是什么

[Deployer](../dubbo-common/src/main/java/org/apache/dubbo/common/deploy/Deployer.java#L24) 是 Dubbo 的部署模块. 负责
Dubbo 的初始化工作.
Deployer
共有两个实现类 [DefaultApplicationDeployer](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L93)
和 [DefaultModuleDeployer](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L59).
它们分别负责部署应用和模块.

# Deployer 的生命周期

Deployer 的生命周期有如下几个状态:

1. PENDING
2. STARTING
3. STARTED
4. STOPPING
5. STOPPED
6. FAILED

# Applciation Deployer 的生命周期

## ApplciationDeployer 对象的创建

回顾下启动分析章节, 我们使用 DubboBootstrap
来[启动 Dubbo 服务](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L43)
.有以下三个步骤:

1. [创建 DubboBootstrap 实例](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L48):
   创建过程中,
   我们会[提供一个 ApplicationModel](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L111).
   DubboBootstrap
   会[保存 ApplicationModel 中的 applicationDeployer](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L166).
2. 配置 DubboBootstrap 相关的参数.
3. 调用 DubboBootstrap
   的 [start](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L53)
   方法.

在 start 方法中, 我们实际调用的是 applicationDeployer
的 [start](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L224)
方法.

我们就从 applicationDeployer 的 start 方法, 开始分析 Deployer 状态的变化:

1. 首次进入 start 方法, applicationDeployer 处于 PENDING 状态.
   在 [onStarting](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L573)
   方法会将 applicationDeployer 的状态改为 STARTING .
2. 如果在 start
   方法中,最终有异常抛出, [则将状态改为 FAILED](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L579)
3. 在
   [doStart](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L577)
   的过程中, applicationDeployer 会调用每一个 DefaultModuleDeployer, 在每一个 Module 启动完成之后,会调用
   applicationDeployer
   的 [notifyModuleChanged](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L264).
   通过 [calculateState 方法](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L865) ,
   通过判断 module deployer 的状态, 设置 applicationDeployer 的状态为 STARTED.

## Module Deployer 的生命周期

1. Module Deployer 的创建 (PENDING)
    1. 在给 DubboBootstrap 设置 ServiceConfig 属性的时候,
       会调用 [applicationModel.getDefaultModule()](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L537)
       方法创建一个默认的 Module.
2. 遍历 ApplicationModel 的 ModuleModels
   ,执行 [start 方法](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L640).
3. 执行
   [onModuleStarting](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L152)
   将自身状态改为 STARTING.
4. 执行
   [initialize 初始化](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L155)
    1. [通过 ConfigManager 加载配置](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L113)
    2. 获取 ModuleConfig 配置 Module
       的 [exportAsync](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L81) , [referAsync](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L82)
       和 [background](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L80)
       属性
5. [暴露服务](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L158)
    1. [从 configManager 中获取 ServiceConfig](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L321)
    2. [刷新](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L329):
    3. [初始化](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L238):
        1. [注册 serviceListeners](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L212)
        2. [构造 ServiceMetadata](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L214)
    4. [doExport](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L243) ->  [doExportUrls](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L391)
        1. [向 ServiceRepository 注册服务](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L402)
        2. [构造 providerModel](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L406)
        3. [向 ServiceRepository 注册 providerModel](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L416)
        4. [获取 registryURLs](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L418)
        5. [遍历所有协议, 按协议暴露](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L429)
    5. [设置 exported](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L261)
    6. [回调 serviceListeners](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/ServiceConfig.java#L758)
6. [引用服务](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L167)

P.S. 异步暴露/引用暂时不深入讨论(TODO)

# 问题

* [ ] [ApplicationDeployer, ModuleDeployer 抽象没抽好](https://github.com/apache/dubbo/issues/11796), 子接口中有重复方法.
* [ ] 实际程序中所有 Deployer 实例, 不是 ApplicationDeployer 就是 ModuleDeployer, 没有一处直接使用 Deployer. 从面向对象的角度来说,
  Deployer 这个接口没有使用李氏替换的场景.
  AbstractDeployer 实现了 Deployer 接口, 封装了 Deployer 接口的生命周期. 但是封装的不完全, 没有把生命周期管理起来,
  而是暴露了一些基本操作, 让外部代码来管理 Deployer 的生命周期. 


 


