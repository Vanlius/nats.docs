# NATS JetStream Walkthrough

## Prerequisite: enabling JetStream

如果你正在运行一个本地的nats-server，停止它并使用nats-server-js重启以开启JetStream功能的支持 (如果还没有完成)  

然后你应该检查是否启用了JetStream   

```shell
nats account info
```
输出如下：  
```
Connection Information:

               Client ID: 6
               Client IP: 127.0.0.1
                     RTT: 64.996µs
       Headers Supported: true
         Maximum Payload: 1.0 MiB
           Connected URL: nats://127.0.0.1:4222
       Connected Address: 127.0.0.1:4222
     Connected Server ID: ND2XVDA4Q363JOIFKJTPZW3ZKZCANH7NJI4EJMFSSPTRXDBFG4M4C34K

JetStream Account Information:

           Memory: 0 B of Unlimited
          Storage: 0 B of Unlimited
          Streams: 0 of Unlimited
        Consumers: 0 of Unlimited 
```

如果你看到的是以下内容，则未启用JetStream  

```text
JetStream Account Information:

   JetStream is not supported in this account
```

## 1. Creating a stream
 
首先创建一个stream来记录和存储发布在“foo”主题上的消息。  
输入`nats stream add < stream name>`(在下面的例子中，我们将stream命名为“my_stream”)，然后输入“foo”作为主题名，然后按回车键使用stream所有其他属性的默认值:  
```shell
nats stream add my_stream
```
example output
```text
? Subjects to consume foo
? Storage backend file
? Retention Policy Limits
? Discard Policy Old
? Stream Messages Limit -1
? Per Subject Messages Limit -1
? Message size limit -1
? Maximum message age limit -1
? Maximum individual message size -1
? Duplicate tracking time window 2m
? Replicas 1
Stream my_stream was created

Information for Stream my_stream created 2021-10-12T08:42:10-07:00

Configuration:

             Subjects: foo
     Acknowledgements: true
            Retention: File - Limits
             Replicas: 1
       Discard Policy: Old
     Duplicate Window: 2m0s
     Maximum Messages: unlimited
        Maximum Bytes: unlimited
          Maximum Age: 0.00s
 Maximum Message Size: unlimited
    Maximum Consumers: unlimited


State:

             Messages: 0
                Bytes: 0 B
             FirstSeq: 0
              LastSeq: 0
     Active Consumers: 0
```

然后你可以检查你刚刚创建的stream信息:  
```shell
nats stream info my_stream
```
which should output something like
```text
Information for Stream my_stream created 2021-10-12T08:42:10-07:00

Configuration:

             Subjects: foo
     Acknowledgements: true
            Retention: File - Limits
             Replicas: 1
       Discard Policy: Old
     Duplicate Window: 2m0s
     Maximum Messages: unlimited
        Maximum Bytes: unlimited
          Maximum Age: 0.00s
 Maximum Message Size: unlimited
    Maximum Consumers: unlimited


State:

             Messages: 0
                Bytes: 0 B
             FirstSeq: 0
              LastSeq: 0
     Active Consumers: 0
```

## 2. Publish some messages into the stream

现在我们创建一个发布者  
```shell
nats pub foo --count=1000 --sleep 1s "publication #{{Count}} @ {{TimeStamp}}"
```

当消息被发布在“foo”主题上时，它们也被记录并存储在流中，你可以通过使用“nats stream info my_stream”来检查，甚至可以使用“nats stream view my_stream”来查看消息本身。
## 3. Creating a consumer

如果现在你创建一个“Core NATS”(即非流式传输)订阅者来监听“foo”主题上的消息，你只会收到订阅者在启动后发布的消息，这对基于“core NATS”的消息传递是正常的且在预期内。为了接收流中所有消息（包括过去发布的消息）的“重放”，我们现在将创建一个“消费者”    

我们可以使用“nats consumer add”命令以管理方式创建消费者，在本例中，我们将消费者命名为“pull_consumer”，并将交付主题设为“nothing”（即在提示符处按回车键）， 因为我们创建的是一个“pull消费者”，所以start policy选择all，然后其他所有的选项你可以使用回车键设置为默认值。创建消费者的流应该是我们刚刚在上面创建的流'my_stream'。  

```shell
nats consumer add
```
example output
```text
? Consumer name pull_consumer
? Delivery target (empty for Pull Consumers)
? Start policy (all, new, last, subject, 1h, msg sequence) all
? Replay policy instant
? Filter Stream by subject (blank for all)
? Maximum Allowed Deliveries -1
? Maximum Acknowledgements Pending 0
? Select a Stream my_stream
Information for Consumer my_stream > pull_consumer created 2021-10-12T09:03:26-07:00

Configuration:

        Durable Name: pull_consumer
           Pull Mode: true
      Deliver Policy: All
 Deliver Queue Group: _unset_
          Ack Policy: Explicit
            Ack Wait: 30s
       Replay Policy: Instant
     Max Ack Pending: 20,000
   Max Waiting Pulls: 512

State:

   Last Delivered Message: Consumer sequence: 0 Stream sequence: 0
     Acknowledgment floor: Consumer sequence: 0 Stream sequence: 0
         Outstanding Acks: 0 out of maximum 20000
     Redelivered Messages: 0
     Unprocessed Messages: 674
            Waiting Pulls: 0 of maximum 512
```

您随时可以使用 `nats consumer info` 查看任意消费者的状态，或使用 `nats stream my_stream` 查看stream的相关消息，甚至使用`nats stream rmm` 从stream中删除单条消息  
## 3. Subscribing from the consumer

现在已经创建了消费者，且stream中有消息，我们可以开始用消费者订阅消息：  
```shell
nats consumer next my_stream pull_consumer --count 1000
```
从第一条消息（过去发布）开始打印流中的所有消息，并在新消息发布时继续打印，直到达到指定的消费总数。

请注意，在此示例中，我们正在创建一个具有“durable”名称的pull消费者，这意味着消费者可以在任意数量的消费进程之间共享。例如，与消费总数设置为1000的单个nats消费者不同，你可以启动两个nats消费者实例，每个实例的消息总数设置为500，这些消费者的消息分布在这些nats实例之间。  

#### Replaying the messages again

一旦你使用消费者消费完流中的所有消息，可以通过创建一个新消费者或删除该消费者（`nats consumer rm`）并重新创建它（`nats consumer add`）来再次获取流中的数据（replay）。

## 4. Cleaning up

您可以使用`nats stream purge` 清理stream（并释放与其关联的资源（例如存储在流中的消息））

您还可以使用`nats stream rm` 删除stream（这也将自动删除可能在该流上定义的所有消费者)
