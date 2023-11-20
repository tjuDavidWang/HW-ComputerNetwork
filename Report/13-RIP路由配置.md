# RIP路由配置

2023/10/30

## 实验目的

探索和学习RIP路由信息协议，熟悉RIP的配置流程和网络间互通性测试。

## 实验原理 

RIP（Routing Information Protocol，路由信息协议）是一种距离矢量协议，用于小型同类网络，RIP是一种简单的内部网关协议，用于在路由器之间交换路由信息。RIP使用跳数作为衡量路径的开销，规定最大跳数为15。RIP协议分为RIPv1和RIPv2两个版本：RIPv1属于有类路由协议，不支持VLSM，以广播形式更新路由信息，更新周期为30秒；而RIPv2则属于无类路由协议，支持VLSM，并以组播形式更新路由。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：TJ-WIFI
- 软件：Cisco Packet Tracer虚拟实验环境


## 实验步骤

1. 按照如下的网络拓补图，连接链路

   ![image-20231030134643574](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231030134643574.png)


2. 按照网络拓补图配置PC机的地址、网关、掩码。

3. 配置路由器的端口地址、串口地址

   在RA、RB的CLI中，依次输入如下指令

   ```cpp
   //路由器A
   interface FastEthernet0/0
   ip address 192.168.1.254 255.255.255.0
   interface FastEthernet0/1
   ip address 10.60.2.254 255.255.255.0
   interface Serial 0/0/0
   ip address 202.120.17.18 255.255.255.0
   Clock rate 56000
   //路由器B
   interface FastEthernet0/0
   ip address 172.16.3.254 255.255.255.0
   interface FastEthernet0/1
   ip address 118.18.4.254 255.255.255.0
   interface Serial 0/0/0
   ip address 202.120.17.29 255.255.255.0
   Clock rate 56000
   ```

4. 几台计算机互相Ping, 观测访问结果

5. 分别配置RA与RB的RIP路由表

   ```cpp
   //RA
   router rip 
   network 192.168.1.11
   network 10.60.2.22
   network 202.120.17.18
   //RB
   router rip 
   network 172.16.3.33
   network 118.18.4.44
   network 202.120.17.29
   ```

6. 几台计算机互相Ping, 观测访问结果

7. 删除RA中的路由表后，几台计算机互相Ping, 观测访问结果

## 实验现象

1. 在路由器A 和B配置RIP之前，几台计算机互相Ping, 观测访问结果

   通过同一路由器连接的PC之间相互ping可以ping到，不同的ping不到

   ![image-20231030155214350](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231030155214350.png)

2. 路由器A或B配置RIP之后，几台计算机互相Ping,观测访问结果

   通过同一路由器连接的PC之间相互ping可以ping到，不同的ping不到

   ![image-20231030155304731](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231030155304731.png)

3. 路由器A和B配置RIP之后，几台计算机互相Ping,观测访问结果

   不同PC之间都可以ping得到
   
   ![image-20231030155542257](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231030155542257.png)

## 分析讨论 

1. 在RIP未配置前，由于缺乏正确的路由信息，PC之间的数据包可能无法正确传递，因此互相Ping可能失败。
2. 当在其中一个路由器上配置RIP后，与该路由器直连的PC可能可以互通，但与另一路由器直连的PC之间可能仍然无法通信。
3. 当两个路由器都配置了RIP后，两个路由器之间会交换路由信息，从而确保整个网络中的所有设备都可以互相通信。这证明了RIP协议在保持网络间互通性方面的重要性。