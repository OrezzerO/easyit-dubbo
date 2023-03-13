# Deployer, ScopeModel 


## Deployer 是什么 
[Deployer](../dubbo-common/src/main/java/org/apache/dubbo/common/deploy/Deployer.java#L24) 是 Dubbo 的部署模块. 负责 Dubbo 的初始化工作.
Deployer 共有两个实现类 [DefaultApplicationDeployer](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L93) 和 [DefaultModuleDeployer](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L59).
它们分别负责部署应用和模块.

## ScopeModel 是什么

[//]: # (todo ScopeModel 是什么)
ScopeModel 一共分为三种:
* FrameworkModel
* ApplicationModel
* ModuleModel

FrameworkModel 包含多个 ApplicationModel, ApplicationModel 包含多个 ModuleModel. ApplicationModel 和 ApplicationDeployer 相互持有; ModuleModel 和 ModuleDeployer 相互持有.   

## Deployer 的生命周期
Deployer 的生命周期有如下几个状态:
1. PENDING
2. STARTING
3. STARTED
4. STOPPING
5. STOPPED
6. FAILED

## Applciation Deployer 的生命周期


### ApplciationDeployer 对象的创建



回顾下启动分析章节, 我们使用 DubboBootstrap 来[启动 Dubbo 服务](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L43).有以下三个步骤:
1. [创建 DubboBootstrap 实例](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L48): 创建过程中, 我们会[提供一个 ApplicationModel](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L111). DubboBootstrap 会[保存 ApplicationModel 中的 applicationDeployer](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L166).
2. 配置 DubboBootstrap 相关的参数. 
3. 调用 DubboBootstrap 的 [start](../dubbo-demo/dubbo-demo-api/dubbo-demo-api-provider/src/main/java/org/apache/dubbo/demo/provider/Application.java#L53) 方法.

在 start 方法中, 我们实际调用的是 applicationDeployer 的 [start](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/bootstrap/DubboBootstrap.java#L224) 方法.

我们就从 applicationDeployer 的 start 方法, 开始分析 Deployer 状态的变化:
1. 首次进入 start 方法, applicationDeployer 处于 PENDING 状态. 在 [onStarting](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L573) 方法会将 applicationDeployer 的状态改为 STARTING .
2. 如果在 start 方法中,最终有异常抛出, [则将状态改为 FAILED](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L579)
3. 在 [doStart](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L577) 的过程中, applicationDeployer 会调用每一个 DefaultModuleDeployer, 在每一个 Module 启动完成之后,会调用 applicationDeployer 的 [notifyModuleChanged](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultModuleDeployer.java#L264).
通过 [calculateState](../dubbo-config/dubbo-config-api/src/main/java/org/apache/dubbo/config/deploy/DefaultApplicationDeployer.java#L865) 方法, 通过判断 module deployer的状态, 设置 applicationDeployer 的状态为 STARTED.


### Module Deployer 的生命周期























 





  



 


## 问题
* [ ] [ApplicationDeployer, ModuleDeployer 抽象没抽好](https://github.com/apache/dubbo/issues/11796), 子接口中有重复方法.
* [ ] 实际程序中所有 Deployer 实例, 不是 ApplicationDeployer 就是 ModuleDeployer, 没有一处直接使用 Deployer. 从面向对象的角度来说, Deployer 这个接口没有使用李氏替换的场景.
AbstractDeployer 实现了 Deployer 接口, 封装了 Deployer 接口的生命周期. 但是封装的不完全, 没有把生命周期管理起来, 而是暴露了一些基本操作, 让外部代码来管理 Deployer 的生命周期. 


 


