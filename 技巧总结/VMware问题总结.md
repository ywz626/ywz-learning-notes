# VMware克隆虚拟机配置

**这里以centos为例**

**1.首先进行“完全克隆”，注意：要克隆的虚拟机在克隆前是需要处于关闭状态。**

选择要克隆的虚拟机右键，选择“管理”，然后选择“克隆”。

![img](https://img2024.cnblogs.com/blog/2080940/202412/2080940-20241217165536363-954054039.png)

然后直到这一步选择“完整克隆”。

*注意：链接克隆是指在一些资源上两个虚拟机会共用；完整克隆是完全独立出来的一个新虚拟机。*

![img](https://img2024.cnblogs.com/blog/2080940/202412/2080940-20241217165604932-511727913.png)

然后下一步，虚拟机名称输入你想要的名称就完成了。

![img](https://img2024.cnblogs.com/blog/2080940/202412/2080940-20241217165644709-2117779944.png)

注意，克隆完之后所有信息与原虚拟机一样，所以下面我们进行一些信息的修改

 

**2.开机前修改mac地址（不用修改也行，VMware好像会自动变）**

（注意：如果是动态生成IP地址，请在启动前先启动被克隆的虚拟机，以保证原来虚拟机的ip不会变，否则原来虚拟机ip会变，克隆后的虚拟机ip是原来的虚拟机ip）

点击编辑虚拟机设置,选择网络适配器，点击右下角高级然后下边就是mac地址，这个mac地址和被克隆的是一样的，我们点击生成，重新生成一个新的：

![img](https://img2024.cnblogs.com/blog/2080940/202412/2080940-20241217165714484-660724238.png)

然后mac地址就修改完了。

 

**3.开机后修改主机名称 \**（多台机器搭建分布式应用时，必须修改）\**

**

修改主机名可能不同linux版本不同，修改方法也不同。

centos7通过vi /etc/hostname 命令来编辑主机名。

```
vi  /etc/hostname
```

node1 

 

**4.修改IP地址（多台机器搭建分布式应用时，必须修改）**

此处需要注意的是：如果虚拟机使用的是动态ip分配，那么不需要更改ip，如果想改为静态ip，请修改：

```
vim  /etc/sysconfig/network-scripts/ifcfg-ens33
```

IPADDR="192.168.80.132"  

只需要修改IPADDR这一样，将IP地址 

 

**5. 修改UUID（不用修改也行，但是为了避免出现冲突，尽量修改下）**

首先通过uuidgen命令生成一个uuid。

```
uuidgen
```

bed247ea-9054-48d4-bea7-28f7aaaee796

然后同修改IP一样，修改配置文件：

```
vim  /etc/sysconfig/network-scripts/ifcfg-ens33
```

UUID="bed247ea-9054-48d4-bea7-28f7aaaee796"

将上面生成的uuid拷贝并粘贴到UUID这里。

**5. 最后重启network**

```
systemctl restart network

```

# vmware克隆虚拟机后被克隆机网络哦无法使用

1.   重新生成mac地址

2.   接下来删掉设备管理器下的70-persistent-net.rules文件,此文件删除重启后会自动生成.

     \#rm -rf /etc/udev/rules.d/70-persistent-net.rules

# VMware linux虚拟机固定IP

### 1、通过vmware看网关

![img](https://img2022.cnblogs.com/blog/1823641/202203/1823641-20220320234218059-257243840.png)

![img](https://img2022.cnblogs.com/blog/1823641/202203/1823641-20220320234310350-1816532375.png)

### 2、查看当前使用的网络

```
[root@localhost ~]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.80.125  netmask 255.255.255.0  broadcast 192.168.80.255
        inet6 fe80::20c:29ff:feac:530b  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:ac:53:0b  txqueuelen 1000  (Ethernet)
        RX packets 148  bytes 15236 (14.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 112  bytes 13801 (13.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

当前网络名称为：ens33

### 3、修改网络配置

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

ens33 根据第二步找到网络名称替换下面的

```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO=static
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
NAME="eno1"
UUID="b4fa2dd3-902b-4294-b1c5-a829ddd2542e"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.80.80"
PREFIX="24"
GATEWAY="192.168.80.2"
DNS1="192.168.80.2"
```

```
BOOTPROTO=static   修改为 static 就是静态地址
DEVICE="ens33"  修改为第二步对应的名称
IPADDR="192.168.80.80" 为你希望让linux虚拟机使用的ip
GATEWAY="192.168.80.2"  修改为第一步查看到的网关ip
DNS1="192.168.80.2"  修改为第一步查看到的网关ip
```

保存配置

### 4、重启网络

```
systemctl restart network
```

![img](https://img2022.cnblogs.com/blog/1823641/202203/1823641-20220320235230925-193406281.png)

