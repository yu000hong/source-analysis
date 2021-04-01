# 第一章 简介

### 分布式系统定义

> A distributed system is a system whose components are located on **different networked computers**, which communicate and coordinate their actions by **passing messages** to one another.

分布式系统就是由不同的机器结点通过网络连接起来，通过网络相互之间传送消息来进行通信协调形成的一个整体。

### 为什么需要分布式系统

由于单机结点存在**单点故障**，同时无法无限纵向扩展，纵向扩展成本太高，即**扩展性不强**、**性能瓶颈**等问题，所以我们需要分布式系统来解决这些问题，但是软件工程里面没有银弹。

所以，分布式系统同时也会带来其他问题，包括：增加了架构设计的复杂性，带来了一致性问题。

分布式提供的三大能力：

- Performance：高性能（包括两方面：低延迟和高吞吐）
- Scalability：扩展性（横向扩展更容易实现，成本较低）
- Availability：高可用（通过多个副本的冗余实现高可用）

分布式系统不仅能拆分数据，同时也能拆分计算：

> Building a distributed system allows us to split and store the data in multiple nodes, while also distributing th processing work amongst them.

### 分布式系统误解

1、网络是可靠的

2、网络是安全的

3、网络带宽是无限的

4、网络类型都是一样的

5、网络拓扑是不会改变的

6、通信延迟为零

7、传输代价为零

8、总是有一个网络管理员

9、分布式系统有一个全局时钟

### 分布式系统为何这么难

主要有三方面原因：

- 异步网络：无法保证通信的送达，响应的及时回复
- 部分失败：保证不了操作的原子性，有的成功了有的失败了，导致状态不一致
- 并发执行：多个操作在不同结点同时执行，相互影响，保证补了隔离性，会导致意想不到的情况

### 分布式系统正确性

分布式系统要保证最终的正确性，那么必须同时满足两个属性：

- Safety(安全)：有些事情是永远不可能发生的，这就是安全属性
- Liveness(活性)：有些事情最终肯定会发生，这就是活性属性

但现实情况是，有的时候要同时满足安全和活性两个属性是不太现实了。而对于分布式系统来说，安全属性的重要性肯定是大于活性属性的。
所以，在无法同时满足安全和活性的情况下，我们一般会舍弃一定程度的活性需求，必须保证安全。

### 系统模型

真实生活中的各个分布式系统往往具有很大的不同，我们怎么去比较不同的分布式系统他们的差异呢？

为了可以进行这种比较，我们建立了系统模型(**System Model**)这个概念，这个模型由不同维度的属性构成。我们在比较两个系统的异同点时，就可以基于这个模型里面的各个属性进行细致的比较。

这个模型里面最重要的两个属性就是：

- 不同结点之间时如何通信的，同步还是异步
- 每个结点的失败类型有哪些

**通信类型**

- 同步系统：不同结点间的通信延迟是具有一定上限的，在指定的时间内肯定会有所响应
- 异步系统：不同结点间的通信延迟不确定，没有上限，可能一直没有响应，也可能很快响应

现实情况下，分布式系统都是异步系统！

**失败类型**

- Fail-Stop：结点**永远**挂了，同时其他结点能够发现这个结点挂了
- Crash：结点挂了，但是其他结点可能发现不了它已经挂了，只能发现跟这个结点的通信没有响应；其他结点也有可能会认为这个结点只是处理有点慢
- Omission：结点没有挂，但是无法响应其他结点的请求了
- Byzantine：结点的行为是不可控的，它有可能不响应其他结点的请求，有可能正常响应，有可能伪造一些错误数据，也有可能已经挂了

> **总结**：在分布式系统中，Fail-Stop是最简单也是最容易处理的失败类型。但在真实的系统环境中，我们很难确定一个结点到底是不是真的挂了！
> 所以，在真实的分布式环境中，我们都会假设系统处于`Crash`这种失败类型。也即是，当一个结点在指定时间没有响应时，我们是无法确定它是真的挂了，还是处理变慢了。

### 仅仅一次

由于网络是不可靠的，消息存在丢失的可能。为了避免丢失消息，结点可能会尝试重新发送该消息。但重试发送其实也存在问题，考虑下面这种情况：

> 因为现实中都是异步系统，所以结点间的通信会设置一个超时时间，在这个超时时间内对方结点如果没有响应的话，我们会假设对方可能没有收到这条消息，**消息丢失了**。为了确保这条消息一定会送达对方，我们会尝试重新发送。由于现实情况可能并不是消息真的丢失了，而是由于网络延迟导致消息接收靠后了，或者由于对方结点处理这条消息太慢了，导致超时时间内没有及时响应。这样就会导致对方结点同时收到两条一样的消息，也即是**消息重复了**。

