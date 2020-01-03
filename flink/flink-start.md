# Flink

## 了解Flink 

应用开发需要先理解Flink 的Streams、State、Time 等基础处理语义以及Flink 兼顾灵活性和方便性的多层次API。

- Streams：流，分为有限数据流与无限数据流，unbounded stream 是有始无终的数据流，即无限数据流；而bounded stream 是限定大小的有始有终的数据集合，即有限数据流，二者的区别在于无限数据流的数据会随时间的推演而持续增加，计算持续进行且不存在结束的状态，相对的有限数据流数据大小固定，计算最终会完成并处于结束的状态。

- State，状态是计算过程中的数据信息，在容错恢复和Checkpoint 中有重要的作用，流计算在本质上是Incremental Processing，因此需要不断查询保持状态；另外，为了确保Exactly- once 语义，需要数据能够写入到状态中；而持久化存储，能够保证在整个分布式系统运行失败或者挂掉的情况下做到Exactly- once，这是状态的另外一个价值。

- Time，分为Event time、Ingestion time、Processing time，Flink 的无限数据流是一个持续的过程，时间是我们判断业务状态是否滞后，数据处理是否及时的重要依据。

- API，API 通常分为三层，由上而下可分为SQL / Table API、DataStream API、ProcessFunction 三层，API 的表达能力及业务抽象能力都非常强大，但越接近SQL 层，表达能力会逐步减弱，抽象能力会增强，反之，ProcessFunction 层API 的表达能力非常强，可以进行多种灵活方便的操作，但抽象能力也相对越小。

## Flink Architecture

第一，Flink 具备统一的框架处理有界和无界两种数据流的能力

第二， 部署灵活，Flink 底层支持多种资源调度器，包括Yarn、Kubernetes 等。Flink 自身带的Standalone 的调度器，在部署上也十分灵活。

第三， 极高的可伸缩性，可伸缩性对于分布式系统十分重要，阿里巴巴双11大屏采用Flink 处理海量数据，使用过程中测得Flink 峰值可达17 亿/秒。

第四， 极致的流式处理性能。Flink 相对于Storm 最大的特点是将状态语义完全抽象到框架中，支持本地状态读取，避免了大量网络IO，可以极大提升状态存取的性能。

## Watermarks

Flink实际上是用 Watermarks 来实现Event – Time 的功能。Watermarks 在Flink 中也属于特殊事件，其精髓在于当某个运算值收到带有时间戳“ T ”的 Watermarks 时就意味着它不会接收到新的数据了。使用Watermarks 的好处在于可以准确预估收到数据的截止时间。举例，假设预期收到数据时间与输出结果时间的时间差延迟5 分钟，那么Flink 中所有的 Windows Operator 搜索3 点至4 点的数据，但因为存在延迟需要再多等5分钟直至收集完4：05 分的数据，此时方能判定4 点钟的资料收集完成了，然后才会产出3 点至4 点的数据结果。这个时间段的结果对应的就是 Watermarks 的部分。

## 状态保存与迁移

流式处理应用无时无刻不在运行，运维上有几个重要考量：

> 1. 更改应用逻辑/修bug 等，如何将前一执行的状态迁移到新的执行？
> 2. 如何重新定义运行的平行化程度？
> 3. 如何升级运算丛集的版本号？

Checkpoint完美符合以上需求，不过Flink中还有另外一个名词保存点（Savepoint），当手动产生一个 Checkpoint 的时候，就叫做一个 Savepoint。Savepoint 跟 Checkpoint 的差别在于 Checkpoint 是 Flink 对于一个有状态应用在运行中利用分布式快照持续周期性的产生 Checkpoint，而 Savepoint 则是手动产生的Checkpoint，Savepoint 记录着流式应用中所有运算元的状态。

Savepoint产生的原理是在Checkpoint barrier 流动到所有的Pipeline 中手动插入从而产生分布式快照，这些分布式快照点即Savepoint。Savepoint 可以放在任何位置保存，当完成变更时，可以直接从Savepoint 恢复、执行。

从Savepoint 的恢复执行需要注意，在变更应用的过程中时间在持续，如Kafka 在持续收集资料，当从Savepoint 恢复时，Savepoint 保存着Checkpoint 产生的时间以及Kafka 的相应位置，因此它需要恢复到最新的数据。无论是任何运算，Event – Time 都可以确保产生的结果完全一致。

假设恢复后的重新运算用Process Event – Time，将 Windows 窗口设为1 小时，重新运算能够在10 分钟内将所有的运算结果都包含到单一的 Windows 中。而如果使用Event – Time，则类似于做Bucketing。在Bucketing 的状况下，无论重新运算的数量多大，最终重新运算的时间以及Windows 产生的结果都一定能保证完全一致。


