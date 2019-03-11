---
title: Disruptor
categories:
 - Java
tags: 
 - Java
 - concurrent
---

Disruptor是一个高性能的异步处理框架，
或者可以认为是最快的消息框架(轻量级的JMS)，
也可以认为是一个观察者模式的实现，
或者事件监听模式的实现。
首先我们可以理解它为一种高效的"生产者-消费者"模型，
其性能远远高于传统的BlockingQueue容器。

Disruptor存储数据的核心叫做RingBuffer,我们通过Disruptor实例
拿到它，然后把数据生产出来，把数据加入到RingBuffer的实例对象中。

#### concepts
* RingBuffer: 被看作Disruptor最主要的组件，然而从3.0开始RingBuffer仅仅负责存储和更新在Disruptor中流通的数据。
    对一些特殊的使用场景能够被用户(使用其他数据结构)完全替代。
* Sequence: Disruptor使用Sequence来表示一个特殊组件处理的序号。
    和Disruptor一样，每个消费者(EventProcessor)都维持着一个Sequence。
    大部分的并发代码依赖这些Sequence值的运转，因此Sequence支持多种当前为AtomicLong类的特性。
* Sequencer: 这是Disruptor真正的核心。
    实现了这个接口的两种生产者（单生产者和多生产者）均实现了所有的并发算法，为了在生产者和消费者之间进行准确快速的数据传递。
* SequenceBarrier: 由Sequencer生成，并且包含了已经发布的Sequence的引用，
    这些的Sequence源于Sequencer和一些独立的消费者的Sequence。它包含了决定是否有供消费者来消费的Event的逻辑
    
随着你不停地填充这个buffer（可能也会有相应的读取），这个序号会一直增长，直到绕过这个环。
![RingBufferWrapped](https://github.com/xuguangwu/blog/blob/master/_posts/images/RingBufferWrapped.png?raw=true)
要找到数组中当前序号指向的元素，
可以通过mod操作：sequence mod array length = array index（取模操作）
以上面的ringbuffer为例（java的mod语法）：12 % 10 = 2。

如果你看了维基百科里面的关于环形buffer的词条，你就会发现，我们的实现方式，与其最大的区别在于：没有尾指针。
我们只维护了一个指向下一个可用位置的序号。
这种实现是经过深思熟虑的—我们选择用环形buffer的最初原因就是想要提供可靠的消息传递。

我们实现的ring buffer和大家常用的队列之间的区别是，我们不删除buffer中的数据，也就是说这些数据一直存放在buffer中，直到新的数据覆盖他们。
这就是和维基百科版本相比，我们不需要尾指针的原因。ringbuffer本身并不控制是否需要重叠。
因为它是数组，所以要比链表快，而且有一个容易预测的访问模式。
这是对CPU缓存友好的，也就是说在硬件级别，数组中的元素是会被预加载的，
因此在ringbuffer当中，cpu无需时不时去主存加载数组中的下一个元素。
其次，你可以为数组预先分配内存，使得数组对象一直存在（除非程序终止）。
这就意味着不需要花大量的时间用于垃圾回收。
此外，不像链表那样，需要为每一个添加到其上面的对象创造节点对象—对应的，当删除节点时，需要执行相应的内存清理操作。