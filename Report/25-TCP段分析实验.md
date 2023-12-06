# TCP段分析实验

2023/12/04

## 实验目的

- 本实验的主要目的是理解和分析TCP（传输控制协议）数据包的结构和传输过程。
- 通过使用工具如Packet Tracer和Wireshark，本实验旨在深入探究TCP的工作原理，包括连接建立、数据传输和连接终止过程。

## 实验原理 

TCP是一种面向连接的、可靠的传输层协议，用于在不可靠的互联网上提供可靠的数据传输。它通过三次握手建立连接，使用序号和确认号来确保数据的有序和可靠传输，并通过四次挥手来终止连接。

#### 三次握手（建立连接）

三次握手是TCP协议建立可靠连接的过程，它包括以下步骤：

1. **SYN**：客户端发送一个SYN（同步序列编号）报文到服务器，并进入SYN_SENT状态。这个报文中包含客户端的初始序列号，用于同步序列号。
2. **SYN-ACK**：服务器收到客户端的SYN报文后，会发送一个SYN-ACK（同步和确认）报文作为响应，并分配缓冲资源和端口，进入SYN_RECEIVED状态。这个报文包含服务器的初始序列号和确认号（客户端的序列号+1）。
3. **ACK**：客户端收到服务器的SYN-ACK报文后，发送一个ACK（确认）报文，确认号是服务器的序列号+1，并分配客户端的缓冲资源和端口。此时，客户端进入ESTABLISHED状态。当服务器收到这个ACK报文后，也进入ESTABLISHED状态，双方建立起稳定的连接。

这个过程确保了双方都能确认对方的接收和发送能力，是建立TCP连接的必要步骤。

#### 四次挥手（断开连接）

四次挥手是TCP协议断开已建立连接的过程，它包括以下步骤：

1. **FIN**：当通信的一方完成数据发送任务后，会发送一个FIN（结束）报文，请求关闭连接，并进入FIN_WAIT_1状态。
2. **ACK**：另一方收到FIN报文后，会发送一个ACK报文作为响应，并进入CLOSE_WAIT状态。发送完ACK报文后，第一方进入FIN_WAIT_2状态。
3. **FIN**：处于CLOSE_WAIT状态的一方，在发送完所有待发送数据后，发送一个FIN报文，并进入LAST_ACK状态。
4. **ACK**：第一方收到FIN后，发送一个ACK报文作为最终确认，并进入TIME_WAIT状态。确认后，第二方关闭连接，进入CLOSED状态。第一方等待足够的时间以确保第二方接收到最终的ACK报文后，也关闭连接，进入CLOSED状态。

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

5. 用Simulation模式打开，打开PC2的Web Browser，输入配置web服务器的域名（`http://www.tongji.edu.cn`），产生TCP数据报文，<u>分析报文情况</u>

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204142430978.png" alt="image-20231204142430978" style="zoom:33%;" />

6. 用WireShark抓取TCP数据包，<u>查看TCP报文字段内容，并解读；仔细研读TCP连接建立过程数据报文；仔细研读TCP拆链过程数据报文</u>。

## 实验现象

1. Packet Tracer中TCP数据报情况

   选取从Switch2 -> Server2这一段进行分析

   ![image-20231204160355337](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204160355337.png)

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204160753131.png" alt="image-20231204160753131" style="zoom:50%;" />

   按照上图对TCP数据块进行分析

   - **输入PDU详情：**
     - **源端口**：1031
     - **目标端口**：80（通常用于HTTP服务）
     - **序列号**和**确认号**均为0，这通常是在建立连接时的SYN报文。
     - **源IP**：192.168.2.11
     - **目的IP**：192.168.2.13
     - **以太网II帧头**：包含目标和源MAC地址。
     - **TCP控制位**：不显示在这个阶段的截图中，SYN位是设置的，这表明这是一个连接请求。
   - **输出PDU详情：**
     - **源端口**：80
     - **目标端口**：1031
     - **序列号**：0
     - **确认号**：1（表明第一个阶段的SYN请求已被确认）
     - **TCP控制位**：设置了SYN和ACK，这是三次握手过程中的第二次握手。

