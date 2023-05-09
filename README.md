order-all
------
# 准备
执行init脚本拉取领域层、基础层、启动层、应用层（usercase）、ui层（web）相关代码

# 介绍
订单服务

# 好的架构是怎么样的？
- 独立于框架：架构不应该依赖某个外部的库或框架，不应该被框架的结构所束缚。
- 独立于UI：前台展示的样式可能会随时发生变化（今天可能是网页、明天可能变成console、后天是独立app），但是底层架构不应该随之而变化。
- 独立于底层数据源：无论今天你用MySQL、Oracle还是MongoDB、CouchDB，甚至使用文件系统，软件架构不应该因为不同的底层数据储存方式而产生巨大改变。
- 独立于外部依赖：无论外部依赖如何变更、升级，业务的核心逻辑不应该随之而大幅变化。

# 项目结构
```
├── domians (领域层)
 ├── customer-domain(会员域)
 ├── order-domain(订单域)
├── infrastructures (基础层，包含dao、messaging、第三方服务的等实现，也可以细粒度拆分出多个基础层。)
 ├── customer-infrastructure(会员域的基础设施层实现)
 ├── order-infrastructure(订单域的基础设施层实现)
├── apps (应用层usercase)
 ├── place-order-app（下单场景的usercase）
├── staters (启动层)
 ├── buy-stater（下单启动类springboot）
├── webs (ui层)
 ├── place-order-web（下单ui层）
```

# 各层职责
- domain领域层:领域层负责对领域实体全生命周期的管理。是核心业务逻辑的集中地。
领域层包含聚合、有状态的实体、值对象、领域服务以及各种外部依赖的接口类（Repository、MQ、外部访问接口）
未来值对象这个模块可能会单独抽成一个子module供其他模块复用。
- app应用层:活动层的主要职责是负责对用例的流程编排，这一层也是很薄的一层。活动层所暴露出的对象为贫血模型，仅仅包含数据，不包含行为。
  活动层依赖领域层Domain的sdk。不依赖任何框架。
- infrastructures基础设施层：基础设施层负责对mq、repository、第三方服务的具体实现，如RocketMQ/Kafka、mybatis/hibernate等具体实现。这一层的作用主要是将业务与技术实现相分离，屏蔽底层实现细节。使我们的软件更“软”基础设施层也可以很细粒度的拆分出多个基础设施，如DataSource/EventStore/LocalFileSystem/AsyncMessaging/OtherServices。
- web层：web这一层很薄，它仅仅只包含controller代码，这一层不应包含任何业务逻辑。
- starter层：springboot启动类，也是项目部署单元。


# 领域划分方式



# 聚合设计原则



# 微服务拆分原则



# 参考资料




	
