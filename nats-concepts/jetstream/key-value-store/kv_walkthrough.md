# NATS Key/Value Store Walkthrough

## Prerequisite: enabling JetStream

如果你正在运行一个本地的`nats-server`，停止它并使用`nats-server-js`重启以开启JetStream功能的支持 (如果还没有完成)

然后你应该检查是否启用了JetStream  
```shell
nats account info
```
应该输出如下内容：  
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

## Creating a KV bucket

就像你必须先创建流才能使用它们一样，您首先需要使用`nats kv add <KV Bucket Name>` 创建一个`KV bucket`：  
```shell
nats kv add my_kv
```
输出如下：
```text
my_kv Key-Value Store Status

         Bucket Name: my_kv
        History Kept: 1
 Maximum Bucket Size: unlimited
  Maximum Value Size: unlimited
     Bucket Location: unknown
       Values Stored: 0
  Backing Store Name: KV_my_kv
```

## Storing a value

现在有了一个bucket，我们可以用'put'(存储)来存储一个值到key:  

```shell
nats kv put my_kv Key1 Value1
```

应该返回`Value1`  
## Getting a value

现在，我们已经在“Key1”中存储了值，我们可以使用“get”来获取该值:  

```shell
nats kv get my_kv Key1
```
输出如下：  
```
my_kv > Key1 created @ 12 Oct 21 20:08 UTC

Value1

```

## Deleting a value

你可以通过使用`nats kv del my_kv Key1`来删除一个Key/Value键值对。  

## Watching a K/V Store

NATS KV Store提供了一种功能(通常key/value存储不提供)，可以“监控”一个bucket(或bucket中的指定的key)，并接收实时更新在存储中的更改。  

例如，运行`nats kv watch my_kv`:在我们刚刚创建的bucket上启动一个监控（观察者）。如果你按照此例子运行，则该key上发生的最后操作是该key已被删除。 因为默认情况下，KV bucket的历史大小设置为1(即只保留最后一次更改)，并且bucket上的最后一次操作是删除“Key1”的值，这是监控（观察者）唯一收到的操作。  
```shell
nats kv watch my_kv
```
输出如下:  
```
[2021-10-12 13:15:03] DEL my_kv > Key1
```

保持`nats kv watch`运行并到另一个窗口对my_key做`put`操作


```shell
nats kv put my_kv Key1 Value2
```

以上操作被执行后，你会得到观察者接收的put事件:  

```shell
[2021-10-12 13:25:14] PUT my_kv > Key1: Value2
```

## Cleaning up

以上操作执行完成后，你可以使用以下命令轻松的删除KV bucket，并释放与其关联的资源：
```shell
nats kv rm my_kv
```