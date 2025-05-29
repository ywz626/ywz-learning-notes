问题出现原因：

不能访问互联网要修改 /etc/sysconfig/network-scripts / ifcfg-ens33文件，

开始时登录账户为非root账户，在修改ifcfg-ens33文件后进行保存时提示为只读型不能保存

解决方法：

开始时使用sudo vim  ifcfg-ens33 命令编辑，提示权限不足

切换为root用户 su root 然后执行 sudo vim  ifcfg-ens33 可以进入编辑，编辑后ESC+：WQ保存成功

 

遇见错误警告：

Can't open file for writing ：权限不足，使用sudo

XXXX  is not in the sudoer file :权限不足，切换root账户