# Consumers

Consumers can be conceived as 'views' into a stream, with their own 'cursor'. Consumers iterate or consume over all or a subset of the messages stored in the stream, according to their 'subject filter' and 'replay policy', and can be used by one or multiple client applications. It's ok to define thousands of these pointing at the same Stream.  
消费者可以被理解为一种带有“游标”的流“视图”。消费者根据他们的“主题过滤器”和“重放策略”，对存储在流中的全部消息或部分消息进行迭代或消费，并且可以被一个或多个客户端应用程序使用。也可以定义上千个消费者消费同一个Stream。 

Consumers can either be `push` based where JetStream will deliver the messages as fast as possible \(while adhering to the rate limit policy\) to a subject of your choice or `pull` to have control by asking the server for messages. The choice of what kind of consumer to use depends on the use-case but typically in the case of a client application that needs to get their own individual replay of messages from a stream you would use an 'ordered push consumer', while in the case of scaling horizontally the processing of messages from a stream you would use a 'pull consumer'.  
消费者获取消息可以是基于push模式，JetStream将尽可能快地(同时遵守速率限制策略)将消息交付给您选择的主题，也可以是请求服务器通过pull模式来获取消息。选择使用哪种消费模式取决于用例，但通常在客户端应用程序需要从流中获取消息做重复消费的情况下，您应该使用“ordered push consumer”，而在处理流消息水平扩展的情况下，你应该使用“pull consumer”。

The rate of message delivery in both cases is subject to `ReplayPolicy`. A `ReplayInstant` Consumer will receive all messages as fast as possible while a `ReplayOriginal` Consumer will receive messages at the rate they were received, which is great for replaying production traffic in staging.  
这两种情况下的消息交付速率均受 ReplayPolicy 策略的约束。 ReplayInstant 模式消费者将尽可能快地接收所有消息，而 ReplayOriginal 模式消费者将以消息的接收速率接收消息，这对于在 staging 中重放生产流量非常有用。  

In the orders example above we have 3 Consumers. The first two select a subset of the messages from the Stream by specifying a specific subject like `ORDERS.processed`. The Stream consumes `ORDERS.*` and this allows you to receive just what you need. The final Consumer receives all messages in a `push` fashion.  
在上面的订单示例中，我们有 3 个消费者。前两个通过指定特定主题（如 ORDERS.processed）从 Stream 中选择消息的子集。 Stream 使用 ORDERS.*，这使您可以接收所需的内容。最终消费者以stream推送消息的方式接收所有消息。  

Consumers track their progress, they know what messages were delivered, acknowledged, etc., and will redeliver messages they sent that were not acknowledged. When first created, the Consumer has to know what message to send as the first one. You can configure either a specific message in the set \(`StreamSeq`\), specific time \(`StartTime`\), all \(`DeliverAll`\) or last \(`DeliverLast`\). This is the starting point and from there, they all behave the same - delivering all of the following messages with optional Acknowledgement.
消费者会跟踪消费的进度，他们知道哪些消息被传递、被确认等，并且会重新传递他们发送的未被确认的消息。当第一次创建时，消费者必须知道（设置）第一条消息的发送将以那种方式发送。您可以配置为从指定消息序号 (StreamSeq)、指定的时间 (StartTime)、全部 (DeliverAll) 或最后 (DeliverLast)消费subject的消息。他们的行为都是一样的，使用可选的Acknowledgement传递所有以下消息。

Acknowledgements default to `AckExplicit` - the only supported mode for pull-based Consumers - meaning every message requires a distinct acknowledgement. But for push-based Consumers, you can set `AckNone` that does not require any acknowledgement, or `AckAll` which quite interestingly allows you to acknowledge a specific message, like message `100`, which will also acknowledge messages `1` through `99`. The `AckAll` mode can be a great performance boost.  
acknowledgement默认为AckExplicit，这是唯一支持的基于pull的消费者模式，这意味着每条消息都需要一个不同的确认。但是对基于push的消费者，你可以设置为AckNone不需要任何确认，或者设置为AckAll，非常有意思的是他可以让你确认特定的消息，比如消息100，他也会确认消息1到99。AckAll模式可以极大地提升性能。

Some messages may cause your applications to crash and cause a never ending loop forever poisoning your system. The `MaxDeliver` setting allow you to set an upper bound to how many times a message may be delivered.  
某些消息可能会导致您的应用程序崩溃并致使无限循环，长期损害您的系统。 MaxDeliver设置允许您设置一条消息可能被传递的次数的上限。

To assist with creating monitoring applications, one can set a `SampleFrequency` which is a percentage of messages for which the system should sample and create events. These events will include delivery counts and ack waits.  
为了帮助创建监控应用程序，可以设置一个 SampleFrequency，它是系统对其采样和创建事件的消息的百分比。这些事件将包括消息交付计数和确认等待。 
 
