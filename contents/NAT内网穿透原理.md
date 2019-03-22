---
title: NAT 内网穿透 原理
date: 2018-10-12 17:37:20
tags: 大前端实录
---


## 一、什么是NAT？


> `NAT`（`Network Address Translation`，网络地址转换）是1994年提出的。当在专用网内部的一些主机本来已经分配到了本地IP地址（即仅在本专用网内使用的专用地址），但现在又想和因特网上的主机通信时，可使用NAT方法。

`NAT`是一个`IETF`(`Internet Engineering Task Force`, Internet工程任务组)标准，允许一个整体机构以一个公用`IP`（`Internet Protocol`）地址出现在Internet上。

**听上去似乎很简单，NAT就是替换了一下IP地址**

### RFC1918规定了三块专有的地址，作为私有的内部组网使用：

* A类：10.0.0.0—10.255.255.255            10.0.0.0/8
* B类：172.16.0.0—172.31.255.255        172.16.0.0/12
* C类：192.168.0.0—192.168.255.255    192.168.0.0/16

这些私有IP地址只在企业内部使用，不能作为全球路由地址，除了企业或组织的管理范围，这类私有地址就不在有任何意义。

#### 注意：任何一个组织都可以在内部使用这些私有地址，因此两个不同网络中存在相同IP地址的情况是很可能出现的，但是同一个网络中不允许两台主机拥有相同IP地址，否则将发生地址冲突。

这三块私有地址本身是可路由的，只是公网上的路由器不会转发这三块私有地址的流量；当一个公司内部配置了这些私有地址后，内部的计算机在和外网通信时，公司的边界路由会通过NAT或者PAT技术，将内部的私有地址转换成外网IP，外部看到的源地址是公司边界路由转换过的公网IP地址，这在某种意义上也增加了内部网络的安全性。


### NAT的三种类型

* Static NAT 
* Pooled NAT
* NAPT
	* SNAT
	* DNAT

>  有了NAT以后，内网的主机不在需要申请公网IP地址，只需要将内网主机地址和端口通过NAT映射到网络出口的公网IP即可，然后通信的两端在无感知的情况下进行通信。这也是为什么说NAT挽救了IPV4，因为大量的内网主机有了NAT，只需要很少的公网地址做映射就可以了，如此就可以节约出很多的IPV4地址空间。


### 1.1 Static NAT将内部网络的私有IP地址转换为公有IP地址，IP地址对是一对一的，是一直不变的。

<img src="http://p4yixtz2j.bkt.clouddn.com/1539067403.png" width = 600/>

### 1.2 Pooled NAT 

动态NAT 是多对多的。将内部网络的私有IP地址转换为公用IP地址时，IP地址是不确定，随机的。所有被授权访问Internet的私有IP地址可随机转换为任何指定合法的IP地址。也就是说，只要指定哪些内部地址可以进行转换，以及用哪些合法地址作为外部地址时，就可以进行动态NAT转换。动态NAT是在路由器上配置一个外网IP地址池，当内部有计算机需要和外部通信时，就从地址池里动态的取出一个外网IP，并将他们的对应关系绑定到NAT表中，通信结束后，这个外网IP才被释放，可供其他内部IP地址转换使用，这个DHCP租约IP有相似之处。当ISP提供的合法IP地址略少于网络内部的计算机数量时。可以采用动态转换的方式。

<img src="http://p4yixtz2j.bkt.clouddn.com/1539067703.png" width = 600/>

### 1.3 NAPT（Network Address Port Translation）

网络地址端口转换是多对一的。改变外出数据包的源端口并进行端口转换，采用端口多路复用方式。内部网络的所有主机均可共享一个合法外部IP地址实现对Internet的访问，可以最大限度地节约IP地址资源。同时，也可以隐藏网络内部的所有主机，有效避免来自Internet的攻击。因此，目前网络中应用最多的就是PAT规则。这是最常用的NAT技术，也是IPv4能够维持到今天的最重要的原因之一，它提供了一种多对一的方式，对多个内网IP地址，边界路由可以给他们分配一个外网IP，利用这个外网IP的不同端口和外部进行通信。NAPT与 动态NAT 不同，它将内部连接映射到外部网络中的一个单独的IP地址上，同时在该地址上加上一个由NAT设备选定的端口号。

<img src="http://p4yixtz2j.bkt.clouddn.com/1539067843.png" width = 600/>

`NAPT`是使用最普遍的一种转换方式，在`HomeGW`中也主要使用该方式。

