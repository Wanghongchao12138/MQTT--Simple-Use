# MQTT--Simple-Use
iOS MQTT 简单使用流程
###  简称为EMQ - 百万级开源MQTT消息服务器
***EMQ*** (Erlang/Enterprise/Elastic MQTT Broker) 是基于 Erlang/OTP 平台开发的开源物联网 MQTT 消息服务器。Erlang/OTP 是出色的软实时(Soft-Realtime)、低延时(Low-Latency)、分布式(Distributed) 的语言平台。MQTT 是轻量的(Lightweight)、发布订阅模式(PubSub) 的物联网消息协议。

EMQ 项目设计目标是承载移动终端或物联网终端海量 MQTT 连接，并实现在海量物联网设备间快速低延时消息路由:
 1. 稳定承载大规模的 MQTT 客户端连接，单服务器节点支持50万到100万连接。
 2. 分布式节点集群，快速低延时的消息路由，单集群支持1000万规模的路由。
 3. 消息服务器内扩展，支持定制多种认证方式、高效存储消息到后端数据库。
 4. 完整物联网协议支持，MQTT、MQTT-SN、CoAP、WebSocket 或私有协议支持。

## 选择 MQTT  SDK 分为多种 
以下介绍其中的两种 [MQTTKit](https://github.com/jmesnil/MQTTKit) 和 [MQTT-Client-Framework](https://github.com/ckrey/MQTT-Client-Framework)  	
这两种都是OC 使用  Swift 版本可参考  [CocoaMQTT](https://github.com/emqtt/CocoaMQTT)`
1、 ** MQTTKit **  `已经不更新 但是基本使用没问题`
> pod 'MQTTKit'    

***头文件***
> #import <MQTTKit.h>
 #define WEAKSELF   __typeof(&*self) __weak weakSelf = self;
@property (nonatomic, strong) MQTTClient *client;


### 初始化 链接
```
	WEAKSELF
    NSString *clientID = @"测试client - 必须是全局唯一的id ";
    MQTTClient *client = [[MQTTClient alloc] initWithClientId:StrFormat(@"%@", clientID)];
    client.username = @"username";
    client.password = @"password";
    client.cleanSession = false;
    client.keepAlive = 20;
    client.port = 11883;// 端口号 根据服务端 选择
    self.client = client;
    // 链接MQTT
    [client connectToHost:@"链接的MQTT的URL" completionHandler:^(MQTTConnectionReturnCode code) {
        if (code == ConnectionAccepted) {
            NSLog(@"链接MQTT 成功 😁😁😁");
            // 链接成功  订阅相对应的主题
        [weakSelf.client subscribe:@"你需要订阅的主题" withQos:AtLeastOnce completionHandler:^(NSArray *grantedQos) {
            DLog(@"订阅 返回 %@",grantedQos);
        }];
        }else if (code == ConnectionRefusedBadUserNameOrPassword){
            NSLog(@"MQTT 账号或验证码错误");
        } else if (code == ConnectionRefusedUnacceptableProtocolVersion){
            NSLog(@"MQTT 不可接受的协议");
        }else if (code == ConnectionRefusedIdentiferRejected){
            NSLog(@"MQTT不认可");
        }else if (code == ConnectionRefusedServerUnavailable){
            NSLog(@"MQTT拒绝链接");
        }else {
            NSLog(@"MQTT 未授权");
        }
    }];
 // 接收消息体
    client.messageHandler = ^(MQTTMessage *message) {
        NSString *jsonStr = [[NSString alloc] initWithData:message.payload encoding:NSUTF8StringEncoding];
        NSLog(@"EasyMqttService mqtt connect success  %@",jsonStr);
    };
```
### 订阅主题
```
// 方法 封装 可外部调用
-(void)subscribeType:(NSString *)example{
    // 订阅主题
    [self.client subscribe:@"你需要订阅的主题" withQos:AtMostOnce completionHandler:^(NSArray *grantedQos) {
        NSLog(@"订阅 返回 %@",grantedQos);
    }];
}
```
### 关闭MQTTKit
```
-(void)closeMQTTClient{
    WEAKSELF
    [self.client disconnectWithCompletionHandler:^(NSUInteger code) {
        // The client is disconnected when this completion handler is called
        NSLog(@"MQTT client is disconnected");
        [weakSelf.client unsubscribe:@"已经订阅的主题" withCompletionHandler:^{
            NSLog(@"取消订阅");
        }];
    }];
}
```
### 发送消息
```
   [self.client publishString:postMsg toTopic:@"发送消息的主题 根据服务端定"  withQos:AtLeastOnce retain:NO completionHandler:^(int mid) {
        if (cmd != METHOD_SOCKET_CHAT_TO) {
            NSLog(@"发送消息 返回 %d",mid);
        }
    }];
```

2、 ** MQTTClient **   `MQTTClient 配置更多 是可持续更新，可配置 SSL `
> `基本使用`
pod 'MQTTClient'
 `websocket` 方式连接
pod 'MQTTClient/MinL'
pod 'MQTTClient/ManagerL'
pod 'MQTTClient/WebsocketL'
    

***头文件***
>`基本使用`
 #import <MQTTClient.h>
 `websocket` 需要添加的头文件
#import <MQTTWebsocketTransport.h>
 #define WEAKSELF   __typeof(&*self) __weak weakSelf = self;
@property (nonatomic, strong) MQTTSession *mySession;
`需要添加协议头`
<MQTTSessionDelegate,MQTTSessionManagerDelegate>

### 初始化 链接
`基本使用`
```

#import "MQTTClient.h"
\@interface MyDelegate : ... <MQTTSessionDelegate>
...
        MQTTCFSocketTransport *transport = [[MQTTCFSocketTransport alloc] init];
        transport.host = @"localhost";
        transport.port = 1883;
        MQTTSession *session = [[MQTTSession alloc] init];
        session.transport = transport;
    	session.delegate = self;
    	[session connectAndWaitTimeout:30];  //this is part of the synchronous API
```
`websocket 连接` 
```
	WEAKSELF
    NSString *clientID = @"测试client - 必须是全局唯一的id ";
    MQTTWebsocketTransport *transport = [[MQTTWebsocketTransport alloc] init];
    transport.host = @"连接MQTT 地址";
    transport.port =  8083;  // 端口号
    transport.tls = YES; //  根据需要配置  YES 开起 SSL 验证 此处为单向验证 双向验证 根据SDK 提供方法直接添加
    MQTTSession *session = [[MQTTSession alloc] init];
    NSString *linkUserName = @"username";
    NSString *linkPassWord = @"password";
    [session setUserName:linkUserName];
    [session setClientId:clientID];
    [session setPassword:linkPassWord];
    [session setKeepAliveInterval:5];
    session.transport = transport;
    session.delegate = self;
    self.mySession = session;
    [self reconnect];
    [self.mySession addObserver:self forKeyPath:@"state" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:nil]; //添加事件监听
```
### `websocket` 监听 响应事件
```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    switch (self.mySession.status) {
        case MQTTSessionManagerStateClosed:
        NSLog(@"连接已经关闭");
        break;
        case MQTTSessionManagerStateClosing:
        NSLog(@"连接正在关闭");
        break;
        case MQTTSessionManagerStateConnected:
        NSLog(@"已经连接");
        break;
        case MQTTSessionManagerStateConnecting:
        NSLog(@"正在连接中");
        
        break;
        case MQTTSessionManagerStateError: {
            //            NSString *errorCode = self.mySession.lastErrorCode.localizedDescription;
            NSString *errorCode = self.mySession.description;
            NSLog(@"连接异常 ----- %@",errorCode);
        }
        break;
        case MQTTSessionManagerStateStarting:
        NSLog(@"开始连接");
        break;
        default:
        break;
    }
}
```
### `session Delegate 协议` 
`连接 返回状态`
```
-(void)handleEvent:(MQTTSession *)session event:(MQTTSessionEvent)eventCode error:(NSError *)error{
    if (eventCode == MQTTSessionEventConnected) {
        NSLog(@"2222222 链接MQTT 成功");
    }else if (eventCode == MQTTSessionEventConnectionRefused) {
            NSLog(@"MQTT拒绝链接");
   }else if (eventCode == MQTTSessionEventConnectionClosed){
            NSLog(@"MQTT链接关闭");
  }else if (eventCode == MQTTSessionEventConnectionError){
            NSLog(@"MQTT 链接错误");
  }else if (eventCode == MQTTSessionEventProtocolError){
            NSLog(@"MQTT 不可接受的协议");
  }else{//MQTTSessionEventConnectionClosedByBroker
            NSLog(@"MQTT链接 其他错误");
  }
   if (error) {
        NSLog(@"链接报错  -- %@",error);
   }
}
```
`收到发送的消息`
```
-(void)newMessage:(MQTTSession *)session data:(NSData *)data onTopic:(NSString *)topic qos:(MQTTQosLevel)qos retained:(BOOL)retained mid:(unsigned int)mid
{
    NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];
    NSLog(@"EasyMqttService mqtt connect success  %@",dic);
    // 做相对应的操作
}
```
### 订阅主题
`基本使用`
```
// 方法 封装 可外部调用
[session subscribeToTopic:@"example/#" atLevel:2 subscribeHandler:^(NSError *error, NSArray<NSNumber *> *gQoss){
    if (error) {
        NSLog(@"Subscription failed %@", error.localizedDescription);
    } else {
        NSLog(@"Subscription sucessfull! Granted Qos: %@", gQoss);
    }
 }]; // this is part of the block API
```
`websocket`
```
- (void)subscibeToTopicAction {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_async(dispatch_get_main_queue(), ^{
            [self subscibeToTopic:@"你要订阅的主题"];
            
        });
    });
}

-(void)subscibeToTopic:(NSString *)topicUrl
{
//    self.manager.subscriptions = [NSDictionary dictionaryWithObject:[NSNumber numberWithInt:MQTTQosLevelAtMostOnce] forKey:topicUrl];
    [self.mySession subscribeToTopic:topicUrl atLevel:MQTTQosLevelAtMostOnce subscribeHandler:^(NSError *error, NSArray<NSNumber *> *gQoss) {
        if (error) {
            NSLog(@"订阅 %@ 失败 原因 %@",topicUrl,error);
        }else
        {
            NSLog(@"订阅 %@ 成功 g1oss %@",topicUrl,gQoss);
            dispatch_async(dispatch_get_main_queue(), ^{
                // 操作
            });

        };
    }];
}
```


### 关闭MQTT-Client
```
-(void)closeMQTTClient{
    [self.mySession disconnect];
    [self.mySession unsubscribeTopics:@[@"已经订阅的主题"] unsubscribeHandler:^(NSError *error) {
        if (error) {
            DLog(@"取消订阅失败");
        }else{
            DLog(@"取消订阅成功");
        }
    }];
}
```
### 发送消息
```
    [self.mySession publishData:jsonData onTopic:@"发送消息的主题 服务端定义" retain:NO qos:MQTTQosLevelAtMostOnce publishHandler:^(NSError *error) {
        if (error) {
            NSLog(@"发送失败 - %@",error);
        }else{
            NSLog(@"发送成功");
        }
    }];
```

