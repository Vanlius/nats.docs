# Example

参考以下架构

![Orders](<../../.gitbook/assets/streams-and-consumers-75p (1).png>)

虽然它不是一个完整的架构，但确实展示了许多关键特征： 

* 多个主题都存储在一个stream中
* 消费者以不同的消费模式接收消息的子集（ORDER.*） 
* 支持多种消息确认（ack）方式  

一条新订单消息到达`ORDERS.received`主题，被消费者`NEW`成功处理后，再发送订单消息到`ORDERS.processed`主题，消费者`DISPATCH`接收处理`ORDERS.processed` 上的订单消息，依次将订单完成消息发送到`ORDERS.completed`。这些操作都是基于`pull`模式的，也就意味着工作队列是能水平扩展的。所有的订单消息都需要做确认（ack），以确保不丢失任何订单消息。  

使用Pub/Sub语义和不做任何消息交付确认（ack）将所有的订单消息投递到`MONITOR`消费者，它们被推送到用户（监视器）。    

当消息被 `NEW` 和 `DISPATCH` 消费者确认时，其中将重新传递计数、确认延迟等消息的百分比采样数据发送到监控系统。  

## Example Configuration

[附加文档](../clustering/administration.md)介绍了nats实用工具，以及如何使用它来创建、监控和管理stream和消费者，但为了完整和参考，这是创建ORDERS场景的方法。我们将为订单相关信息配置1年保留:  
```bash
nats stream add ORDERS --subjects "ORDERS.*" --ack --max-msgs=-1 --max-bytes=-1 --max-age=1y --storage file --retention limits --max-msg-size=-1 --discard=old
nats consumer add ORDERS NEW --filter ORDERS.received --ack explicit --pull --deliver all --max-deliver=-1 --sample 100
nats consumer add ORDERS DISPATCH --filter ORDERS.processed --ack explicit --pull --deliver all --max-deliver=-1 --sample 100
nats consumer add ORDERS MONITOR --filter '' --ack none --target monitor.ORDERS --deliver last --replay instant
```