* ``源NAT`（`Source NAT`，`SNAT`）：修改数据包的源地址。源`NAT`改变第一个数据包的来源地址，它永远会在数据包发送到网络之前完成，数据包伪装就是一个`SNAT`的例子。
 
* `目的NAT`（`Destination NAT`，`DNAT`）：修改数据包的目的地址。`Destination NAT`刚好与`SNAT`相反，它是改变第一个数据解析的目的地地址，如平衡负载、端口转发和透明代理就是属于`DNAT`。

## 二、什么是内网穿透？

**NAT的缺陷之一就是只能由内网主机发起连接，外网主机无法主动连接到内网。** 这就意味着外部节点无法和内网主机进行P2P通信

### 内网穿透的方式有哪些？

### 2.1 STUN——UDP打洞

全称为`Simple Tranversal of UDP through NAT`   

<img src="http://p4yixtz2j.bkt.clouddn.com/1539069227.png" width = 600/>

假设两个不同网络中的设备A和B想穿透NAT进行点对点通信，其中STUN SERVER是部署在公网中的STUN服务器。    

1. A通过NAT网关向SERVER发送STUN请求消息(UDP)，查询并注册自己经过NAT映射后的公网地址  
2. SERVER响应，并将A经过转换后的公网IP地址和端口填在响应报文中  
3.  B通过NAT网关向STUN SERVER发送STUN请求消息(UDP)，查询并注册自己经过NAT映射后的公网地址
4.  SERVER响应，并将B经过转换后的公网IP地址和端口填在响应报文中
5.   此时A已经知道了自己映射后对应的公网IP地址和端口号，它把这些信息打包在请求中发送给SERVER，请求和B进行通信
6.    SERVER查询到B注册的公网地址和端口，然后将请求通过NAT网关转发给B
7.     B从消息中知道A的公网地址和端口，于是通过此地址和端口，向A发送消息，消息中包含B映射后的公网地址和端口号，A收到消息后就知道了B的公网地址及端口，这样在A和B之间建立起了通信通道。

###  2.2 TURN    
STUN穿透技术的缺点在于无法穿透对称型NAT，这可以通过TURN技术进行改进。TURN的工作过程和STUN非常相似。

**区别在于在TURN中，公网地址和端口不由NAT网关分配，而是由TURN服务器分配。**   

由于所有的请求都需要经过TURN服务器，所以网络延迟较大。

目前一些适用于Windows系统上的网络穿透工具普遍使用的是这种方式

* 花生壳
* Ngrok
* Natapp
* Frp
* Lanproxy
* Spike

花生壳的原理如下：

<img src="http://p4yixtz2j.bkt.clouddn.com/1539069969.png" width = 600/>

在花生壳端口映射配置页面配置好映射端口


<img src="http://p4yixtz2j.bkt.clouddn.com/1539074278.png" width = 700/>

内网穿透起到了地址转换的功能，也就是把公网的地址进行翻译，转成为一种私有的地址。

<img src="http://upload-cdn.oray.com/upload/help/1806/201806211156382125.jpg" width = 600/>

可以看到，花生壳有一个公网服务器，这个服务器就是这里所谓的`TURN SERVER` 先向这个服务器注册号自己的内网端口号和内网主机号，然后再本地开启花生壳App。

<img src="http://p4yixtz2j.bkt.clouddn.com/1539073721.png" width = 800/>

从上图中可以看出真个TURN内网穿透的流程，花生壳App通过一个动态的端口向`TURN SERVER` （103.46.128.49）发起了请求，建立了连接，然后本地自己建立了一个进程，这个进程所监听的端口正是我们在花生壳的端口配置界面所写入的那个端口。这个进程就是内网服务器。

之后外网的某个客户端想要访问内网进程，就需要去和`TURN SERVER` 发起连接，这样花生壳App和外网客户端的连接就通过公网服务器连接了起来，之间发送的所有数据都要通过公网服务器。

**那么为什么内网服务器可以收到外网客户端的数据呢？**

由于花生壳App和内网进程在同一个局域网下，所以外网客户端发送的数据，最终通过公网服务器流向花生壳App，再由花生壳App发送给内网服务器进程。

###  2.3 UPnP

`UPnP`(`Universal Plug and Play`)意为通用即插即用协议，是由微软提出的一种`NAT`穿透技术。使用UPnP需要内网主机、网关和应用程序都支持`UPnP`技术。   

`UPnP`通过网关映射请求为客户分配映射表项，而NAT网关只需要执行地址和端口的转换。UPnP客户端发送到公网侧的信令或者控制消息中，会包含映射之后公网IP和端口，接收端根据这些信息就可以建立起P2P连接。    

#### UPnP穿透的过程大致如下：

###  2.3.1 发送查找消息：    
一个设备添加到网络以后，会多播大量发现消息来通知其嵌入式设备和服务，所有的控制点都可以监听多播地址以接收通知，标准的多播地址是239.255.255.250：1900。可以通过发送http请求查询局域网中upnp设备，消息形式如下：    


```xml
<xml>
    M-SEARCH * HTTP/1.1 \r\n    
    HOST 239.255.255.250:1900 \r\n    
    ST:UPnP rootdevice \r\n    
    MAN:\"ssdp:discover\" \r\n    
    MX:\r\n\r\n 
