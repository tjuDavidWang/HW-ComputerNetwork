# DNS实验

2023/12/18

## 实验目的

本实验旨在通过对DNS（域名系统）的深入理解和分析，掌握其基本原理和工作过程。实验目的是使学生能夠了解DNS解析域名到IP地址的过程，理解DNS服务的体系结构，以及掌握DNS域名解析的顺序。

## 实验原理 

DNS是一种组织成域层次结构的计算机和网络服务命名系统。它用于TCP/IP网络，主要功能是将主机名和域名转换为IP地址。DNS服务基于应用层协议工作，为多种应用层协议（如HTTP、SMTP和FTP）提供主机名到IP地址的解析。实验中将深入探讨DNS的工作过程、服务体系架构、分布式集群工作方式以及域名称空间的组织方式。

## 实验设备 

- 操作系统：Windows 11
- 网络环境：局域网
- 软件：Cisco Packet Tracer虚拟实验环境

## 实验步骤

1. 打开Cisco Packet Tracer虚拟实验环境，按照下面的网络结构图连线

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231204142448659.png" alt="image-20231204142448659" style="zoom: 33%;" />

2. 为每个PC配置各自的静态IP，在 Server2、3 上配置 Gateway/IP Address/Subnet Mask（可以使用DHCP，按照上面的进行配置）

3. 将PC2的DNS设置得与Server3的DNS相同，以便能够通过Server3构建映射，访问到Server2

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231218141737751.png" alt="image-20231218141737751" style="zoom: 25%;" />

4. 在Server3上配置DNS，将域名设置为`www.tongji.edu.cn`，Detail处写映射到Server2的IP地址（`192.168.2.13`）

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231218141801736.png" alt="image-20231218141801736" style="zoom:33%;" />

5. 用Simulation模式打开，打开PC3的Web Browser，输入配置web服务器的域名（`http://www.tongji.edu.cn`），产生DNS数据报，<u>分析DNS报文情况</u>

   <img src="C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231218141851568.png" alt="image-20231218141851568" style="zoom:33%;" />

6. 用WireShark捕获这些DNS数据包，<u>查看DNS报文字段内容，并解读</u>

## 实验现象

1. 在Packet tracer中DNS报文情况

   选取从Switch1 -> Server3这一段进行分析

   ![image-20231218142508762](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231218142508762.png)

   - DNS头部信息

     DNS头部包含了多个字段，其中每个字段都有其特定的含义和作用：
   
     - **事务ID（ID）**: 这是由客户端生成的一个随机数，用于匹配DNS请求和响应。这样客户端可以识别并关联响应与它的原始查询。
     - **标志位（Flags）**: 包括多个部分，如QR（查询/响应标志），Opcode（操作码），AA（授权应答），TC（截断），RD（期望递归），RA（递归可用），以及Rcode（响应代码）。每个标志位提供了关于查询或响应特性的不同信息。
     - **问题计数（QDCOUNT）**: 表示查询中问题的数量。在这里，它是1，意味着这个DNS请求只查询了一个域名。
   
   - DNS应答部分
   
     应答部分是响应报文中对应DNS查询的部分。这里的每个字段都与域名查询的结果密切相关：
   
     - **NAME**: 这是客户端请求解析的域名，这里是`www.tongji.edu.cn`。DNS服务器需要在其记录中查找这个特定的域名，并返回相应的IP地址。
     - **TYPE**: 查询类型字段指示了客户端请求的DNS记录类型。TYPE为0x0001代表这是一个A记录查询，客户端期望得到域名对应的IPv4地址。
     - **CLASS**: 表示查询的类别，这里是0x0001，代表IN（Internet），说明这是一个标准的互联网DNS查询。
     - **TTL（Time To Live）**: 这个字段指定了DNS记录的生存时间。86400秒表示，一旦这个DNS记录被DNS解析器缓存，它将在缓存中保留24小时，除非被提前清除。
     - **LENGTH**: 这个字段通常表示应答资源记录中数据字段的长度。在这个例子中，LENGTH为0，这通常表明响应中没有包含域名对应的IP地址。这可能是因为请求的域名在服务器的DNS记录中不存在，或者DNS服务器没有配置正确的转发或根提示。
   
2. 分析WireShark抓取的DNS数据报

   ![image-20231218143807555](C:\Users\12920\AppData\Roaming\Typora\typora-user-images\image-20231218143807555.png)

   - 事务ID (Transaction ID): 0xd9ef3
   
   - 标志 (Flags): 0x0100 标准查询 (Standard query)，表明这是一条标准DNS查询消息。消息不是响应，它是一个查询请求。查询操作码为标准查询。消息没有被截断。客户端希望DNS服务器递归查询。
   
   - 问题数 (Questions): 1
   
   - 回答资源记录数 (Answer RRs): 0，表示响应中没有包含答案资源记录，表示没有返回与查询域名相对应的IP地址。
   
   - 授权资源记录数 (Authority RRs): 0，表示响应中没有授权资源记录，表示没有返回权威DNS服务器的信息。
   
   - 附加资源记录数 (Additional RRs): 0，表示响应中没有附加资源记录，表示没有额外的资源记录信息。
   
   - 查询 (Queries)：
   
     - 名称 (Name): 查询的域名是`1.tongji.edu.cn`。
   
     - 类型 (Type): A (1) 表示请求的是主机地址，即IPv4地址。
   
     - 类 (Class): IN (0x0001) 表示查询的类别是互联网（IN）。
   
   - 响应编号 (Response In): 2423，这表明响应可以在Wireshark捕获的第2423号帧中找到。

## 分析讨论 

DNS（域名系统）是互联网上不可或缺的一部分，它负责将用户友好的域名转换为机器可识别的IP地址。在我们的实验中，我们通过Wireshark捕获并分析了DNS查询的过程，这一过程是网络浏览中的第一步，即在TCP连接和HTTP请求之前。实验结果显示，如果DNS查询无法返回正确的IP地址，即使网络本身没有问题，用户也无法访问目标网站。因此，DNS的高效运作对整个网络访问流程至关重要。通过这种方式，DNS作为网络通信的桥梁，与TCP和HTTP协议一起，确保了用户能够顺畅地访问和交互互联网上的内容。