When defining Consumers the items below make up the entire configuration of the Consumer:  
在定义消费者时，以下选项构成了消费者的整个配置：

## AckPolicy

How messages should be acknowledged. If an ack is required but is not received within the AckWait window, the message will be redelivered.  
如何确认消息。如果ack是必需的，但是在AckWait窗口中没有收到，消息将被重新发送。

> IMPORTANT
>
> The server may consider an ack arriving out of the window. If a first process fails to ack within the window it's entirely possible, for instance in queue situation, that the message has been redelivered to another consumer. Since this will technically restart the window, the ack from the first consumer will be considered.  
服务器可能会认为一个ack抵达在窗口之外。例如在队列的情况下，如果一个进程未能在窗口内进行ack,则完全有可能将消息重新投递给另外一个消费者。由于技术上将重新启动窗口，因此将考虑来自第一个消费者的ack。  


### AckExplicit

This is the default policy. It means that each individual message must be acknowledged. It is the only allowed option for pull consumers.  
这是默认策略。这意味着每个消息都必须确认。这个设置仅允许pull消费者

### AckNone

You do not have to ack any messages, the server will assume ack on delivery.   
您不必确认任何消息，服务器将假定消息投递都已ack确认。

### AckAll

If you receive a series of messages, you only have to ack the last one you received. All the previous messages received are automatically acknowledged.  
如果您收到一系列消息，您只需确认您收到的最后一条消息。所有以前收到的消息都会自动确认。

## AckWait

Ack Wait is the time in nanoseconds that the server will wait for an ack for any individual message. If an ack is not received in time, the message will be redelivered.吧 
Ack Wait 是服务器等待任何单个消息的 ack 的时间（以纳秒为单位）。如果没有及时收到确认，则消息将被重新传递。

## DeliverPolicy / OptStartSeq / OptStartTime

When a consumer is first created, it can specify where in the stream it wants to start receiving messages. This is the `DeliverPolicy` and it's options are as follows:  
首次创建消费者时， DeliverPolicy可以指定要在流中的哪个位置开始接收消息。他的选项如下：

### DeliverAll

All is the default policy. The consumer will start receiving from the earliest available message.  
All是默认策略。消费者将从最早的可用消息开始接收。

### DeliverLast

When first consuming messages, the consumer will start receiving messages with the last message added to the stream, so the very last message in the stream when the server realizes the consumer is ready.  
当开始消费消息时，消费者将接收添加到流中的最后一条消息。（~~服务器意识到消费者已就绪时，消费者将开始接收以添加到流中的最后一条消息起的所有消息.~~)


### DeliverLastPerSubject

When first consuming messages, start with the latest one for each filtered subject currently in the stream.  
当开始消费消息时，从当前流中每个filtered subject的最新消息开始消费。

### DeliverNew

When first consuming messages, the consumer will only start receiving messages that were created after the consumer was created.  
当开始消费消息时，消费者只会接收在此消费者创建之后subject所创建的消息。

### DeliverByStartSequence

When first consuming messages, start at this particular message in the set. The consumer is required to specify `OptStartSeq`, the sequence number to start on. It will receive the closest available message moving forward in the sequence should the message specified have been removed based on the stream limit policy.  
当开始消费消息时，从集合中的这个特定消息开始。消费者需要指定OptStartSeq，即要指定开始消费的序列号。如果指定的消息已根据流限制策略删除，则它将接收序列中最接近的可用消息。

### DeliverByStartTime

When first consuming messages, start with messages on or after this time. The consumer is required to specify `OptStartTime`, the time in the stream to start at. It will receive the closest available message on or after that time.  
当开始消费消息时，从此时间点或之后的消息开始。消费者需要指定 OptStartTime，即流中开始的时间。它将接收在该时间点之后距离最近的可用消息。

## DeliverySubject

The subject to deliver observed messages. Not allowed for pull subscriptions. A delivery subject is required for queue subscribing as it configures a subject that all the queue consumers should listen on.  
发布观察消息的主题。不支持pull模式。订阅队列需要delivery subject，因为它配置了所有队列消费者都应该监听的主题。


## Durable \(Name\)

The name of the Consumer, which the server will track, allowing resuming consumption where left off. By default, a consumer is ephemeral. To make the consumer durable, set the name.  
服务器跟踪的消费者名称，允许从消息中断的地方恢复消费。默认情况下，消费者是临时的（消费结束后或者超时将无法从之前消费的位置继续消费）。为了使消费者长期有效，需要设置名称。

## FilterSubject

When consuming from a stream with a wildcard subject, this allows you to select a subset of the full wildcard subject to receive messages from.  
从带有通配符主题的流中消费时，允许您选择完整通配符主题的子集来接收消息。

