# ACL访问控制实验

2023/10/23

## 实验目的

1. 理解接入控制列表 (ACLs) 的原理和功能。
2. 学习如何配置标准和扩展的 IP 访问列表。
3. 验证ACL配置对网络访问的影响。

## 实验原理 

1. **接入控制列表 (ACLs) 原理**: ACLs(Access Control Lists)，也被称为访问列表或防火墙，通过定义规则对网络设备接口上的数据报文进行控制：允许或丢弃，从而增强网络的可管理性和安全性。
2. **IP 访问列表**: IP ACL分为标准和扩展两种。标准IP访问列表只基于数据包的源IP地址过滤，而扩展IP访问列表可以根据数据包的源IP、目的IP、源端口、目的端口、协议来进行过滤。
3. **接口应用**: IP ACL在接口上应用时，分为入栈和出栈两种方式。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：局域网
- 软件：Cisco Packet Tracer虚拟实验环境

## 实验步骤

1. 按照下图所示，连接电线构成网络（注意需要为两个路由器添加串口）

   ![image-20231029201520530](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029201520530.png)

2. 配置PC、服务器的地址、网关和掩码

   ![image-20231029202228700](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029202228700.png)

   ![image-20231029202134751](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029202134751.png)

3. 配置路由器端口地址、串口地址、静态路由表

   对于Router0而言，在CLI中输入以下指令

   ```cpp
   //配置端口地址
   enable
   configure terminal
   interface FastEthernet0/0
   ip address 192.168.1.254 255.255.255.0
   no shutdown
   exit
   interface FastEthernet0/1
   ip address 10.60.2.254 255.255.255.0
   no shutdown
   //配置串口地址
   enable
   configure terminal
   interface Serial0/1/0
   ip address 202.120.17.18 255.255.255.0
   clock rate 56000
   no shutdown
   //配置静态路由表
   ip route 172.16.3.0 255.255.255.0 Serial0/1/0
   ip route 118.18.4.0 255.255.255.0 Serial0/1/0
   ```

   也可以在配置中进行配置

   ![image-20231029203226636](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029203226636.png)

   ![image-20231029203334190](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029203334190.png)

   ![image-20231029203421768](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029203421768.png)

   对于Router1而言，在CLI中输入以下指令

   ```cpp
   //配置端口地址
   enable
   configure terminal
   interface FastEthernet0/0
   ip address 172.16.3.254 255.255.255.0
   no shutdown
   exit
   interface FastEthernet0/1
   ip address 118.18.4.254 255.255.255.0
   no shutdown
   //配置串口地址
   enable
   configure terminal
   interface Serial0/1/0
   ip address 202.120.17.29 255.255.255.0
   clock rate 56000
   no shutdown
   //配置静态路由表
   ip route 192.168.1.0 255.255.255.0 Serial0/1/0
   ip route 10.60.2.0 255.255.255.0 Serial0/1/0
   ```

4. 在其他PC上重新尝试访问`172.16.3.33`服务器（通过ping和http方法），并观察结果。

5. 配置Router1的ACL表：

   ```cpp
   //拒绝ping包
   access-list 101 deny icmp host 192.168.1.11 host 172.16.3.33
   //允许www访问
   access-list 101 permit tcp host 192.168.1.11 host 172.16.3.33 eq www
   //应用到端口上
   interface Serial0/1/0
   ip access-group 101 in
   ```

6. 在其他PC上重新尝试访问`172.16.3.33`服务器（通过ping和http方法），并观察结果。

## 实验现象

1. 配置ACL前，各PC访问服务器均成功

   ![image-20231029203808591](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029203808591.png)

2. 配置ACL后，访问各个PC端，可以发现下表

   | PC   | ping                                                         | http                                                         |
   | ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | PC0  | 失败![image-20231029203954466](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029203954466.png) | 成功![image-20231029204005855](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029204005855.png) |
   | PC1  | 失败![image-20231029204019809](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029204019809.png) | 失败![image-20231029204030463](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029204030463.png) |
   | PC2  | 成功![image-20231029204039679](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029204039679.png) | 成功![image-20231029204047001](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231029204047001.png) |

   

## 分析讨论 

本实验展示了如何在路由器上配置ACL以及ACL对网络通信的影响。通过逐步地配置和观察结果，我们了解了ACL的强大功能和其在网络管理中的重要性。为了确保网络的稳定和安全，管理员需要仔细地设计和实施ACL策略。

通过ACL，我们对特定的数据流进行了限制。在本实验中，我们对来自PC0的到服务器的ping请求进行了拦截，但允许了http请求。而对于PC1，它的http请求也被拦截。PC2则没有受到任何限制。

- **PC0:** ping失败，http成功。这是由于ACL拦截了ping包，但允许了http请求。
- **PC1:** ping和http都失败。这可能意味着在ACL中，有其他规则阻止了来自PC1的请求或者PC1的配置有问题。
- **PC2:** ping和http都成功。这说明PC2没有受到ACL的任何限制。
