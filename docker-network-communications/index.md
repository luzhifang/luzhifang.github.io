# Docker 网络互通


完整示例请移步 [Github](https://github.com/luzhifang/prometheus-grafana-demo)

## docker 网卡介绍

```Shell
$ docker network ls
NETWORK ID     NAME                                         DRIVER    SCOPE
60ca2bcc3925   bridge                                       bridge    local
4fba5fc9c823   host                                         host      local
15b262a2e321   none                                         null      local
```

- bridge：默认网卡，类似于 VMware 的 NAT 模式，如果需要访问容器内部的端口需要进行端口映射。
- host：直接使用主机网络，类似于 VMware 的桥接模式，访问容器内部的端口时不需要进行端口映射，直接访问即可，但是可能会与主机的端口号冲突。
- none：禁止所有联网，没有网络驱动。

## docker 自定义网络

使用 `docker network create custom-local-net` 创建一个名为 custom-local-net 的 Docker 网卡，这个网卡是基于 bridge 模式的，但是和 bridge 模式又有一定的区别。

1. 用户自定义的网络是基于 bridge 的，并且容器间可以通过容器名或别名进行自动 DNS 相互解析的功能！
2. 使用默认的 bridge 网络仅仅能通过各自的 IP 地址来进行通信，除非使用 --link 选项
3. --link 选项实现的容器相互访问功能已经被官方认定是过时的

## link 方式共享

### docker run 使用 link 参数

在不使用 Docker Compose 的时候，在启动命令中加入 --link 参数，就可以实现容器之间的访问

```Shell
# 启动 prometheus 容器
docker run --rm --name prometheus -p 9090:9090 -d quay.io/prometheus/prometheus

# 启动 grafana 容器
docker run --rm --name grafana -p 3000:3000 -d grafana/grafana

# 进入 grafana 容器，查看是否可以 ping 通
docker exec -it 60ca2bcc3925 bash
ping prometheus
```

### [docker compose](/docker-compose) 使用 links 参数

相比使用 docker run 命令启动容器，显然 docker compose 才是更为推荐的那一个方式，把启动所需的镜像、环境等全都写到一份 docker-compose.yaml 文件中，方便使用。

如果使用 docker compose 的方式，就是把两个容器的启动信息都写到同一份文件中，再在需要依赖另一个容器的容器启动信息中加入 depends_on、links 参数，这样 grafana 也可以使用 prometheus 解析到其容器的 IP 地址，得到的结果和上面使用 docker run 的方式得到的结果是一样的。

```
version: '3.4'
services:
  prometheus:
    image: quay.io/prometheus/prometheus
    container_name: prometheus
    hostname: prometheus
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules:/etc/prometheus/rules
  grafana:
    image: grafana/grafana
    container_name: grafana
    hostname: grafana
    ports:
      - 3000:3000
    volumes:
      - ./grafana.ini:/etc/grafana/grafana.ini
    depends_on:
      - prometheus
    links:
      - prometheus
```

## 自定义网络共享

在上面我们简单地介绍了一下如何自定义网络，并且知道使用 link 参数的方式已经是过时的，那通过自定义网络，让 grafana 容器和 prometheus 容器处于同一个网络，并且他们可以互相进行 DNS 解析，就可以让 grafana 访问到 prometheus 容器了。

### docker run 使用自定义网络

docker run 命令也可以手动指定容器连接的网络，使用 network 参数。

```Shell
# 自定义网络 custom-local-net
docker network create custom-local-net

# 启动 prometheus 容器
docker run --rm --name prometheus -p 9090:9090 --network custom-local-net -d quay.io/prometheus/prometheus

# 启动 grafana 容器
docker run --rm --name grafana -p 3000:3000 --network custom-local-net -d grafana/grafana

# 进入 grafana 容器中 ping prometheus 查看是否能ping 通
docker exec -it 60ca2bcc3925 bash
ping prometheus
```

### docker compose 使用自定义网络

docker-compose.yaml 文件中有两种使用自定义网络的方式：

1. 创建并使用，如果没有手动使用 `docker network create` 命令，需要在使用前创建

```
version: '3.4'
services:
  prometheus:
    image: quay.io/prometheus/prometheus
    container_name: prometheus
    hostname: prometheus
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules:/etc/prometheus/rules
    networks:
      - custom-local-net-1
  grafana:
    image: grafana/grafana
    container_name: grafana
    hostname: grafana
    ports:
      - 3000:3000
    volumes:
      - ./grafana.ini:/etc/grafana/grafana.ini
    depends_on:
      - prometheus
    networks:
      - custom-local-net-1
networks:
  custom-local-net-1:
    # 声明使用的网络是使用 bridge 驱动来创建的
    driver: bridge
    ipam:
      # 网络配置
      config:
        # 分配的子网网段
        - subnet: 172.25.64.0/18
          # 网关地址
          gateway: 172.25.64.1
```

2. 声明并使用，如果已经手动创建了网络，在 docker-compose.yaml 文件中只需声明一下即可

```
version: '3.4'
services:
  prometheus:
    image: quay.io/prometheus/prometheus
    container_name: prometheus
    hostname: prometheus
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules:/etc/prometheus/rules
    networks:
      - custom-local-net
  grafana:
    image: grafana/grafana
    container_name: grafana
    hostname: grafana
    ports:
      - 3000:3000
    volumes:
      - ./grafana.ini:/etc/grafana/grafana.ini
    depends_on:
      - prometheus
    networks:
      - custom-local-net
networks:
  custom-local-net:
  	# 声明这个网络是外部定义的
    external: true
```

## 总结

Docker 容器之间相互访问是实际生产中难以绕开的一道坎，诚然可以使用桥接模式，但是桥接模式不太利于环境的迁移。显然使用能自动完成 DNS 解析的网络模式会更为灵活，也更为优雅。

## 参考

https://juejin.cn/post/7143883774818795527

