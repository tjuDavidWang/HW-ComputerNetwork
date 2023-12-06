# UDP数据包分析实验 

2023/12/04

## 实验目的

- 本实验旨在分析和理解UDP（用户数据报协议）的工作机制。
- 通过实际操作，了解UDP的特点和应用场景，掌握UDP数据包的结构和功能。
- 使用Packet Tracer和WireShark工具，进行实际的UDP数据包捕获和分析，加深对UDP协议的理解。

## 实验原理 

4. UDP是一种无连接的传输层协议，它在IP协议的基础上提供了复用、分用和错误检测的最基本服务。
2. UDP的特点包括：无连接、开销小、首部开销小、无拥塞控制，适用于实时应用如视频会议和在线游戏。
3. 实验中将使用Packet Tracer来模拟网络环境和UDP数据传输，而WireShark将用于捕获和分析UDP数据包。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：局域网
- 软件：Cisco Packet Tracer虚拟实验环境

## 实验步骤

1. 打开Cisco Packet Tracer虚拟实验环境，按照下面的网络结构图连线

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204142448659.png" alt="image-20231204142448659" style="zoom: 33%;" />

2. 为每个PC配置各自的静态IP，在 Server2、3 上配置 Gateway/IP Address/Subnet Mask（可以使用DHCP，按照上面的进行配置）

3. 将PC2的DNS设置得与Server3的DNS相同，以便能够通过Server3构建映射，访问到Server2

   ![image-20231204145932089](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204145932089.png)

4. 在Server3上配置DNS，将域名设置为`www.tongji.edu.cn`，Detail处写映射到Server2的IP地址（`192.168.2.13`）

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204141054078.png" alt="image-20231204141054078" style="zoom: 33%;" />

5. 用Simulation模式打开，打开PC2的Web Browser，输入配置web服务器的域名（`http://www.tongji.edu.cn`），产生UDP数据流量，<u>分析UDP报文情况</u>

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204142430978.png" alt="image-20231204142430978" style="zoom:33%;" />

6. 用WireShark捕获这些UDP数据包，<u>查看UDP报文字段内容，并解读</u>

## 实验现象

1. 在Packet tracer中UDP报文情况

   选取从Switch2 -> Server3这一段进行分析

   ![image-20231204181253979](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204181253979.png)

   UDP报文情况：

   - **输入PDU详情：**
     - **源端口（Source Port）**: `53`，表示这是一个来自DNS服务器的响应。
     - **目的端口（Destination Port）**: `1062`，这是客户端的端口，用于接收DNS响应。
     - **长度（Length）**: `0x29`，这表示UDP报文的长度，包括首部和数据，十进制为41字节。
     - **校验和（Checksum）**: `0x0`，校验和用于错误检测，这里显示为0，可能是在某些网络环境下不使用或者未被计算。
   - **输出PDU详情：**
     - **源端口（Source Port）**: `1062`，客户端用于发送DNS查询的端口。
     - **目的端口（Destination Port）**: `53`，目标是DNS服务器的标准端口。
     - **长度（Length）**: `0x48`，报文长度，包括首部和数据，十进制为72字节。
     - **校验和（Checksum）**: `0x0`，同上，校验和显示为0。
   
2. 分析WireShark抓取的数据报

   ![image-20231204182400704](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204182400704.png)

   - **目的地址（Destination IP）**: `100.80.165.198`，接收此UDP数据报的设备的IP地址（也是本地的IP地址）。
   - **源端口（Source Port）**: `8000`，发送数据报的应用程序端口号。
   - **目的端口（Destination Port）**: `4023`，数据报应当被送达的应用程序端口号。
   - **长度（UDP Length）**: `79`，UDP包的长度，只包括UDP首部和数据。
   - **校验和（Checksum）**: `0xb190`，UDP校验和值，用于错误检测。

## 分析讨论 

UDP协议的主要特点是其无连接和不保证可靠性。在实验中，我们通过配置Web和DNS服务器，以及使用Packet Tracer和Wireshark工具，可以清楚地观察到UDP的这些特性。首先，UDP的无连接特性表现在数据包的快速传输上，没有连接建立的过程，这减少了通信的延迟。然而，这也意味着UDP不保证数据包的顺序和完整性。在Wireshark抓包分析中，我们可能观察到某些数据包的丢失或到达顺序的不确定性，这正是因为UDP不进行数据包排序或重传处理。此外，UDP的首部开销较小，只有8字节，相比于TCP的至少20字节的首部，使得UDP在传输小量数据时更高效。但这也意味着UDP缺少了TCP中的一些关键控制信息，如序号、确认应答和窗口大小等，这些都是TCP实现可靠传输的重要机制。
