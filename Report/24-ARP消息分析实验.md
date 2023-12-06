# ARP消息分析实验

2023/11/27

## 实验目的

本实验旨在深入理解ARP（Address Resolution Protocol，地址解析协议）的工作原理及其在网络通信中的重要作用。通过分析ARP报文的结构和行为，学习如何从理论到实践应用ARP协议进行IP地址到MAC地址的映射，以及如何利用工具观测和分析ARP通信过程。

## 实验原理 

ARP协议的核心功能是<u>将网络层的IP地址转换为数据链路层的MAC地址</u>。这一转换对于在以太网环境中的数据传输至关重要，因为数据链路层的通信依赖于MAC地址。实验将通过静态映射和动态映射两种ARP映射方式来展示地址转换过程。静态映射涉及手动创建ARP表，而动态映射则依赖于ARP和RARP协议自动查找相应地址。实验还将探讨ARP请求和响应的流程，以及ARP报文的结构和字段。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：局域网
- 软件：Cisco Packet Tracer虚拟实验环境

## 实验步骤

1. 打开Cisco Packet Tracer虚拟实验环境，按照下面的网络结构图连线

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231127165143864.png" alt="image-20231127165143864" style="zoom:33%;" />
2. 使用`arp -d`清除PC端的arp缓存，使用clear arp-cache清空Router的arp缓存

   清理前后如下图所示：

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231127165845804.png" alt="image-20231127165845804" style="zoom: 50%;" />

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231127170014078.png" alt="image-20231127170014078" style="zoom: 50%;" />

3. 打开Simulation模式，使用PC1（`192.168.2.13`）去ping PC0（`192.162.1.11`），使用simulation去捕获其中的数据包，<u>分析其中的ARP数据包</u>

4. <u>查看本机ARP内容</u>

5. 使用WireShark抓取ARP报文，<u>对报文进行分析</u>

## 实验现象

1. **Cisco虚拟环境中的报文**

   以PC1->Router1为例分析ARP报文：

   ![image-20231127195630415](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231127195630415.png)

   - **OSI Model**：展示了ARP请求报文是如何被Router1接收的。在OSI模型的第二层（数据链路层），Ethernet II Header显示源MAC地址为`0002.1722.CDCA`（PC1），目的地MAC地址为广播地址`FFFF.FFFF.FFFF`，表明这是一个广播帧。ARP包的源IP地址是`192.168.2.13`（PC1），目的IP地址是`192.168.2.1`（Router右端口），这说明主机试图解析IP地址`192.168.2.1`对应的MAC地址。
   - **输入PDU详情**：详细描述了ARP请求的格式。
     - **硬件类型**（Hardware Type）：在这个ARP响应中，硬件类型为`0x1`，指示这是一个以太网帧。
     - **硬件长度**（HLEN, Hardware Length）：为6，表示物理地址（MAC地址）的长度为6个字节。
     - **协议长度**（PLEN, Protocol Length）：为4，表示协议地址（IP地址）的长度为4个字节。
     - **操作码**（Opcode）：为`0x1`，代表这是一个ARP请求报文。
     - **发送方MAC地址**（Sender MAC Address）：在ARP响应中，发送方的MAC地址为`0002.1722.CDCA`，表示PC1的物理地址。
     - **发送方IP地址**（Sender IP Address）：与发送方MAC地址相对应的IP地址为`192.168.2.13`。
     - **目标MAC地址**（Target MAC Address）：由于是响应，这里填充了发起ARP请求的设备的MAC地址，即`0000.0000.0000`，因为这是一个ARP请求，发送方正在查询这个MAC地址。
     - **目标IP地址**（Target IP Address）：请求ARP解析的IP地址为`192.168.2.1`，请求相应的设备。
   - **输出PDU详情**：详细描述了ARP响应的格式。
     - **硬件类型**（Hardware Type）：在这个ARP响应中，硬件类型为`0x1`，指示这是一个以太网帧。
     - **硬件长度**（HLEN, Hardware Length）：为6，表示物理地址（MAC地址）的长度为6个字节。
     - **协议长度**（PLEN, Protocol Length）：为4，表示协议地址（IP地址）的长度为4个字节。
     - **操作码**（Opcode）：为`0x2`，代表这是一个ARP响应报文。
     - **发送方MAC地址**（Sender MAC Address）：在ARP响应中，发送方的MAC地址为`00E0.8FAC.8302`，表示Router1右端口的物理地址。
     - **发送方IP地址**（Sender IP Address）：与发送方MAC地址相对应的IP地址为`192.168.2.1`。
     - **目标MAC地址**（Target MAC Address）：由于是响应，这里填充了发起ARP请求的设备的MAC地址，即`0002.1722.CDCA`。
     - **目标IP地址**（Target IP Address）：请求ARP解析的IP地址为`192.168.2.13`，对应于发起请求的设备。

2. **本机的ARP内容**

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231127210145359.png" alt="image-20231127210145359" style="zoom:25%;" />

   3. **对WireShark抓取ARP报文进行分析**

      > 我在终端输入了`ping 100.80.165.195`

      <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231127211851714.png" alt="image-20231127211851714" style="zoom: 25%;" />

      ARP请求包含以下信息：

      - Hardware type使用的硬件类型：这里的1代表以太网。
      - Protocol type协议类型：0x0800是IPv4地址。
      - Hardware size硬件地址的长度：这里是6，对应于MAC地址的字节长度。
      - Protocol size协议地址的长度：这里是4，对应于IPv4地址的字节长度。
      - Opcode操作码：1代表这是一个ARP请求。
      - Sender MAC address发送者的MAC地址：与上面的Src字段相同。
      - Sender IP address发送者的IP地址：这里是100.80.165.198。
      - Target MAC address目标MAC地址：在ARP请求中这通常是0，因为这是正在查询的地址。
      - Target IP address目标IP地址：这里是100.80.165.195。发送者想要知道这个IP地址对应的MAC地址。

      

## 分析讨论 

1. **为什么需要一开始要先清除ARP缓存才能访问到ARP包？**

   因为ARP包只会在以下情况下发出，一开始Cisco会自动帮助发送ARP，这时再simulation则无法接收到。

   1. 新设备加入网络：当一台设备首次尝试与网络上的另一台设备通信时，它需要将目标设备的IP地址解析为MAC地址，这时会发出ARP请求。
   2. ARP缓存中没有条目：如果一个设备的ARP缓存中没有目标IP地址的条目，或者条目已经过期，它会发出ARP请求来获取或更新这个IP地址对应的MAC地址。
   3. 网络通信：任何时候设备尝试与同一局域网(LAN)上的另一个设备通信，但不知道其MAC地址时，会发出ARP请求。
   4. IP地址冲突：如果一个设备怀疑有IP地址冲突，即可能有另一个设备使用了相同的IP地址，它可能会发送ARP请求来确认。
   5. IP地址更改：当网络上的设备更改其IP地址时，它可能会发送ARP通告，通知网络上的其他设备它的新MAC-IP映射。
   6. ARP缓存超时：大多数操作系统都会定期清除ARP缓存中的旧条目，如果之后还需要旧的IP-MAC映射关系，系统就会发送新的ARP请求。
   7. 网络设备重启或连接恢复：当网络设备重启或从断开状态恢复连接时，它们通常会发送ARP请求，以确保它们在ARP缓存中有最新的网络连接信息。
