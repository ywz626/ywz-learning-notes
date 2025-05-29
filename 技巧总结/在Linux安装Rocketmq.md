# 1. 先安装jdk



![image-20250528130900772](image-20250528130900772.png)

用这个命令解压Java

```
export JAVA_HOME=/app/java/jdk8u452-b09

export JRE_HOME=/app/java/jdk8u452-b09/jre

export ROCKETMQ_HOME=/app/Rocketmq/rocketmq-all-5.3.1-bin-release

export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib

export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$ROCKETMQ_HOME/bin:$PATH:$HOME/bin

export NAMESRV_ADDR=192.168.70.132:9876
```

在/etc/profile中配置环境

java压缩包在这里E:\edgedownload

# 2.在安装rocketmq

正常解压然后配置环境变量

# 3. 然后启动NameServer

/app/Rocketmq/rocketmq-all-5.3.1-bin-release/bin 在这个目录中启动
