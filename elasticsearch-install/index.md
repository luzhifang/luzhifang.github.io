# Elasticsearch 安装


## 单节点安装

修改相关配置文件

```Shell
# 或者执行  sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
echo "* soft nproc 131072" >> /etc/security/limits.conf
echo "* hard nproc 131072" >> /etc/security/limits.conf
echo "* soft nofile 131072" >> /etc/security/limits.conf
echo "* hard nofile 131072" >> /etc/security/limits.conf
```

### 二进制方式安装

官网下载[链接](https://www.elastic.co/cn/downloads/elasticsearch)

```
cd /usr/local
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-xxx-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-xxx-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-xxx-linux-x86_64.tar.gz.sha512
tar -xzf elasticsearch-xxx-linux-x86_64.tar.gz
cd elasticsearch-xxx/
./bin/elasticsearch -d -p pid
# 验证是否启动成功
curl --cacert $ES_HOME/config/certs/http_ca.crt -u elastic https://localhost:9200
```

### docker 方式安装

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.6.0
docker network create elastic
docker run --name es01 --net elastic -p 9200:9200 -it docker.elastic.co/elasticsearch/elasticsearch:8.6.0
# 重置相关密码
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password
```

## 集群

首先按照单节点安装能跑起来先

然后修改各自配置文件 config/elasticsearch.yml

下面是三份配置示例，具体配置可能不同版本稍有不同，请参照官方文档

**node-1**

```
# 集群名称，三台集群，要配置相同的集群名称！！！
cluster.name: my-es-cluster
# 节点名称
node.name: node-1
# 是不是有资格主节点
node.master: true
# 是否存储数据
node.data: true
# ⽹关地址
network.host: 0.0.0.0
network.publish_host: 192.168.0.1
# http端⼝
http.port: 9200
# 内部节点之间沟通端⼝
transport.port: 9300
# es7.x 之后新增的配置，写⼊候选主节点的设备地址，在开启服务后可以被选为主节点
discovery.seed_hosts: ["192.168.0.1:9300","192.168.0.2:9300","192.168.0.3:9300"]
# es7.x 之后新增的配置，初始化⼀个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
# 数据和存储路径
path.data: /home/ubuntu/es/data
path.logs: /home/ubuntu/es/logs
```

**node-2**

```
# 集群名称，三台集群，要配置相同的集群名称！！！
cluster.name: my-es-cluster
# 节点名称
node.name: node-2
# 是不是有资格主节点
node.master: true
# 是否存储数据
node.data: true
# ⽹关地址
network.host: 0.0.0.0
network.publish_host: 192.168.0.2
# http端⼝
http.port: 9200
# 内部节点之间沟通端⼝
transport.port: 9300
# es7.x 之后新增的配置，写⼊候选主节点的设备地址，在开启服务后可以被选为主节点
discovery.seed_hosts: ["192.168.0.1:9300","192.168.0.2:9300","192.168.0.3:9300"]
# es7.x 之后新增的配置，初始化⼀个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
# 数据和存储路径
path.data: /home/ubuntu/es/data
path.logs: /home/ubuntu/es/logs
```

**node-3**

```
# 集群名称，三台集群，要配置相同的集群名称！！！
cluster.name: my-es-cluster
# 节点名称
node.name: node-3
# 是不是有资格主节点
node.master: true
# 是否存储数据
node.data: true
# ⽹关地址
network.host: 0.0.0.0
network.publish_host: 192.168.0.3
# http端⼝
http.port: 9200
# 内部节点之间沟通端⼝
transport.port: 9300
# es7.x 之后新增的配置，写⼊候选主节点的设备地址，在开启服务后可以被选为主节点
discovery.seed_hosts: ["192.168.0.1:9300","192.168.0.2:9300","192.168.0.3:9300"]
# es7.x 之后新增的配置，初始化⼀个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["node-1", "node-2","node-3"]
# 数据和存储路径
path.data: /home/ubuntu/es/data
path.logs: /home/ubuntu/es/logs
```

修改好配置之后，重启三台 elasticsearch

## 参考

https://blog.csdn.net/carefree2005/article/details/118027877

