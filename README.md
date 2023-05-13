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
  未来值对象可能会单独抽成一个模块供其他模块复用。
- app应用层:活动层的主要职责是负责对用例的流程编排，这一层也是很薄的一层。活动层所暴露出的对象为贫血模型，仅仅包含数据，不包含行为。
  活动层依赖领域层Domian的sdk。不依赖任何框架。
- infrastructures基础设施层：基础设施层负责对mq、repository、第三方服务的具体实现，如RocketMQ/Kafka、mybatis/hibernate等具体实现。
  这一层的作用主要是将业务与技术实现相分离，屏蔽底层实现细节。使我们的软件更“软”
  基础设施层也可以很细粒度的拆分出多个基础设施，如DataSource/EventStore/LocalFileSystem/AsyncMessaging/OtherServices。
- web层：web这一层很薄，它仅仅只包含controller代码，这一层不应包含任何业务逻辑。
- starter层：springboot启动类，也是项目部署单元。

# 领域划分方式
- step1：需求分析：

  在进行域划分之前，用户需求场景分析，使用uml图画出usercase用例图，识别出actor及usecase。
- step2：分析模型鲁帮图，识别出业务场景中所有的实体对象：
  鲁棒图包含三种图形：边界、控制、实体
  1. 边界——起与外界交互的作用，它只能与控制对象和执行者有关系
  2. 控制——对业务控制、流程控制的作用，它能与边界对象和实体对象有关系
  3. 实体——业务元素的存储对象，与领域模型中的对象有良好的关系。它只能与控制对象有关系

  通过step1的每个usecase进行鲁棒图绘制，找出所有的实体对象。
- step3：领域划分，将所有识别出的实体对象进行分类
  个人而言，我对广泛的领域（这里不讨论子域）的理解是解决某一类问题的领域，域的职责是一个或多个实体相同“类型”对象的信息集合，并对所管理的实体对象的生命周期进行管理。
  接下来，我们通过对所有实体对象进行分类，就可以完成域划分了。划分依据:
  1. 基于领域模型的对象关系，有些对象明显属于一个某一个范围，这个范围相当于对这些对象的一层抽象和总结，比如SKUDO、SPUDO、SkuVersionDO这些虽然没有表达商品的概念也没有显示的定义
     ItemDO但是我们可以很容易的通过关联关系来把他们画成一类。
  2. 基于数据模型的对象关系，通过E-R图来分析，如果某些表之间通过id或者其他业务属性字段进行关联，那它们就有亲缘关系可以划分至一个领域中，但这并不是绝对的，比如商品里常见有一些类别属性、
     而用户也有类别属性、交易也有一些类别属性，虽然商品、用户、交易、都与类别有关联，但这种时候也可以把类别单独抽成一个领域去划分，因为域本质上是一类问题的领域，拆分的好处可以让商品、
     用户、交易更关注自己内部核心的业务逻辑。
  3. 基于业务流程的交互关系，尝试划分领域的一个最好的实践就是去了解业务流程，通过时序图，通过流程图我们可以从不同的节点事件来表达这些节点事件的关联关系。比如外卖，对
     于用户来说他做的就是浏览菜单下单支付然后拿外卖，在用户操作的这个领域来说他就只有几个节点，这些节点还包括商家接单，商家把单子告诉厨房，厨房准备菜品，做好了外卖员要么在取餐的路上要么在等餐的路上，
     或者在送餐的路上。那这个例子就告诉我们一个业务流程如果我们不分割为多个领域的话我们几乎无法完成一整套的操作。
  4. 基于抽象复用的要求，公司有多个业务线，每个业务线都包含用户、权限等功能，那么可以把用户权限独立出来单独领域建模
- step4：评估域划分合理性，并进行优化：

  判定原则非常简单，就是看域划分是否符合“高内聚、低耦合”原则。通过对每个usecase画出所有的时序图在这个时序图上，生命周期线是按照“域”这个
  维度来看。在绘制过程中，我们可以看域与域之间的调用是否过于频繁？甚至是反复调用不同的域服务？如果存在这种情况，就意味着这两个域之间存在比较严重的耦合。
  这往往通过直观就能有个大致的判断。如果要从定量角度来分析，可以参考代码圈复杂度的度量算法，我们也可以设定一个“域依赖度”算法，来衡量域与域之间的依赖度。
  对于依赖度比较高的几个域，我们可以采用：1）域合并 2）域拆分 3）提取第三方域做依赖倒置等降低耦合。

# 聚合设计原则
- 聚合内的保证一致性，事务一致性原子、最终一致性。
- 设计小聚合。
- 通过唯一标识引用其他聚合。
- 边界之外使用最终一致性。
- 迪米特原则 最小知识，任何对象的方法只能调用对象自身的方法，传入参数对象的方法，所创建对象的方法，自身包含对象的方法。

# 微服务拆分原则
领域划分属于逻辑层面的划分，而微服务属于"物理架构"。

微服务拆分主要是两刀，一刀纵向切，根据场景进行拆分，比如"下单"可以拆分出一个下单服务、"商品查询"也可以拆分出一个商品查询服务、
"商品编辑"也可以拆分成一个商品编辑微服务。
第二刀横向切，数据库操作CURD这一层不允许直连，得通过数据服务去管控实体。


# 参考资料
- https://www.ryu.xin/2018/04/08/domain/
- https://mp.weixin.qq.com/s/MU1rqpQ1aA1p7OtXqVVwxQ
- https://developer.aliyun.com/article/861364?spm=a2c6h.12873639.article-detail.65.59a562b4UHZxw9
- 《领域驱动设计：软件核心复杂性应对之道》
- 《实现领域驱动设计》
- 《架构整洁之道》


  
