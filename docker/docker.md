### Docker简介
docker 是一个开源的应用容器引擎，一个完整的docker有以下几部分组成：dockerClient客户端、dockerDaemon守护进程、dockerImage镜像、dockerContainer容器。  
UnionFS(联合文件系统)：是一种分层、轻量级并且高性能的文件系统，他支持对文件系统的修改作为一次提交来一层层叠加。
### docker命令  
```
docker system df  查看容器/镜像/数据卷所占用的空间  
docker run --name 指定名字
            -d 后台运行并返回容器id  
            -i 已交互模式运行容器  
            -t 为容器分配一个伪终端  
            -p 分配端口    
docker log 查看日志
docker top 查看容器内运行进程
docker inspect 查看容器内部细节
docker exec 进入容器
docker attach 进入容器 直接进入容器启动命令的终端，不会启动新的进程，使用exit退出，会导致容器的停止。
docker cp 容器id：容器内路径  目的路径  拷贝文件
docker export 容器id > abc.tar 将容器导出成tar包
cat abc.tar | docker import - 镜像用户/镜像名：镜像版本号  将tar包导入成容器
``` 
