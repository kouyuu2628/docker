## 下载以太坊的镜像
```
docker pull ethereum/client-go
```
- ethereum/client-go:latest 开发版本的Geth  
- ethereum/client-go:stable 稳定版本的Geth  
- ethereum/client-go:{version} 指定版本的Geth  
- ethereum/client-go:release-{version} 指定稳定版本的Geth
> windows和虚拟机的centos7上的节点都安装的是v1.9.9稳定版，所以此处也需要指定版本。  
> `docker pull ethereum/client-go:v1.9.9`，如果拉取失败，可以关闭国内镜像加速，反而很快。（下载完后再开启镜像加速）
## 启动容器前的准备
- 创世区块配置文件
```
#/usr/local/ethereumdemo/genesis.json
{
  "config": {
    "chainId": 1234,
    "homesteadBlock": 0,
    "eip150Block": 0,
    "eip155Block": 0,
    "eip158Block": 0,
    "byzantiumBlock": 0,
    "constantinopleBlock": 0,
    "petersburgBlock": 0
  },
  "alloc": {},
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x400",
  "extraData": "",
  "gasLimit": "0x2fefd8",
  "nonce": "0x0000000000000042",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
}
```
- 编写初始化节点脚本`init04.sh`
```
#!/bin/sh
#初始化创世区块
geth --datadir /workspace/selfchain04/ init /workspace/genesis.json
if [  $# -lt 1 ]; then
  exec "/bin/sh"
else
  exec /bin/sh -c "$@"
fi
```
## 创建子网
```
docker network create --subnet=172.18.0.0/24 docker-diy-network
```
> 掩码用24(255.255.255.0)，与宿主机掩码一致
```
Error response from daemon: Failed to Setup IP tables: Unable to enable SKIP DNAT rule:  (iptables failed: iptables --wait -t nat -I DOCKER -i br-fad2adb8a7aa -j RETURN: iptables: No chain/target/match by that name.
 (exit status 1))
```
> 如果报错，是因为关闭了防火墙之后，docker需要重启。
> 执行`systemctl restart docker`即可。
### 查看是否成功
```
[root@localhost ethereumdemo]# docker network ls
NETWORK ID          NAME                 DRIVER              SCOPE
094f240a88fc        bridge               bridge              local
25176c69ef64        docker-diy-network   bridge              local
a1ee832da3a2        host                 host                local
624ad25aec0e        none                 null                local
```
## 启动容器
```
docker run -d -it --name selfchain04 --net docker-diy-network --ip 172.18.0.2 -p 8548:8548 -p 3003:3003 -v /usr/local/ethereumdemo:/workspace -v /etc/localtime:/etc/localtime:ro --privileged=true --entrypoint /workspace/init04.sh ethereum/client-go:v1.9.9
```
- `--name selfchain04`：容器名称
- `--net docker-diy-network`：创建的子网名称
- `--ip 172.18.0.2`：指定一个子网IP给该容器
- `-v /etc/localtime:/etc/localtime:ro`：容器挂载目录，使容器时间与宿主时间一致。时间不一致，节点的区块同步不会成功。
- `-v /usr/local/ethereumdemo:/workspace`：容器挂载目录，用于存放节点信息
- `--entrypoint /workspace/init04.sh`：执行初始化节点脚本，初始化创世区块
- `–privileged=true`：使用该参数，container内的root拥有真正的root权限
- `-p 8548:8548 -p 3003:3003`：预留的RPC端口
> 这里有个坑，如果在windows使用MobaXterm连接centos，通过notepad++编辑init.sh，会存在编码格式问题，一定要把格式转换成"unix(LF)"
## 查看容器
```
[root@localhost ethereumdemo]# docker ps -as
CONTAINER ID        IMAGE                COMMAND                CREATED             STATUS              PORTS                                                                                 NAMES
26d2a0e5be55 e       ethereum/client-go   "/workspace/init.sh"   3 seconds ago       Up 2 seconds        0.0.0.0:3003->3003/tcp, 8545-8547/tcp, 0.0.0.0:8548->8548/tcp, 30303/tcp, 30303/udp   selfchain04
```
## 进入容器
```
docker exec -it selfchain04 sh
```
切换至工作目录，查看目录信息
```
/ # cd workspace/
/workspace # ls
demo1218         genesis.json     selfchain03      selfchain04
demo1219         init.sh          selfchain03.log  voting
```
里面已经有了节点目录`selfchain04`
## 启动私有链网络
```
geth --networkid 123 --datadir /workspace/selfchain04 --rpc --rpcaddr 172.18.0.2 --rpcport 8548 --port 3003 --rpcapi personal,db,eth,net,web3 --allow-insecure-unlock --ipcdisable --nodiscover console 2>>selfchain04.log
```
> 可以写入shell脚本，方便下次启动
```
cd /workspace/
touch selfchain04-start.sh
vi selfchain04-start.sh
```
`selfchain04-start.sh`
```
#!/bin/sh
geth --networkid 123 --datadir /workspace/selfchain04 --rpc --rpcaddr 172.18.0.2 --rpcport 8548 --port 3003 --rpcapi personal,db,eth,net,web3 --allow-insecure-unlock --ipcdisable --nodiscover console 2>>selfchain04.log
```
> 如果启动时提示没有权限
```
chmod -R 777 selfchain04-start.sh
```
## 在centos上启动第三个节点`selfchain03`
新开一个终端，连接centos，启动节点
```
geth --networkid 123 --datadir /usr/local/ethereumdemo/selfchain03 --rpc --rpcaddr 192.168.11.237 --rpcport 8547 --port 3002 --rpcapi personal,db,eth,net,web3 --allow-insecure-unlock --ipcdisable --nodiscover console 2>>selfchain03.log
```
获取第三个节点的节点信息
> centos分配的ip为192.168.11.237
```
> admin.nodeInfo.enode
"enode://63cc3a090ddd2efc586d43b9ef93bd17c187b6abd1b3f3a7dd19972f44783f91f0ae860aed3f341f7ca4c1f00c9afe07bad49815ea1d4bcb9aaf7dac1c6dbb5d@192.168.11.237:3002?discport=0"
```
## 容器内的节点`selfchain04`连接centos的第三个节点`selfchain03`
```
> admin.addPeer("enode://63cc3a090ddd2efc586d43b9ef93bd17c187b6abd1b3f3a7dd19972f44783f91f0ae860aed3f341f7ca4c1f00c9afe07bad49815ea1d4bcb9aaf7dac1c6dbb5d@192.168.11.237:3002?discport=0")
true
```
## 查看节点连接信息
```
> admin.peers #查看节点连接信息

[{
    caps: ["eth/63", "eth/64"],
    enode: "enode://63cc3a090ddd2efc586d43b9ef93bd17c187b6abd1b3f3a7dd19972f44783f91f0ae860aed3f341f7ca4c1f00c9afe07bad49815ea1d4bcb9aaf7dac1c6dbb5d@192.168.11.237:3002?discport=0",
    id: "186f5c557449a61d63839097222c742924b9bfdf6443a1ac6a134456ea0a30ff",
    name: "Geth/v1.9.9-stable-01744997/linux-amd64/go1.13.5",
    network: {
      inbound: false,
      localAddress: "172.17.0.2:51914",
      remoteAddress: "192.168.11.237:3002",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 50907657,
        head: "0x41fef35868b7e717ff27dbe970deb7d32625ac5848dd8760463c04a84fc7e0c4",
        version: 64
      }
    }
}]
```

