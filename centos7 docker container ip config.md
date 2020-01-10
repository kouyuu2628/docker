# centos7中docker容器的ip配置

## 创建子网
```
docker network create --subnet=172.18.0.0/24 my-docker-net
```
> 掩码用24(255.255.255.0),与宿主机掩码一致
```
Error response from daemon: Failed to Setup IP tables: Unable to enable SKIP DNAT rule:  (iptables failed: iptables --wait -t nat -I DOCKER -i br-fad2adb8a7aa -j RETURN: iptables: No chain/target/match by that name.
 (exit status 1))
```
> 如果报错，是因为关闭了防火墙之后，docker需要重启
### 查看是否成功
```
[root@localhost ethereumdemo]# docker network ls
NETWORK ID          NAME                 DRIVER              SCOPE
094f240a88fc        bridge               bridge              local
25176c69ef64        docker-diy-network   bridge              local
a1ee832da3a2        host                 host                local
624ad25aec0e        none                 null                local
```
