# NAT网络地址转换

2023/10/30

## 实验目的

熟练掌握NAT的基本技术原理和配置方法，理解NAT在解决IP地址不足及提供网络隔离的作用。

## 实验原理 

- **NAT (Network Address Translation)** 是一个网络地址转换技术，用于在Internet接入中转换IP地址，从而解决<u>IP地址不足的问题</u>。NAT将局域网内部的本地地址转换为全局地址后转发数据包，隔离了内部和外部网络，加强了安全性。
- NAT分为三种：静态NAT、动态NAT、和NAPT。静态NAT是一对一的映射，动态NAT是一对多的映射，而NAPT使用不同的端口来映射多个内网IP地址到一个指定的外网IP地址。

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

4. 配置路由器RA与RB的静态路由表、NAT出入口、NAT转换

   ```cpp
   //RA
   //静态路由表
   ip route 218.100.3.0 255.255.255.0 serial 0/0/0
   ip route 118.18.4.0 255.255.255.0 serial 0/0/0
   //NAT出入口
   interface FastEthernet0/0 
   ip nat inside
   interface Serial 0/0/0 
   ip nat outside
   //NAT转换
   ip nat inside source static 192.168.1.11 210.120.1.11
   //RB
   ip route 10.60.2.0 255.255.255.0 serial 0/0/0
   ip route 210.120.1.0 255.255.255.0 serial 0/0/0
   //NAT出入口
   interface FastEthernet0/0 
   ip natinside
   interface Serial 0/0/0 
   ip natoutside
   //NAT转换
   ip nat inside source static 172.16.3.33 218.100.3.33
   ```

5. 在RA与RB中，分别输入`show ip nat translations`观察结果

6. 在每个PC端访问输入以下测试是否能ping通

   ```cpp
   ping 192.168.1.11
   ping 210.120.1.11
   ping 10.60.2.22
   ping 172.16.3.33
   ping 218.100.3.33
   ping 118.18.4.44
   ```

## 实验现象

1. 在RA与RB中，分别输入`show ip nat translations`观察结果

   ![image-20231030155813486](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231030155813486.png)

2. 在PC1上ping的结果

   |                  | PC1   | PC2   | PC3   | PC4   |
   | ---------------- | ----- | ----- | ----- | ----- |
   | **192.168.1.11** | **✓** | **✓** | **✕** | **✕** |
   | **210.120.1.11** | **✕** | **✕** | **✓** | **✓** |
   | **10.60.2.22**   | **✓** | **✓** | **✓** | **✓** |
   | **172.16.3.33**  | **✕** | **✕** | **✓** | **✓** |
   | **218.100.3.33** | **✓** | **✓** | **✕** | **✕** |
   | **118.18.4.44**  | **✓** | **✓** | **✓** | **✓** |

   ![image-20231030161411866](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231030161411866.png)

## 分析讨论 

通过NAT技术，对4个PC的ip进行总结，得到下表

|      |    内部IP    |    公共IP    |
| :--: | :----------: | :----------: |
| PC1  | 192.168.1.11 | 210.120.1.11 |
| PC2  |  10.60.2.22  |              |
| PC3  | 172.16.3.33  | 218.100.3.33 |
| PC4  | 118.18.4.44  |              |

可以总结如下：

- 同一网内：能访问内网IP，无法访问公网IP
- 不同网内：能访问公网IP，无法访问内网IP