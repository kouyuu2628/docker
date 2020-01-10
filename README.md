# centos7 docker install

- 查看linux发行版，内核
```
[root@localhost ~]# cat /etc/redhat-release  #查看版本号
CentOS Linux release 7.7.1908 (Core)
[root@localhost ~]# uname -r  #查看Linux内核
3.10.0-1062.9.1.el7.x86_64
```
- 替换阿里云yum源
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo #下载阿里yum源2
yum makecache  #生成仓库缓存
```
- 安装docker
```
yum install docker -y
```
- 启动docker
```
systemctl start docker

```
- 查看docker 版本
```
[root@localhost ~]# docker version
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: docker-1.13.1-103.git7f2769b.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      7f2769b/1.13.1
 Built:           Sun Sep 15 14:06:47 2019
 OS/Arch:         linux/amd64

Server:
 Version:         1.13.1
 API version:     1.26 (minimum version 1.12)
 Package version: docker-1.13.1-103.git7f2769b.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      7f2769b/1.13.1
 Built:           Sun Sep 15 14:06:47 2019
 OS/Arch:         linux/amd64
```
- 使用国内镜像加速
```
vi /etc/docker/daemon.json #修改这个文件为如下内容

{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/","https://hub-mirror.c.163.com","https://registry.docker-cn.com"],
"insecure-registries": ["10.0.0.12:5000"]
}
#事后重启守护进程并重启docker
systemctl daemon-reload
systemctl restart docker
```

## 其他命令
- 开机启动docker
```
systemctl enable docker
```
- 查看docker状态
```
systemctl status docker -l
```
- 重启docker服务
```
systemctl restart docker
#或者
service docker restart
```
- 关闭docker
```
systemctl stop docker
#或者
service docker stop
```
- 查看docker的系统信息
```
docker info
```
- 查看docker的镜像
```
docker images
```
- 删除镜像
```
docker rmi 镜像id
```
- 查看当前所有正在运行的容器
```
docker ps
```
- 查看全部容器
```
docker ps -as
```
- 关闭容器
```
docker stop 容器id
```
- 删除docker中的容器
```
docker rm 容器id
```
- 创建自定义网络
```
docker network create --subnet=172.18.0.0/16 docker-diy-network
```
- 查看docker网络
```
docker network ls
```
## docker容器入门
https://www.jianshu.com/p/4c5a251e0768