所以现实情况再一次对我们进行了毒打，我们没法真正实现`仅仅一次`的消息送达！在提及到“仅仅一次”语义的时候，一定要分清楚到底是消息送达还是消息处理。

虽然我们无法保证消息的**仅仅一次送达**，但是我们可以通过一些手段来保证消息的**仅仅一次处理**，有两种方式可以实现：

- 幂等操作
- 去重方法

**幂等操作**

什么是幂等操作呢？就是我们在多次处理同一消息之后导致的系统状态和我们仅仅处理该消息一次后对应的系统状态必须是一致的。

- 比如DEL操作就是幂等的，多次删除和一次删除最终结果都是对应的数据被删除掉
- 比如SET操作就是幂等的，多次SET和一次SET导致数据的值都是一样的
- 比如INCR操作就非幂等的，每次增加都是在原来的基础上的，多次增加和一次增加结果就不一样

现实情况是，我们很难保证所有的操作都是幂等的！因此，我们需要**去重方法**来保证仅仅一次！

**去重方法**

要实现去重方法的话，我们需要对系统的两端都能掌控，也即是在发送端要为每条消息都是生成一个全局唯一ID，接收端收到这条消息之后要能通过这个全局唯一ID去判断这条消息是否已经处理过了。如果没处理，则进入处理流程；如果已经处理过了，那么直接忽略。这样我们就保证了消息的**仅仅一次处理**。

举个例子，我们在接入支付系统的时候，第三方系统的支付接口都会让我们提供一个唯一ID，这个唯一ID一般是我们自己系统里面的订单ID。第三方支付系统会根据这个订单ID来检测是否重复调用接口，保证了我们系统订单不会被重复支付，这就是去重方法的有效运用！但现实系统就是这么复杂，我们如果在接入多个支付三方的情况下很难保证订单不会重复支付。

考虑下面这种情况，当用户使用微信支付进行订单支付的时候，由于某些原因超时了，用户界面显示的可能就是网络异常或支付失败等。为了完成订单支付，用户改用支付宝进行支付，最终支付成功了。过了一会儿，微信支付提醒用户“您的支付已完成”，这时候其实这个订单已经重复支付了。虽然我们使用了全局唯一的订单ID，但最后还是导致重复支付了，这种情况是因为微信支付和支付宝是两个系统，他们无法互通信息。这里要避免重复支付的话，还得我们系统自己再做一次**去重方法**，在第二次调用支付宝接口前，先去微信支付查询这个订单是否支付成功。如果已经支付成功，则告知用户；如果支付未成功，继续使用支付宝支付。

到这里，大家可能觉得很完美，但是但是，从理论角度讲，这种情况我们其实无法真正避免重复支付！因为我们在去微信支付查询的时候，有可能由于网络延迟微信支付还没有收到我们的确认支付请求，也有可能微信支付还在对这笔支付处理中，也有可能我们得不到任何响应，所以，还会可能再进行支付宝支付导致重复支付。这就是真实情况下的分布式系统！所以，我们就必须定期对**未知状态**的支付纪录进行查询，直到拿到一个最终结果再进行补偿处理（重复支付则进行退还操作）。严格来说，我们不仅仅要对未知状态的支付进行查询，对**待支付状态**的支付纪录最好进行再次确认查询，避免因为网络延时导致的重复支付情况！

> **总结**：
> 1. 我们无法保证消息的**仅仅一次送达**，但是我们可以通过一些手段来保证消息的**仅仅一次处理**
>
> 2. 针对消息送达，我们没法保证仅仅一次，但是我们可以保证**最多一次送达**和**至少一次送达**

### 分布式系统故障检测

### 无状态系统 & 有状态系统

**无状态系统**

无状态系统就是我们不会持有数据，系统只会根据输入的数据进行处理，并把处理结果及时返回或者存储到第三方系统。
这些输入数据可以是直接的也可以间接的，直接的就是包含在请求中的数据，间接的就是要从第三方系统去取的数据。

无状态系统最大的优势就是横向扩展性，可以根据并发的吞吐量部署任意多台机器来处理。

**有状态系统**

有状态系统就是系统本身要去修改和维护系统相关数据，维护这些数据会给系统带来复杂性，比如我们怎么存储这些数据、怎么备份、怎么保证高可用高性能、如何保证副本数据的一致性。

> **总结**：无状态系统比有状态系统更容易实现更具扩展性，那是因为无状态系统中的所有结点都是一样的，而有状态系统中的不同结点会持用数据的不同分区/部分！