## MaxAckPending

MaxAckPending implements a simple form of _one-to-many_ flow control. It sets the maximum number of messages without an acknowledgement that can be outstanding, once this limit is reached message delivery will be suspended. It cannot be used with AckNone ack policy. This maximum number of pending acks applies for _all_ of the consumer's subscriber processes. A value of -1 means there can be any number of pending acks (i.e. no flow control).   
MaxAckPending实现了一种简单的一对多流控制形式。它设置没有确认的消息的最大数量，一旦达到这个限制，消息传递将被暂停。它不能与AckNone ack策略一起使用。这个挂起的ack的最大数目适用于使用者的所有订阅者进程。值-1表示可以有任意数量的挂起ack(即没有流量控制)。

### Note about push and pull consumers: 

The MaxAckPending's one-to-many flow control functionality is only useful for push consumers. For pull consumers you should disable it (i.e. -1), as it can otherwise place a limit on the horizontal scalability of the processing of the stream. Because delivery of the messages to the client application through pull consumers is client demand-driven rather than server initiated, there is no need for any kind of one-to-many flow control. With pull consumers at a given point in time, the number of pending acks is a function of the number of client applications calling `fetch` on the pull consumer and the requested batch size for that operation.  
MaxAckPending 的一对多流控制功能仅对push模式消费者有用。对于pull模式的消费者，您应该禁用它（即-1），否则它会限制流处理的水平可伸缩性。因为基于pull模式的消费者，消息传递给客户端应用程序是客户端需求驱动的，而不是服务器发起的，所以不需要任何类型的一对多流控制。对于在给定时间点上的pull模式消费者，挂起ack的数量是一个函数，它由在pull消费者上调用fetch的客户端应用程序的数量和该操作请求的批处理大小组成。

## FlowControl

This flow control setting is to enable or not another form of flow control in parallel to MaxAckPending. But unlike MaxAckPending it is a _one-to-one_ flow control that operates independently for each individual subscriber to the consumer. It uses a sliding-window flow-control protocol whose attributes (e.g. size of the window) are _not_ user adjustable.  
FlowControl流控制可以配置启用/不启用，除MaxAckPending方式外的另一种流量控制形式。但与 MaxAckPending 不同的是，它是一对一的流量控制，为消费者的每个订阅独立运行。它使用滑动窗口流控制协议，其属性（例如窗口大小）不是用户可调整的。

## IdleHeartbeat

If the idle heartbeat period is set, the server will regularly send a status message to the client (i.e. when the period has elapsed) while there are no new messages to send. This lets the client know that the JetStream service is still up and running, even when there is no activity on the stream. The message status header will have a code of 100. Unlike FlowControl, it will have no reply to address. It may have a description like "Idle Heartbeat"  
如果设置了空闲心跳周期，当没有新的消息要发送时候，服务器将定期向客户端发送状态消息（即当周期已过时）。这让客户端知道 JetStream 服务仍在运行，即使流上没有任何活动。消息状态标头的代码为 100。与 FlowControl 不同，它不会回复地址。它可能有类似“Idle Heartbeat”的描述。

## MaxDeliver

The maximum number of times a specific message will be delivered. Applies to any message that is re-sent due to ack policy.  
投递特定消息的最大次数。适用于任何因ack策略而重新发送的消息。

## RateLimit

Used to throttle the delivery of messages to the consumer, in bits per second.  
用于限制向消费者传递消息的速度，单位为bit/s。

## ReplayPolicy

The replay policy applies when the DeliverPolicy is `DeliverAll`, `DeliverByStartSequence` or `DeliverByStartTime` since those deliver policies begin reading the stream at a position other than the end. If the policy is `ReplayOriginal`, the messages in the stream will be pushed to the client at the same rate that they were originally received, simulating the original timing of messages. If the policy is `ReplayInstant` \(the default\), the messages will be pushed to the client as fast as possible while adhering to the Ack Policy, Max Ack Pending and the client's ability to consume those messages.  
当DeliverPolicy为DeliverAll, DeliverByStartSequence或DeliverByStartTime时，重放策略适用，因为这些消息交付策略开始读取流的位置不是结束位置。如果策略是ReplayOriginal，流中的消息将以最初接收到的速率推到客户端，模拟消息的原始计时。如果策略是ReplayInstant(默认)，消息将被尽可能快地推送到客户端，同时遵循Ack策略、Max Ack Pending和客户端消费这些消息的能力。

## SampleFrequency

Sets the percentage of acknowledgements that should be sampled for observability, 0-100 This value is a string and for example allows both `30` and `30%` as valid values.  
设置应为可观察性采样的确认百分比，0-100 此值是一个字符串，例如允许 30% 和 30% 作为有效值。 

