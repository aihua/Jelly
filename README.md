# Jelly
---
### Jelly是一款基于Netty4.x开发的即时通讯服务器端程序；并且提供了Java客户端API。

## 功能包括
- 账户：登录、注册、登出
- 好友：添加、删除、好友在线状态
- 消息：个人消息、讨论组消息(支持离线消息)
- 讨论组：创建和解散讨论组、添加和删除成员
- 个人信息：修改个人信息、查看个人信息

## Architecture
### 模块介绍
- jelly-launcher &#12288; 启动模块(就一个类而已)
- jelly-transport &#12288; 数据传输模块
- jelly-serialization &#12288; 序列化模块
- jelly-service &#12288; 服务模块
- jelly-dao &#12288; 数据访问模块

### 应用层协议
```
                                        Jelly Protocol
    __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __
   |           |           |           |           |              |                          |
         2           1           1           1            4               Uncertainty
   |__ __ __ __|__ __ __ __|__ __ __ __|__ __ __ __|__ __ __ __ __|__ __ __ __ __ __ __ __ __|
   |           |           |           |           |              |                          |
       Magic        Sign        Type       Status     Body Length         Body Content
   |__ __ __ __|__ __ __ __|__ __ __ __|__ __ __ __|__ __ __ __ __|__ __ __ __ __ __ __ __ __|
 
   协议头9个字节定长
       Magic      // 数据包的验证位，short类型
       Sign       // 消息标志，请求／响应／通知，byte类型
       Type       // 消息类型，登录／发送消息等，byte类型
       Status     // 响应状态，成功／失败，byte类型
       BodyLength // 协议体长度，int类型
```
*数据交换格式：JSON（框架：Gson）*

### 工作流程
```
              --------> Request        - - - - -> Response       -- -- --> Notice                    
---------------------------------------------------------------------------------------------------- 
                                                                                                     
                                                                                                     
                             __ __ __ __ __ __ __ __ __ __ __ __ __ __                               
                            |                  Server                 |                              
      __ __ __ __           |  __ __ __ __ __         __ __ __ __ __  |            __ __ __ __       
     |           | Request  | |              |       |              | |  Notice   |           |      
     |   Client  |--------> | | BlockingQueue| ----> |  ThreadPool  | | -- -- --> |   Client  |      
     |__ __ __ __|          | |__ __ __ __ __|       |__ __ __ __ __| |           |__ __ __ __|      
           |                |                                |        |                              
           |                |__ __ __ __ __ __ __ __ __ __ __|__ __ __|                              
           |                                                 |                                       
           |                    Response                     |                                       
            <- - - - - - - - - - - - - - - - - - - - - - - -                                         
                                                                                                     
                                                                                                     
---------------------------------------------------------------------------------------------------- 
```

### 其它说明
#### 1. 登录成功后
- 服务器端登录信息验证成功后生成Long类型的Token返回给客户端，此Token用于断线重连的验证信息
- 开启心跳检测，客户端每空闲5s发送一个心跳包，服务器端每空闲6s计一次心跳失败
- username和channel维护在一个Map集合中

#### 2. 断线重连
- 使用Token尝试重连一次

#### 3. 讨论组信息
- 为了减小内存压力，，在Server启动时会开启一个定时任务，每隔五分钟检查一次groupMap（保存讨论组信息的Map集合），最后一次活跃时刻过去超过10分钟的讨论组被从内存中remove掉；直到下一次活跃时刻才会被调入内存（活跃就是组员发消息）

### 客户端API
提供的都有API都是异步的，调用之后会返回一个Future，使用该Future添加相应的监听器来得到的服务器的响应结果。
<p>
**API详细说明：[JellyAPI](https://github.com/Yohann-Codes/JellyAPI)**