# 帧中继配置Frame Relay实验

2023/10/23

## 实验目的

熟悉帧中继(Frame Relay)的配置方法，理解帧中继的基本原理和特点，并验证其在网络中的工作效果。

## 实验原理 

帧中继（Frame Relay）是一种在广域网络(WAN)中用于高速数据传输的技术。其核心原理和特点如下：

1. **面向连接的数据链路技术**：帧中继建立在虚电路（Virtual Circuit，VC）的概念之上，其中每个虚电路都有一个唯一的数据链路连接标识符（DLCI）来标识。这意味着数据在传输之前会首先建立一个连接，并在此连接上进行数据交换。
2. **DLCI的功能与作用**：DLCI是帧中继虚电路的本地标识。它不仅用于标识路由器的接口，还用于标识交换机中的路径。重要的是，DLCI在帧中继网络中是局部意义的，它在全局范围内可能不是唯一的。
3. **多路复用**：帧中继允许多个虚电路在同一个物理链路上进行多路复用，这样可以有效地利用带宽资源，降低通信成本。
4. **两种虚电路类型**：PVC（永久虚电路）和SVC（交换虚电路）。其中，PVC一旦建立就会持续存在，而SVC是临时的，建立连接后数据传输完成会被撤销。
5. **数据转发**：在帧中继交换机中存在一个映射表，该表将DLCI映射到相应的输出端口。当交换机收到一个帧时，它会根据帧的DLCI查找映射表，并将帧转发到与其相关联的输出端口。
6. **地址映射**：在Cisco路由器中，地址映射可以是手动配置的，或者使用动态地址映射。动态地址映射时，帧中继地址解析协议(ARP)会为特定的连接查找下一个跳转的协议地址。帧中继ARP通常也被称为反向ARP。
7. **封装选项**：在连接Cisco路由器时，默认封装是Cisco，但当与非Cisco路由器连接时，应选择“IETF”作为封装选项。

通过这些原理，帧中继为广域网络提供了一个高效、灵活和可靠的数据传输机制，特别是在需要高带宽和多路复用的场景中。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：局域网
- 软件：Cisco Packet Tracer虚拟实验环境

## 实验步骤

1. 按照下图所示，连接电线构成网络；在连接时，需要注意，需要在路由器上增加HWIC-2T接口卡，以提供串行端口。

   ![image-20231029193021131](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029193021131.png)

   ![image-20231029125712636](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029125712636.png)

2. 进行帧中继交换机配置（配置交换机帧中继）。

   需要在Serial0、Serial1、Serial2中配置，为DLCI命名，指示从一个端口的子链接（Sublink）到另一个端口的子链接的连接关系

   ![image-20231029130401148](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029130401148.png)

   ![image-20231029130445420](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029130445420.png)

3. 依次在CLI中输入以下命令以配置Router0

   ```c++
   //启动端口和封装帧中继
   enable
   configure terminal
   interface Serial0/1/0
   no shutdown
   encapsulation frame-relay
   exit
   //配置102子端口用于DLCI为102（即RO-R1）的子链路
   interface Serial0/1/0.102 point-to-point
   ip address 1.1.1.1 255.255.255.252
   frame-relay interface-dlci 102
   exit
   //配置103子端口用于DLCI为103（即RO-R2）的子链路
   interface Serial0/1/0.103 point-to-point
   ip address 1.1.1.5 255.255.255.252
   frame-relay interface-dlci 103
   ```

   ![image-20231029184251074](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029184251074.png)

4. 依次在CLI中输入以下命令以配置Router1

   ```cpp
   //启动端口和封装帧中继
   enable
   configure terminal
   interface Serial0/1/0
   no shutdown
   encapsulation frame-relay
   exit
   //配置201子端口用于DLCI为201（即R1-R0）的子链路
   interface Serial0/1/0.201 point-to-point
   ip address 1.1.1.2 255.255.255.252
   frame-relay interface
   frame-relay interface-dlci 201
   ```

5. 依次在CLI中输入以下命令以配置Router2

   ```cpp
   //启动端口和封装帧中继
   enable
   configure terminal
   interface Serial 0/0/0
   no shutdown
   encapsulation frame-relay
   exit
   //配置301子端口用于DLCI为301（即R2-R0）的子链路
   interface Serial 0/0/0.301 point-to-point
   ip address 1.1.1.6 255.255.255.252
   frame-relay interface-dlci 301
   ```

6. <u>检查接口间能否相互ping通</u>，观察现象