## 为何节点连接成功，但是区块同步一直失败？
### 1、本地节点的时间一定要与连接节点的时间一致
- centos7同步网络时间
```
yum -y install ntp ntpdate  #安装ntpdate
ntpdate 0.asia.pool.ntp.org #同步网络时间
hwclock --systohc           #将系统时间写入硬件时间，防止系统重启后时间被还原
```
- docker容器与宿主机centos时间同步（docker默认是utc时间）
```
#运行容器时，添加如下参数，挂载宿主机的时间目录
-v /etc/localtime:/etc/localtime:ro
```
- 节点与节点之间能互相ping通。可能中途突然ping不通，重启网络。
```
#重启centos网络
systemctl restart network
#重启docker
systemctl restart docker
```
### 2、geth中用`admin.peers`检测节点连接情况，可能因为某些原因断开。如果断开，查看节点日志，找到断开原因。
### 3、是否有将节点的端口添加到公共区域
```
systemctl start firewalld
firewall-cmd --zone=public --add-port=端口/tcp --permanent
firewall-cmd --reload
```
### 4、新创建的节点，没有账户，需要新建一个账户作为挖矿账户，然后挖矿一段时间。
- 创建账户
```
personal.newAccount("密码")
```
- 开始挖矿。  
> 头一次时间会比较长，需要生成挖矿使用的DAG数据集（大约1个G），它是用于以太坊工作量证明PoW算法的数据集。
```
miner.start()
```
> 在这个过程中，尝试去停止挖矿`miner.stop()`应该不会成功，除非`exit`退出geth。
- 挖矿过程中观察日志，查看是否有报错，分析原因。
- 尝试查看节点的区块`eth.blockNumber`，看看是否已经与连接的节点同步。  
- 如果很长时间之后（5分钟）区块仍然为0，说明区块还未同步。可以尝试以下做法：
  - 查看日志
  - 执行`步骤2`
  - 退出节点，重新启动后开始挖矿

# 节点连接场景
截至目前，完成了以下场景的私有链节点连接：
1. windows<-->windows
2. windows<-->centos
3. windows<-->centos下docker容器节点
4. centos<-->centos
5. centos<-->centos下docker容器节点
6. centos下docker容器1的节点<-->centos下docker容器2的节点
> 1/4/6场景下的节点连接和区块同步是最容易成功的，基本上连接上后就自动完成了同步。跨机器连接就会需要额外的操作。