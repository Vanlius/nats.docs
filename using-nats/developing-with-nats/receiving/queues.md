# Queue Subscriptions

Subscribing to a [queue group](../../../nats-concepts/core-nats/queue-groups/queue.md) is only slightly different than subscribing to a subject alone. The application simply includes a queue name with the subscription. The server will load balance between all members of the queue group. In a cluster setup, every member has the same chance of receiving a particular message.  
Keep in mind that queue groups in NATS are dynamic and do not require any server configuration.  
订阅队列组与单独订阅主题略有不同。应用程序只需要在订阅中包含队列名称。服务器将在队列组的所有成员之间实现负载平衡。在集群设置中，每个成员都有相同的机会接收特定的消息。  
请记住，NATS中的队列组是动态的，不需要任何服务器配置。

![](../../../.gitbook/assets/queues.svg)

As an example, to subscribe to the queue `workers` with the subject `updates`:

{% tabs %}
{% tab title="Go" %}
```go
nc, err := nats.Connect("demo.nats.io")
if err != nil {
    log.Fatal(err)
}
defer nc.Close()

// Use a WaitGroup to wait for 10 messages to arrive
wg := sync.WaitGroup{}
wg.Add(10)

// Create a queue subscription on "updates" with queue name "workers"
if _, err := nc.QueueSubscribe("updates", "workers", func(m *nats.Msg) {
    wg.Done()
}); err != nil {
    log.Fatal(err)
}

// Wait for messages to come in
wg.Wait()
```
{% endtab %}

{% tab title="Java" %}
```java
Connection nc = Nats.connect("nats://demo.nats.io:4222");

// Use a latch to wait for 10 messages to arrive
CountDownLatch latch = new CountDownLatch(10);

// Create a dispatcher and inline message handler
Dispatcher d = nc.createDispatcher((msg) -> {
    String str = new String(msg.getData(), StandardCharsets.UTF_8);
    System.out.println(str);
    latch.countDown();
});

// Subscribe
d.subscribe("updates", "workers");

// Wait for a message to come in
latch.await(); 

// Close the connection
nc.close();
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
nc.subscribe(subj, {
    queue: "workers",
    callback: (_err, _msg) => {
      t.log("worker1 got message");
    },
});

nc.subscribe(subj, {
    queue: "workers",
    callback: (_err, _msg) => {
      t.log("worker2 got message");
    },
});
```
{% endtab %}

{% tab title="Python" %}
```python
nc = NATS()

await nc.connect(servers=["nats://demo.nats.io:4222"])

future = asyncio.Future()

async def cb(msg):
  nonlocal future
  future.set_result(msg)

await nc.subscribe("updates", queue="workers", cb=cb)
await nc.publish("updates", b'All is Well')

msg = await asyncio.wait_for(future, 1)
print("Msg", msg)
```
{% endtab %}

{% tab title="Ruby" %}
```ruby
require 'nats/client'
require 'fiber'

NATS.start(servers:["nats://127.0.0.1:4222"]) do |nc|
  Fiber.new do
    f = Fiber.current

    nc.subscribe("updates", queue: "worker") do |msg, reply|
      f.resume Time.now
    end

    nc.publish("updates", "A")

    # Use the response
    msg = Fiber.yield
    puts "Msg: #{msg}"
  end.resume
end
```
{% endtab %}

{% tab title="C" %}
```c
static void
onMsg(natsConnection *conn, natsSubscription *sub, natsMsg *msg, void *closure)
{
    printf("Received msg: %s - %.*s\n",
           natsMsg_GetSubject(msg),
           natsMsg_GetDataLength(msg),
           natsMsg_GetData(msg));

    // Need to destroy the message!
    natsMsg_Destroy(msg);
}


(...)

natsConnection      *conn = NULL;
natsSubscription    *sub  = NULL;
natsStatus          s;

s = natsConnection_ConnectTo(&conn, NATS_DEFAULT_URL);

// Create a queue subscription on "updates" with queue name "workers"
if (s == NATS_OK)
    s = natsConnection_QueueSubscribe(&sub, conn, "updates", "workers", onMsg, NULL);

(...)


// Destroy objects that were created
natsSubscription_Destroy(sub);
natsConnection_Destroy(conn);
```
{% endtab %}
{% endtabs %}

If you run this example with the publish examples that send to `updates`, you will see that one of the instances gets a message while the others you run won't. But the instance that receives the message will change.  
如果您使用发送更新的发布示例运行此示例，您将看到其中一个实例获得消息，而您运行的其他实例没有。但是接收消息的实例将改变。