7. 为Router1、Router2进行静态路由配置

   ```cpp
   ip route 1.1.1.4 255.255.255.252 1.1.1.1
   ip route 1.1.1.0 255.255.255.252 1.1.1.5
   ```

8. <u>检查接口间能否相互ping通</u>，观察现象

9. 创建三个电脑PC0、PC1、PC2，分别连接至Router0、Router1、Router2的FastEthernet0/0口上

   ![image-20231029191859905](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029191859905.png)

10. 再次配置Router0、Router1、Router2

    对于Router0：

    - interface FastEthernet0/0

    - ip address 1.1.1.9 255.255.255.252

    - 在RIP路由协议中添加1.0.0.0

      ![image-20231029135646807](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029135646807.png)

    对于Router1：

    - interface FastEthernet0/0

    - ip address 1.1.1.13 255.255.255.252

    - 在RIP路由协议中添加1.0.0.0

    对于Router2：

    - interface FastEthernet0/0

    - ip address 1.1.1.17 255.255.255.252

    - 在RIP路由协议中添加1.0.0.0

11. 对PC0、PC1、PC2进行配置

    对于PC0：

    - 网关：1.1.1.9

    - FastEthernet0：1.1.1.10 255.255.255.252

      ![image-20231029140111452](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029140111452.png)

    对于PC1：

    - 网关：1.1.1.13

    - FastEthernet0：1.1.1.14 255.255.255.252

    对于PC2：

    - 网关：1.1.1.17

    - FastEthernet0：1.1.1.18 255.255.255.252

12. <u>PC0、PC1、PC2相互ping</u>，观察现象

## 实验现象

1. 最开始配置好交换机帧中继与路由器后发现：Router0分别与Router1和Router2之间能够相互ping通，但是Router1与Router2之间不能够ping通。

   ![image-20231029191747522](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029191747522.png)

2. 为Router1与Router2配置好静态路由后可以相互ping通

   ![image-20231029191848034](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029191848034.png)

3. PC0、PC1、PC2之间相互ping均成功

   ![image-20231029140129516](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029140129516.png)

## 分析讨论 

1. **使用帧中继技术的原因**

   本次实验通过仿真局域网配置了帧中继技术，目的是实现Router0与Router1和Router2的有效通信。在没有使用帧中继之前，Router0需要与Router1和Router2进行直接连接，这样会导致Router0需要更多的端口，并且可能因为两个路由器距离较远而增加连接的成本和降低传输性能。通过帧中继的引入，在数据链路层上可以复用Router0的端口，从而只需要将Router0连接到帧中继交换机，达到了多路复用和中继的目的。

2. **没有配置静态路由时无法ping到**

   这是因为它们的Serial口地址属于不同的网段。

   考虑到子网掩码为255.255.255.252，这意味着只有最后两位用于主机编码，而前30位用于网段编码。【且由于二进制的两位可以表示的值为00、01、10和11，但我们需要排除全0（网络地址）和全1（广播地址），所以实际上只有两个可用的主机地址】<u>当两个路由器通过帧中继网络连接，但它们的接口不在同一个IP网段时，这两个路由器之间的通信需要额外的路由信息</u>。

   为了解决这个问题，需要配置静态路由，这样Router1和Router2就可以通过Router0进行通信。具体的语法如下：`ip route:+目标网络的网络地址+子网掩码+这是下一跳的IP地址`。

3. **PC端之间的通信**

   另外，考虑到PC连接的部分，由于地址空间有限（最多4个地址），因此每个PC与其连接的路由器处于不同的网段。根据约定，网络的首地址和末地址有特殊用途，所以选择中间的地址作为路由器的端口地址和PC的地址，由此，便有如下：

   对于PC0：

   - 网关：1.1.1.9

   - FastEthernet0：1.1.1.10 255.255.255.252

   对于PC1：

   - 网关：1.1.1.13

   - FastEthernet0：1.1.1.14 255.255.255.252

   对于PC2：

   - 网关：1.1.1.17

   - FastEthernet0：1.1.1.18 255.255.255.252

   但PC之间还是ping不通，原因是原始配置中缺乏了对于路由器到PC之间的配置，此时采用RIP(Routing Information Protocol) 动态路由协议，使用`network 1.0.0.0`，使得所有1.x.x.x地址范围的接口都应该广播RIP更新，并接收此范围内的其他路由器的RIP更新，这样就可以实现自动广播和学习。
