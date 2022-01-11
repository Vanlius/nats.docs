# Receiving Messages

In general, applications can receive messages asynchronously or synchronously. Receiving messages with NATS can be library dependent.

Some languages, like Go or Java, provide synchronous and asynchronous APIs, while others may only support one type of subscription.

In all cases, the process of subscribing involves having the client library tell the NATS system that an application is interested in a particular subject. When an application is done with a subscription it unsubscribes telling the server to stop sending messages.

A client will receive a message for each matching subscription, so if a connection has multiple subscriptions using identical or overlapping subjects \(say `foo` and `>`\) the same message will be sent to the client multiple times. 

通常情况下，应用程序可以异步或同步接收消息。接收NATS消息取决于依赖库。  
有些语言，如Go或Java，提供同步和异步api，而其他语言可能只支持一种类型的订阅。    
在所有情况下，订阅过程都涉及客户端应用程序订阅NATS系统中感兴趣的特定主题，当应用程序订阅完成后，它会取消订阅，并通知服务器停止发送消息。  
客户端将收到每个匹配订阅的消息，因此，如果一个连接有多个订阅，使用相同或重叠的主题(比如foo和>)，相同的消息将被多次发送给客户端。 

