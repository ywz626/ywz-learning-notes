### 首先检查配置文件的服务器地址

输入vi /etc/resolv.conf

将地址改为8.8.8.8和8.8.4.4

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/db161feafcf9653b4c1f81fa74ef7fa6.png)

### 然后将yum的源改成阿里云（或者163等源）

直接输入[curl](https://so.csdn.net/so/search?q=curl&spm=1001.2101.3001.7020) -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

CentOS6改的话直接将7改为6即可

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c4c964da8c0a7446dd4a9266a601e6b9.png)

安装完成以后yum就可以正常使用了

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/49f669d8b400191eabae0a335f9a82f1.png)

之后运行一下 yum clean all

 yum makecache 生成缓存

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5877e7ea7eebad9be48b93f18c7a3730.png)

此时可以通过 yum repolist 来查询yum的状态了

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/4f5d6d252c886eb39bf89bbe582bfb06.png)