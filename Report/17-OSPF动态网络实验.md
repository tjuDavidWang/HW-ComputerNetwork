# OSPF动态网络路由配置实验

2023/11/06

## 实验目的

1. 理解和掌握OSPF（开放最短路径优先）动态路由配置方法
1. 熟悉OSPF的工作原理及其在自治系统内部的应用
1. 通过配置OSPF来实现网络中的路由器之间的动态路由交换，提高网络的可靠性和数据传输效率。

## 实验原理 

OSPF是一个内部网关协议（IGP），它使用链路状态路由算法来确保每个OSPF路由器拥有自治系统内完整拓扑的一致视图。OSPF通过构建一个链路状态数据库（LSDB）并使用该数据库来计算最短路径树，为数据包寻找到最佳路径。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：局域网
- 软件：Cisco Packet Tracer虚拟实验环境

## 实验步骤

1. 按照下图所示，连接电线构成网络（注意需要为两个路由器添加串口配置）

   ![image-20231106155420918](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106155420918.png)

2. 按照图中描述的配置PC、服务器的地址、网关和掩码

3. 按照图中的描述配置路由器的端口地址

4. 输入以下命令配置路由器的串口端口地址

   ```CPP
   //路由器A
   interface Serial 0/0/0
   ip address 202.120.17.18 255.255.255.0
   clock rate 56000
   //路由器B
   interface Serial 0/0/0
   ip address 202.120.17.29 255.255.255.0
   clock rate 56000
   ```

5. <u>检查不同的PC之间是否能够ping通</u>

6. 分别配置RA与RB的OSPF路由表

   ```CPP
   //路由器A
   router ospf 1//指示路由器启动OSPF进程，并为该进程分配了一个进程ID号1
   //0.0.0.255是通配掩码，与子网掩码相反，它在OSPF配置中用来指定哪些IP位需要匹配，0表示必须匹配
   network 192.168.1.0 0.0.0.255 area 0//所有在192.168.1.0网络上的接口都应该使用OSPF进行通告，该网络属于OSPF区域0
   network 10.60.2.0 0.0.0.255 area 0
   network 202.120.17.0 0.0.0.255 area 0
   //路由器B
   router ospf 1
   network 172.16.3.0 0.0.0.255 area 0
   network 118.18.4.0 0.0.0.255 area 0
   network 202.120.17.0 0.0.0.255 area 0
   ```

7. <u>路由器A或B配置OSPF之后，几台计算机互相Ping，观测访问结果。</u>

8. <u>路由器A和B配置OSPF之后，几台计算机互相Ping，观测访问结果。</u>

9. 通过`Router#sh ip ospf neighbor`，<u>查看路由器的邻居</u>

## 实验现象

1. 配置OSPF之前，任意两个PC之间都不能ping通

   ![image-20231106161904994](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106161904994.png)

2. 路由器A或B配置OSPF之后，几台计算机互相ping，RA的两台PC之间可以ping通，其他的均不行

   <img src="C:\Users\12920\AppData\Roaming\Tencent\QQ\Temp\W7SJHO}HRYZLW3~DPH3VS5W.png" alt="W7SJHO}HRYZLW3~DPH3VS5W" style="zoom:0%;" />

3. 路由器A和B配置OSPF之后，几台计算机互相ping，均可以ping通

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106162945843.png" alt="image-20231106162945843" style="zoom:67%;" />

4. 路由器的邻居如下

   ![image-20231106163351223](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231106163351223.png)

## 分析讨论 

1. **配置OSPF之前PC1与PC2之间ping不通**

   PC1和PC2虽然连接到同一个路由器RA，但它们配置在不同的子网（网络部分不同）中：

   - PC1有IP地址192.168.1.11，子网掩码255.255.255.0，网关192.168.1.254
   - PC2有IP地址10.60.2.22，子网掩码255.255.255.0，网关10.60.2.254

   即使它们连接到同一个物理路由器，如果没有配置跨子网的路由，它们也无法直接通信。

   ```CPP
   router ospf 1
   network 192.168.1.0 0.0.0.255 area 0
   network 10.60.2.0 0.0.0.255 area 0
   ```

   以上代码便是允许跨子网进行通信。
