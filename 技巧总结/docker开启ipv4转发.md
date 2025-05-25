<font color=red>**问题报错**</font>

```
WARNING: IPv4 forwarding is disabled. Networking will not work.
```

问题原因
是没有开启转发,docker网桥配置完后，需要开启转发，不然容器启动后，就会没有网络，配置/etc/sysctl.conf,添加net.ipv4.ip_forward=1

- **问题解决：**

修改文件

```
vim /etc/sysctl.conf

net.ipv4.ip_forward=1    #添加此行配置
```

注：也可修改此文件：/usr/lib/sysctl.d/00-system.conf

重启[network](https://so.csdn.net/so/search?q=network&spm=1001.2101.3001.7020)和docker服务

```
systemctl restart network && systemctl restart docker
```

查看是否修改成功

```
sysctl net.ipv4.ip_forward
```

如果返回为“net.ipv4.ip_forward = 1”则表示修改成功

再次执行查看，使用docker不再报错