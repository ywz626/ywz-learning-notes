- docker pull 。。。。从远程拉取镜像
- docker run 。。。运行镜像 -d 后台运行 –name指定容器名  -p xxxx:yyyy指定端口映射
- docker stop 容器名 # 停止
- docker  restart  重启
- docker rm -f  容器名  强制删除运行中的容器
- docker ps  # 仅显示运行中的容器
  docker ps -a  # 显示所有容器（包括已停止）
- docker exec -it my_container /bin/bash  # 交互式操作
- docker pull nginx:1.23  # 指定版本
- docker rmi nginx:1.23  # 按标签删除
  docker rmi $(docker images -aq)  # 删除所有镜像

