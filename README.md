# 包含漏洞的应用程序
### 服务器搭建：
![image](https://user-images.githubusercontent.com/45362268/113259098-55e02200-92ff-11eb-96ce-d472d34ab3bc.png)
        
1. 除跳板主机，其余都在内网内。主机A拥有两个地址，一个外网地址，一个内网地址。其余的主机都只有内网地址。
2. 内网中有两台虚拟网络设备，一个是虚拟路由器，负责内网流量的路由。一个是NAT网关，它拥有外网地址，负责将路由过来的外网请求进行网络地址转换。内网由此和外网隔离起来，外部无法主动访问内网主机。
3. 特别地，我们还可以看到，主机A内运行了docker容器服务，而且wordpress站点是部署在docker中的，可以由外界访问。但我们在主机A的iptables 配置中，阻断了docker网段对内网的访问。因此就算wordpress站点被攻击，危害仍然只是会被限制到docker 容器内部，而无法进一步渗透进内网。

### 攻击设计
基于此，我们设计的最终目标是拿到内网中Jenkins服务器的Flag。攻击路径如下：
1. 攻击机利用CVE-2019-17571，拿下跳板主机，此后所有的访问和攻击流量均由跳板主机发出，以隐藏攻击者本身. 
2. 利用CVE-2020-12800拿下容器内wordpress站点的www-data执行权限
3. 利用CVE-2019-18634提权至root
4. 利用CVE-2019-5736从容器中逃逸，获取A主机的root执行权限，并以A主机为第二跳板，进行内网的渗透
5. 利用CVE-2019-10392成功到Jenkins服务器上的flag

![image](https://user-images.githubusercontent.com/45362268/113259535-d141d380-92ff-11eb-951f-cedef876711a.png)

### 防御设计
防御主要集中在主机A上，有两个点，第一是容器的安全性，第二是主机的安全性：
1. 主机A上部署了Snort入侵行为检测系统，用于保护wordpress站点的安全性。
2. 内网中部署了完整性检测系统OSSEC，这是一套C/S架构的系统，服务端部署在内网的一个单独主机上，客户端部署在主机A上。用于保护主机A的主机安全。
