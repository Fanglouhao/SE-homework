# Sigma

## Sigma是什么

始于 2011 年建设的 Sigma 是服务阿里巴巴在线业务的调度系统。随着双11的越发火爆，阿里面对的问题是如何用有限的成本最大化提升用户体验和集群吞吐能力，而Sigma在双11的表现相当好，是一个优秀的集群调度系统。

## Sigma基础架构

![Sigma基础架构图](https://github.com/592McAvoy/homework1/blob/master/%E5%9B%BE%E7%89%87/%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

Sigma 有 Alikenel、SigmaSlave、SigmaMaster 三层大脑联动协作。

Alikenel 部署在每一台物理机上，对内核进行增强，在资源分配、时间片分配上进行灵活的按优先级和策略调整，对任务的时延，任务时间片的抢占、不合理抢占的驱逐都能通过上层的规则配置自行决策。

SigmaSlave 可以在本机进行容器 CPU 分配、应急场景处理等。通过本机 Slave 对时延敏感任务的干扰快速做出决策和响应，避免因全局决策处理时间长带来的业务损失。

SigmaMaster 是一个最强的中心大脑，可以统揽全局，为大量物理机的容器部署进行资源调度分配和算法优化决策。

## Characteristic

### 统一大资源池模式
最初阿里的调度系统是T4系统，T4将资源分为各个小组（大部分都是小规模小组），将各个部门的资源池独立出来，并且独立研发多套调度系统。
这样的调度系统具有明显的缺点：
- 规模：各T4分组规模不一，大部分都是小规模：资源碎片化。
- 调度：T4分组内小规模调度，核心应用打散受限。
- 资源分配：双11期间参差不齐，表现在交易相关CPU充分售卖，无空闲CPU；但众多T4分组，宿主机尚未分配容器实例。

![T4架构图](https://github.com/592McAvoy/homework1/blob/master/%E5%9B%BE%E7%89%87/T4.png)

随后阿里对其进行了改变，即之后的Sigma系统架构，其主要体现在云化架构、混合云：
- 规模：统一大资源池模式。
- 调度：大资源池下，Sigma调度对核心应用的各种策略保障，得以更充分地发挥价值。
- 资源分配：双11充分只用了所有资源，没有闲置。

![Sigma架构图](https://github.com/592McAvoy/homework1/blob/master/%E5%9B%BE%E7%89%87/Sigma.png)

### 兼容性
Sigma兼容Kubernetes API，和开源社区共建。同时，即便在Sigma中阿里并没有使用比较普遍的docker容器而是自行研发的Pouch容器，但其依旧兼容OCI标准。

### 灵活可配置的调度策略
Sigma允许以插件化地方式基于外部输入实时调控集群打分模型，并且允许用户配置优化调度策略。业务团队开发出新的策略，可立即配置生效，不需要代码发布。

![灵活配置](https://github.com/592McAvoy/homework1/blob/master/%E5%9B%BE%E7%89%87/%E7%81%B5%E6%B4%BB%E9%85%8D%E7%BD%AE.png)

### 离线任务与在线任务混部
详见Sigma与Fuxi混部架构

## Sigma与Fuxi混部架构

Fuxi和Sigma都是阿里的调度架构，但Fuxi是一个离线计算引擎，主要解决离线任务。
阿里根据流量走势，动态对Sigma和Fuxi进行扩容和缩容，以进一步提高利用率。

![混部](https://github.com/592McAvoy/homework1/blob/master/%E5%9B%BE%E7%89%87/%E6%B7%B7%E9%83%A8.png)

如图所示，根据流量走势可以得出凌晨1点开始，在线业务流量进入低谷，此时既可以对离线任务（Fuxi）进行扩容，对Sigma缩容。清晨7点开始，在线业务流量开始增长，此时可以对离线任务进行缩容，对Sigma扩容，以此达到更大的资源利用率。

## Pros & Cons

### Pros
如前文所说，Sigma具有如下的优点：
- 利用率高：Sigma摒弃了之前阿里调度架构的小规模、碎片化的缺点，采用统一大资源池模式，大幅提升利用率。
- 兼容性好：兼容目前比较流行的Kubernetes调度系统的API。
- 灵活度高：可以自行配置调度策略而无需代码发布。
- 混部提升利用率：与Fuxi结合的混部架构，采取分时复用，动态分配容量，进一步提升资源利用率。

### Cons
- 在与Fuxi混部方面，容量预测目前还是个挑战，并且扩容缩容的稳定性还没有办法得到保证。

## Comments

Sigma是一个非常优秀的调度框架，其中相对于原来T4的进步主要在于实现了统一大资源池模式更高效地使用了资源。而Sigma和Fuxi的混部更像是天然契合，根据流量走势而动态分配离线、在线任务容量的想法也非常具有可行性和有效性。

但Sigma在双11表现十分优秀很大程度上也由于Sigma对于双11的特殊情况做了许多specific的优化，它在一般情况上的表现是否优于别的调度架构仍不确定。

## 参考资料

 1. [阿里巴巴集群与调度系统Sigma][1]
 2. [阿里巴巴 Sigma 调度和集群管理系统架构详解][2]
 3.  [史无前例开放！阿里内部集群管理系统Sigma混布数据][3]


  [1]: http://www.infoq.com/cn/presentations/alibaba-scheduling-and-cluster-management-system-sigma
  [2]: http://blog.51cto.com/13778063/2155360
  [3]: https://blog.csdn.net/dongyuxia15810857916/article/details/77866404
  
  
  
