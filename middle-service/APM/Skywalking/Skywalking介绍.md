### Skywalking背景

> - Apache 孵化器项目
> - 加入 OneAPM 公司

### Skywalking 介绍（分布式 APM 系统 / 分布式追踪系统） 

> - ##### 全自动探针检测，不需要修改应用程序代码（可支持的中间件和组件：）
>
> - ##### 支持手动探针监控，用埋点的方式手动上传业务数据；
>
> - ##### SkyWalking提供了支持 OpenTracing 标准的 SDK。也支持 OpenTracing-Java 支持的组件
>
> - ##### 自动监控和手动监控可以同时使用，使用手动监控弥补自动监控不支持的组件，或者私有化组件；
>
> - ##### 纯 Java 后端分析程序，提供 RESTful 服务，可为其他语言探针提供分析能力；
>
> - ##### 高性能纯流式分析；
>
> - ##### 可将 traceId。集成到主流的日志框架中输出，如 log4j，logback 等
>
> - ##### 对应的 WEB 页面由单独的工程进行发布（sky-walking-ui）

### Skywalking 架构图

> ![架构图](https://note.youdao.com/yws/api/personal/file/WEB4b3841616dbf0e7d80036fc53c2bdf3b?method=download&shareKey=33b552f37bf7ec2e597b48868affea3e)
>
> - Skywalking-web：web 可视化平台，用来展示数据
> - skywalking-collector：链路数据归集器，数据可以落地 ElasticSearch，单机也可以落地 H2，不推荐，H2 仅作为临时演示用
> - skywalking-agent：探针，用来收集和发送数据到归集器

### Skywalking 目前暴露的三个问题

> **现象**：
>
> > - Agent 和 Collector 正常工作，没有异常日志
> > - 已经对系统进行过访问，Trace 查询有数据
> > - UI 除 Trace 查询页面外，其他页面无数据
>
> **原因**： Collector 和被监控应用的系统主机时间，没有同步
>
> **解决方法**： 同步各主机操作系统时间

> **现象** ： Kafka 消息消费端链路断裂
>
> **原因**： Kafka 探针只是追踪了对 Kafka 的拉取动作，而整个后续处理过程不是由 kafka consumer 发起。故需要在消费处理的发起点，进行手动埋点
>
> **解决方法**: 可以通过 Application Toolkit 中的`@Trace`标注，或者 OpenTracing API 进行手动埋点

> **现象**：
>
> > - 加载探针并启动应用
> > - Console 中被 GRPC 日志刷屏
>
> **原因**：Skywalking 采用了 GRPC 框架发送数据，GRPC 框架读取 log 的配置文件进行日志输出。
>
> **解决方法**: 在log的配置文件中添加对`org.apache.skywalking.apm.dependencies`包的过滤

### Dapper 的缺点

> - 合并的影响：我们的模型隐含的前提是不同的子系统在处理的都是来自同一个被跟踪的请求。在某些情况下，缓冲一部分请求，然后一次性操作一个请求集会更加有效。（比如，磁盘上的一次合并写入操作）。在这种情况下，一个被跟踪的请求可以看似是一个大型工作单元。此外，当有多个追踪请求被收集在一起，他们当中只有一个会用来生成那个唯一的跟踪 ID，用来给其他 span 使用，所以就无法跟踪下去了。我们正在考虑的解决方案，希望在可以识别这种情况的前提下，用尽可能少的记录来解决这个问题。
> - 跟踪批处理负载：Dapper 的设计，主要是针对在线服务系统，最初的目标是了解一个用户请求产生的系统行为。然而，离线的密集型负载，例如符合MapReduce[10]模型的情况，也可以受益于性能挖潜。在这种情况下，我们需要把跟踪ID与一些其他的有意义的工作单元做关联，诸如输入数据中的键值（或键值的范围），或是一个 MapReduce shard。
> - 寻找根源：Dapper 可以有效地确定系统中的哪一部分致使系统整个速度变慢，但并不一定能够找出问题的根源。例如，一个请求很慢有可能不是因为它自己的行为，而是由于队列中其他排在它前面的(queued ahead of)请求还没处理完。程序可以使用应用级的 annotation 把队列的大小或过载情况写入跟踪系统。可以采用成对的采样技术可以解决这个问题。它由两个时间重叠的采样率组成，并观察它们在整个系统中的相对延迟。
> - 记录内核级的信息：一些内核可见的事件的详细信息有时对确定问题根源是很有用的。我们有一些工具，能够跟踪或以其他方式描述内核的执行，但是，想用通用的或是不那么突兀的方式，是很难把这些信息到捆绑到用户级别的跟踪上下文中。我们正在研究一种妥协的解决方案，我们在用户层面上把一些内核级的活动参数做快照，然后绑定他们到一个活动的 span 上。

### Skywalking竞品

> #### 同类型企业级收费产品
>
> > - **OneAPM**：http://www.oneapm.com/
> > - **透视宝**：http://www.toushibao.com/product_server.html
> > - **华为**：http://www.huaweicloud.com/product/apm.html?utm_source=Baidu&utm_medium=cpc&utm_campaign=CP-APM&utm_content=CJ&utm_term=APM
> > - **Testin听云**：https://www.testin.cn/
>
> #### 开源工具
>
> > - **Twitter Zipkin**：http://zipkin.io/ （finagle 分布式微服务框架）
> > - **美团点评 CAT**：https://link.jianshu.com/?t=https://github.com/dianping/cat
> > - **应用性能管理工具 PinPoint**：https://link.jianshu.com/?t=https://github.com/naver/pinpoint
> > - **Apache HTrace**：https://link.jianshu.com/?t=http://htrace.incubator.apache.org/

<div style="text-align:center;margin-top:50px;margin-bottom:50px;">
    <img src="https://note.youdao.com/yws/api/personal/file/C2C6FCFDC10942B6A3532E6F0928E455?method=download&shareKey=c554dacfc5193c29d4b35682aa1226d9" />
</div>
