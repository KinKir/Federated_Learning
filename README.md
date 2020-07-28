# 在自己的电脑上运行此项目

- FedClient 为该项目的客户端，是 Android 类型的项目，使用Android Studio打开
- FedServer 为该项目的服务端，使用 Intellij IDEA 打开此项目

此项目目前通过了本地运行测试，WebSocket使用的是内网进行测试

在本地运行项目前需修改Android项目中的ws地址，将

![image-20200728142745278](https://github.com/zhanghad/Federated_Learning-Project/tree/master/image/image-20200728142745278.png)

```
ws://192.168.1.4:8887/
```

改为

```
ws://[本机内网ip]:8887/
```



# 通信协议

## 网络应用层协议选择

**HTTP or Websocket ？**

### 相同点：

- 均为应用层协议
- 均为基于TCP协议的可靠通信协议



### 不同点：

**HTTP：**

- ​	短连接：每一次通信均需要进行三次握手
- ​	长连接：一次三次握手建立连接，建立后每次请求客户端只需要发送一个request请求	

特点：

- ​	请求只能由客户端主动发起，服务器被动接收请求并做出反应，通信只可以单向进行
- ​	实时性弱（要实现实时通信，可采用轮询的方式，开销较大）
- ​	每条信息均有HTTP的包头，产生额外的通信开销



**WebSocket：**

首先客户端通过一次HTTP请求与服务器建立一个持久性连接，之后的通信直接通过TCP连接进行。

特点：

- 全双工通信，通信可双向进行，服务端可主动向客户端推送消息
- 实时性强
- 建立连接后，通信没有额外的包头，减少了冗余信息，通信开销低
- 可以自定义通信协议，设计数据包格式
- 对网络状况要求高，需对**断连情况**设计好处理方法
- WebSocket默认最大并发数为200，可修改



**暂定 WebSocket**



## 通信协议设计

《Towards Federated Learning at Scale: System Design》

![image-20200727145650894](https://github.com/zhanghad/Federated_Learning-Project/tree/master/image/image-20200727145650894.png)

**联邦学习中客户端与服务端的通信**



![image-20200728122706459](https://github.com/zhanghad/Federated_Learning-Project/tree/master/image/image-20200728122706459.png)

## start

开始联邦学习任务



## Select

**选择哪些client加入新一轮训练？**（**clients选择算法**）

选择依据：

1. client评价：
   - client设备的状态（CPU，存储，网络状况）
   - 历史参与信息
   - 其他反映client价值的信息
2. 训练计划
   - 选择合适数量的client参与一轮训练

根据计划，达到指定clients数量时进行下一阶段。



## Config

在配置阶段，server会向selection阶段选择的clients发送相关的信息，包括

**发送哪些配置信息？**（**数据结构设计**）

- 全局模型（梯度信息，配置信息）
- 训练计划



## Report

client在训练完之后向server上传更新模型（**安全，隐私**），server对所有更新梯度进行聚合（**聚合算法**）

server在收到本轮第一个更新的梯度后开始计时，超时的梯度将不再接收。

约束：

- 完成report的clients数目达到最低限制
- 在规定时间内完成

如果规定时间内未接收到最低数量的更新的梯度则本轮训练失败，将会重新开始本轮训练。



## end

成功完成规定轮数后 server 会更新全局模型，生成最后的训练结果。

# 系统架构

![image-20200728122438323](https://github.com/zhanghad/Federated_Learning-Project/tree/master/image/image-20200728122438323.png)

## 移动端

- Client：app主进程

- 模型库：训练任务的存储库，存放各种任务的模型，当需要执行时，主进程创建一个子线程调用模块中已有的模型执行训练任务。（该模块初期实验只实现一个模型，后期可以添加其他训练模型）

  在执行训练任务前，训练模块会检测当前手机状态，并会有一定**运行限制**（比如手机处于空闲状态，连接WIFI，正在充电）

- 数据管理模块：管理训练数据，处理训练数据（在开始的简单的例子中，直接将数据内嵌到app中，后期会开发独立的**数据管理模块**。）

- WebSocketClient：负责与服务器进行通信

## 服务端

- Server：服务端主进程
- Web：服务端管理模块，提供给管理人员Web接口，对系统进行管理
- 任务库：存储联邦学习任务以及联邦学习的聚合算法
- WebSocketServer：负责与客户端通信



![image-20200728132844106](https://github.com/zhanghad/Federated_Learning-Project/tree/master/image/image-20200728132844106.png)

服务端可以同时执行多个联邦学习任务，每一个任务拥有独立的WebSocket服务，独立的端口。

客户端在同一时间只可加入一个联邦学习任务。



# 进一步工作

## 核心模块搭建

- 通信协议进一步细化
- 通信效率优化
- 隐私保护算法设计与实现
- 资源分配算法设计与实现
- 激励机制的设计与算法实现
- 联邦系统的安全问题解决

## 系统搭建

- 系统需求分析
- 系统设计
- 实现与部署
- 测试及优化



------

Created by zhanghad. 2020/7/28
