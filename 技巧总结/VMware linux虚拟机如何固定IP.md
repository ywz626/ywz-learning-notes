# <font color=yellow>VMware linux虚拟机如何固定IP</font>

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