2. 分析WireShark抓取的数据报

   - **TCP链接建立过程**

     ![image-20231204161933553](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204161933553.png)

     - **源端口号（Source Port）**：`11152`，这是发送方的端口号。
     - **目的端口号（Destination Port）**：`64066`，这是接收方的端口号。
     - **序列号（Sequence Number）**：`4194102590`，这是发送的数据的第一个字节的序列号。
     - **确认号（Acknowledgment Number）**：`2323`，这是接收方期望收到的下一个字节的序列号，它确认了接收方已经接收到的数据。
     - **数据偏移（Header Length）**：20字节（5个32位字），表示没有额外的TCP选项被设置。
     - **标志位（Flags）**：`0x018`（PSH, ACK），这里设置了PSH（Push Function）和ACK（Acknowledgment）标志位。
       - PSH：指示接收方应该立即将这些数据传递给上层应用而不是等待缓冲。
       - ACK：确认收到了匹配的序列号的数据。
     - **窗口大小（Window）**：`149`，这是接收方当前的接收窗口大小，告知发送方它还能接收多少字节的数据。
     - **校验和（Checksum）**：`0x6bd5`，用于错误检测，这个值是对整个TCP段（包括TCP伪头部，TCP头部和数据）计算得出的。
     - **紧急指针（Urgent Pointer）**：`0`，因为URG标志没有设置，所以这个字段不被使用。
     - **选项（Options）**：截图中没有显示TCP选项，但由于头部长度为20字节，可以推断没有额外的选项。

   - **TCP连接拆链过程**

     ![image-20231204163251397](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204163251397.png)

     - **源端口（Source Port）和目的端口（Destination Port）**：这个数据包是从源端口`80`发送到目的端口`64181`。
     - **序列号（Sequence Number）**：`1`，这是相对序列号，Wireshark将其显示为1是因为它是相对于TCP连接初始化时的序列号。
     - **确认号（Acknowledgment Number）**：`293474333`，这是相对确认号，表示发送方已经收到对方发送的数据直到这个序号。
     - **标志位（Flags）**：`0x011`，其中FIN和ACK标志都被设置。FIN标志表示发送方已经完成了数据传输并希望开始终止连接。ACK标志是对先前接收到的TCP段的确认。
     - **窗口大小（Window）**：`57`，这是接收窗口的大小，表示在不接收进一步确认的情况下接收方还能接收多少字节的数据。
     - **校验和（Checksum）**：`0x6353`，用于验证数据包在传输过程中的完整性和正确性。
     - **紧急指针（Urgent Pointer）**：`0`，因为URG标志未设置，所以此字段不被使用。

## 分析讨论 

1. **DNS相关知识**

   DNS，或域名系统，是网络上的一种关键服务，它负责将用户友好的域名，如`www.example.com`，转换为机器可识别的IP地址，例如`192.168.1.13`。这个转换过程使用户无需记忆复杂的IP地址，只需通过容易记住的域名即可访问网站。当用户输入一个网址时，他们的设备首先会检查本地DNS缓存来解析该域名。如果缓存中没有找到相应记录，设备配置的DNS服务器就会被查询。如果这个DNS服务器不知道请求的IP地址，它会进行一系列递归查询，通过询问其他服务器，直到找到正确的IP地址。在DNS配置方面，可以手动为设备指定一个静态DNS服务器地址，或者使用动态DNS（DDNS）来允许自动的DNS记录更新，这通常与DHCP服务配合使用，以便自动分配IP地址和更新DNS记录。这种配置提供了便捷性和灵活性，特别是在需要对IP地址进行更改或更新时。

   在本实验中，

   - 在Server2和Server3上配置静态IP和网关：这确保了网络中的设备可以互相通信并且可以通过网关访问外部网络。
   - 在Server3上配置DNS并将PC2的DNS设置指向Server3：这允许PC2通过Server3来解析域名。当PC2试图访问`www.tongji.edu.cn`时，它会查询Server3上的DNS服务，该服务已配置为将此域名解析为Server2的IP地址。
   - 在Server3上将`www.tongji.edu.cn`映射到Server2的IP地址：这使得任何向Server3查询这个域名的设备都被告知Server2的IP地址，允许正确的名称解析和访问控制。
