**vim 编辑器的下载与安装** 

在使用 docker容器时，有时候里边没有安装vim，运行vim命令时提示说：vim: command not found，这个时候就需要安装vim，当运行以下安装命令时：

代码语言：txt

AI代码解释



```txt
apt-get install vim 
```

Error报错提示：

代码语言：txt

```txt
Reading package lists… Done 
Building dependency tree 
Reading state information… Done
E: Unable to locate package vim 
```

根据 apt 命令下载并安装软件的惯例，我们需要对 apt进行更新！！！

代码语言：txt

```txt
apt-get update 
```

上述命令是为了同步  /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，这样才能获取到最新的软件包。

等更新完毕以后再运行命令： 

代码语言：txt



```txt
apt-get install vim 
```

代码语言：txt



```txt
vim /etc/mysql/mysql.conf.d/mysqld.cnf 
```

1.apt-get install vim

![img](https://ask.qcloudimg.com/http-save/yehe-8572099/o0j3uh90tl.png)

2.apt-get update

![img](https://ask.qcloudimg.com/http-save/yehe-8572099/i47epvxmj4.png)

3.完成 vim 编辑器安装

![img](https://ask.qcloudimg.com/http-save/yehe-8572099/w072qspfz6.png)