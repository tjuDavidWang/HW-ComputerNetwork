# 动态IP地址分配DHCP实验

2023/11/06

## 实验目的

1. 了解和掌握DHCP（Dynamic Host Configuration Protocol）的工作原理。
2. 学习如何在路由器上配置DHCP服务。
3. 观察和理解动态分配IP地址的过程。

## 实验原理 

- 动态主机配置协议（DHCP）是一种网络管理协议，用于自动分配IP地址给网络中的计算机，以便它们可以进行通信。DHCP允许设备（称为DHCP客户端）在加入网络时自动获取必要的网络配置参数，而不需要网络管理员手动进行配置。
- DHCP协议采用UDP作为传输协议，主机发送请求消息到DHCP服务器的67号端口，DHCP服务器回应应答消息给主机的68号端口。
- 使用DHCP客户端可以带来如下好处：①降低了配置和部署设备时间; ②降低了发生配置错误的可能性; ③可以集中化管理设备的IP地址分配。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：局域网
- 软件：Cisco Packet Tracer虚拟实验环境

## 实验步骤

1. 按照下图所示，连接电线构成网络

   ![image-20231106150240082](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106150240082.png)

2. 配置路由器Router0的接口

   可以在CLI中输入以下命令进行配置，也可以使用图形化界面

   ```CPP
   interface FastEthernet0/0
   ip address 192.168.1.1 255.255.255.0
   interface FastEthernet0/1
   ip address 192.168.2.1 255.255.255.0
   ```

3. <u>查看各PCIP地址的情况</u>

4. 在Router0中输入以下命令，以配置DHCP左右两边的网络

   ```CPP
   //路由器DHCP左边网络
   ip dhcp excluded-address 192.168.1.0 192.168.1.10 //设置DHCP服务器不分配给客户端的IP地址范围
   ip dhcp pool myleftnet//创建了一个名为"myrightnet"的DHCP地址池
   network 192.168.1.0 255.255.255.0//指定了DHCP地址池中可用的IP地址范围
   default-router 192.168.1.1//指定了DHCP客户端在获取IP地址后使用的默认网关路由器的IP地址
   option 150 ip 192.168.1.3//指定了DHCP客户端可以使用的TFTP服务器的IP地址
   dns-server 192.168.1.2//指定了DHCP客户端在获取IP地址后使用的DNS服务器的IP地址
   ```

   ```cpp
   //路由器DHCP右边网络
   ip dhcp excluded-address 192.168.2.0 192.168.2.10
   ip dhcp pool myrightnet
   network 192.168.2.0 255.255.255.0
   default-router 192.168.2.1
   option 150 ip 192.168.2.3
   dns-server 192.168.2.2
   ```

5. <u>查看各IP地址的情况</u>

6. 按照如下网络拓补图连线

   ![image-20231106151630591](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106151630591.png)

7. 同上4对CopyRouter0进行配置（将'1'、'2'修改成'3'、'4'即可）
8. 同时，对仿照**RIP实验**对串口线进行配置，使得左边的可以对右边进行访问【也可以使用静态路由配置】
9. <u>左右两边的电脑相互ping，观察实验现象</u>

## 实验现象

1. 配置DHCP前查看各PC的IP地址情况

   各个PC的IP地址均相同，且都为0.0.0.0

   ![image-20231106152943782](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106152943782.png)

2. 配置DHCP后查看各PC的IP地址情况

   ![image-20231106153116801](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106153116801.png)

   myleftnet中的两台分别是`192.168.1.12`与`192.168.1.14`【每次分配不一样】

   myrightnet中的两台分别是`192.168.2.12`与`192.168.2.14`【每次分配不一样】

3. 配置完DHCP后，左右两边的电脑相互ping，均能够ping通

   ![image-20231106153838534](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106153838534.png)

## 分析讨论 

1. 配置好DHCP之后左右两边还是无法ping通

   需要将PC的Gateway/DNS IPv4调整为DHCP（不再是Static）

   > 当PC的DNS设置仍然是静态的，这意味着它们可能使用的是不正确或不再有效的DNS服务器地址。虽然这通常不会影响本地网络内的ping操作（因为ping基于IP地址，不需要DNS解析），但如果ping操作是通过主机名而不是IP地址执行的，那么DNS解析是必须的。如果DNS服务器配置不正确，主机名就无法解析为正确的IP地址，导致ping失败。在本项目中使用的是DHCP技术对获得IP地址、Gateway地址、DNS服务器地址等信息进行配置。

   ![配置](C:\Users\12920\Documents\WeChat Files\wxid_fgqjq7s5d61c22\FileStorage\File\2023-11\DHCP实验图片\配置.png)

   
