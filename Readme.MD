## 消息队列 DH.NMQ
DH.NMQ 是一个轻量级消息队列，无延迟分发，支持消费组和消息去重，不支持集群部署。  

主要特点：  
+ 轻量级。服务端和客户端都是通过`nuget`引用`DH.NMQ`即可，也可以在Windows上直接安装发行版`DH.NMQServer`。  
+ 低延迟。默认不做持久化，发布消息后直接分发到目标队列，客户端长连接实时消费。（特别适用于跨系统异步通信）  
+ 消费者。支持广播消息给多个消费者。  
+ 消息延迟和去重。支持延迟消息，支持对重复发送的消息进行去重，降低消费成本。  

### 2019-02-20 v2.0混合测试通过  
1, 实现了基础的发送与消费  
2, 每个Topic下有一个队列，目前使用并行字典实现，自增消息ID为key，便于查找和删除
3, 每个消费组（相同登录用户）记录消费偏移量


```csharp
// 初始化客户端
var client = new MQClient
{
    Servers = new[] { "tcp://127.0.0.1:6789" },
    Log = XTrace.Log,

    Topic = "测试主题",
};

// 发布消息
var msgid = client.Public("发布测试").Result;
XTrace.WriteLine("msgid={0}", msgid);

// 消费消息
client.OnConsume = msgs =>
{
    foreach (var item in msgs)
    {
        XTrace.WriteLine("消费到 {0}", item);
    }
};
client.StartConsume();
```

主要术语：
+ `MQHost` 消息队列主机，管理多主题的订阅和发布  
+ `Topic` 主题队列，每个主题有一个队列用于缓冲保存消息，同时记录多个消费者  
+ `Consumer` 消费者，发布到主题的消息，会广播给各个消费者。多个订阅者构成消费者集群，消息只推送给其中一个订阅者。  
+ `Subscriber` 订阅者，使用相同消费者标识的不同来源，视为不同订阅者，比如同一个消费者账号的不同网络连接。  