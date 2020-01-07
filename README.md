# CENTOS7 DOCKER INSTALL  

## 查看linux发行版，内核
```
[root@localhost ~]# cat /etc/redhat-release  #查看版本号
CentOS Linux release 7.7.1908 (Core)
[root@localhost ~]# uname -r  #查看Linux内核
3.10.0-1062.9.1.el7.x86_64
```
## 替换阿里云yum源
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo #下载阿里yum源2
yum makecache  #生成仓库缓存
```
## 安装docker
```
yum install docker -y
```
## 启动docker
```
systemctl start docker  #启动docker
systemctl enable docker #开机启动docker
systemctl status docker #查看docker状态
```
## 查看docker 版本
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
## DaoCloud 加速器
DaoCloud 加速器 是广受欢迎的 Docker 工具，解决了国内用户访问 Docker Hub 缓慢的问题。DaoCloud 加速器结合国内的 CDN 服务与协议层优化，成倍的提升了下载速度。
> 使用前请先确保您的 Docker 版本在 1.8 或更高版本，否则无法使用加速。
```
vi /etc/docker/daemon.json #修改这个文件为如下内容

{
    "registry-mirrors": [
        "http://95822026.m.daocloud.io"
    ],
    "insecure-registries": []
}

#事后重启docker
systemctl restart docker
```