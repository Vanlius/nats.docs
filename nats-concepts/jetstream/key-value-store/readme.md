# Key/Value store

JetSteam, the persistence layer of NATS, doesn't just allow for higher qualities of service and features associated with 'streaming', but it also enables some functionalities not found in messaging systems.  
One such feature is the Key/Value store functionality, which allows client applications to create 'buckets' and use them as immediately consistent, persistent [associative arrays](https://en.wikipedia.org/wiki/Associative_array).  

JetSteam, 是NATS的数据持久化层，不仅提供了更高质量的服务与“streaming”相关的功能，而且还支持一些消息传递系统中没有的功能。
其中一个功能是key/value存储功能，它允许客户端应用程序创建“buckets”，并能用它们做具有持久化的实时键值数据关联。

You can use KV buckets to perform the typical operations you would expect from an immediately consistent key/value store:  
* put: associate a value with a key  
* get: retrieve the value associated with a key  
* delete: clear any value associated with a key  
* purge: clear all the values associated with all keys  
* create: associate the value with a key only if there is currently no value associated with that key (i.e. compare to null and set)  
* update: compare and set (aka compare and swap) the value for a key
* keys: get a copy of all the keys (with a value or operation associated to it)  

使用KV buckets 做基本操作，你会立即得到一个key/value（键值对）存储
* put：将key与value关联，设置的一个key的值
* get：检索key所关联的值
* delete：清除key所关联的所有值
* purge：清除所有key关联的所有值
* create：创建key/value，只有当前不存在的key才将其值与之关联  
* update：比较并设置（比较并交换）key的值，更新key的值
* keys：获取所有keys的副本(带有与之关联的值或操作)

You can set limits for your buckets, such as:
* the maximum size of the bucket
* the maximum size for any single value
* a TTL: how long the store will keep values for  

你可以为你的buckets设置一些限制，例如:
* bucket的最大大小
* 任意单个值的最大大小
* TTL:值在存储（持久化层）上将保留多久时间

You finally, you can even also do things that you typically can not do with a Key/Value Store:
* watch: watch for changes the changes happening for a key, which is similar to subscribing (in the publish/subscribe sense) to the key: the watcher receives updates due to put or delete operations on the key pushed to it in real-time as they happen
* watch all: watch for all the changes happening on all the keys in the bucket 
* history: retrieve a history of the values (and delete operations) associated with each key over time (by default the history of buckets is set to 1, meaning that only the latest value/operation is stored)
  
最后，你可以做一些通常不能用key/value存储做到的事情:
* watch:监控一个key发生的变更，这类似于订阅一个key(发布/订阅者模式):当key发生变化时，观察者会实时接收key在put或delete操作时更新的值。（类似触发器）
* watch all:监控bucket中所有key上发生的所有变更
* history:检索指定时间段内与每个key相关的value(包含delete操作)的历史记录 (默认情况下，buckets的历史记录被设置为1，这意味着只存储最新的值/操作历史)