</xml>
```
   


###  2.3.2 获得根设备描述url    
如果网络中存在upnp设备，此设备会向发送了查找请求的多播通道的源IP地址和端口发送响应消息，其形式如下：    

```xml
<xml>
    HTTP/1.1 200 OK    
    CACHE_CONTROL: max-age=100    
    DATE: XXXX    
    LOCATION:http://192.168.1.1:1900/igd.xml    
    SERVER: TP-LINK Wireness Router UPnP1.0    
    ST: upnp:rootdevice  
</xml>
```

首先通过200 OK确定成功的找到了设备。然后要从响应中找到根设备的描述URL（例如上面响应报文中的`http://192.168.1.1:1900/igd.xml`），通过此URL就可以找到根设备的描述信息，从根设备的描述信息中又可以得到设备的控制URL，通过控制URL就可以控制UPNP的行为。上面这个响应中表示我们在局域网中成功的找到了一台支持UPNP的无线路由器设备。    

###  2.3.3 通过 设备描述URL 的地址得到设备描述URL得到XML文档

发送HTTP请求消息：

```xml
<xml>
    GET /igd.xml HTTP/1.1 
    HOST: 192.168.1.1:1900 
    Connection: Close
</xml>    
```

然后就能得到一个设备描述文档，从中可以找到服务和UPnP控制URL。每一种设备都有对应的`serviceURL`和`controlURL`。其中和端口映射有关的服务是`WANIPConnection`和`WANPPPConnection`。

###  2.3.4 进行端口映射

   拿到设备的控制URL以后就可以发送控制信息了。每一种控制都是根据HTTP请求来发送的，请求形式如下：    

```xml
<xml>
    POST path HTTP/1.1    
    HOST: host:port    
    SOAPACTION:serviceType#actionName    
    CONTENT-TYPE: text/xml    
    CONTENT-LENGTH: XXX    
    ....
</xml>   
```

其中path表示控制url，host:port就是目的主机地址，actionName就是控制upnp设备执行响应的指令。UPNP支持的指令如下：

| actionName | 描述 | 
| ------ | ------ |
| GetStatusInfo| 查看UPNP设备状态 | 
| AddPortMapping | 添加一个端口映射|
|DeletePortMapping|删除一个端口映射|
|GetExternalIPAddress|查看映射的外网地址|
|GetConnectionTypeInfo|查看连接状态|
|GetSpecificPortMappingEntry|查询指定的端口映射|
|GetGenericPortMappingEntry|查询端口映射表|

常我们需要用到的是AddPortMapping进行端口映射，以及GetExternalIPAddress获取到映射的公网地址。

## 三、手动操作路由器实现内网穿透

家用的 宽带光猫+路由器 分配主机IP的模式可以通过手动添加端口映射的方式来实现内网穿透

### 访问路由器的IP地址来查看路由器的NAT端口映射

<img src="http://p4yixtz2j.bkt.clouddn.com/1539058327.png" width = 600/>

上面这个页面定义了内部监听的端口号：40404和
外网需要访问时连接的端口号：50505


<img src="http://p4yixtz2j.bkt.clouddn.com/1539078050.png" width = 600/>

这里是宽带光猫的配置页面，需要注意的是填写好一对一地址转换这里：

* 外网IP地址：即为电信运营商分配的家庭公网IP地址
* 内网IP地址：即光猫为路由器分配的内网地址

在添加好这两个地方之后 一般的家庭路由器也能够实现内网穿透了 不需要借助其他工具，但是需要注意的是必须要路由器支持NAT服务。



