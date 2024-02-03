# Ubuntu 安装 Docker


## 使用官方安装脚本自动安装

注意：脚本安装适合在测试和开发环境使用。

安装命令如下：

```Shell
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```Shell
curl -sSL https://get.daocloud.io/docker | sh
```

## 手动安装

### 更新库和安装依赖

```Shell
apt-get update
apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

### 添加 docker 的官方 GPG key

```Shell
mkdir -p /etc/apt/keyrings
# 官方源
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod -R 0755 /etc/apt/keyrings
```

### 将 Docker 的源添加到 /etc/apt/sources.list.d

```Shell
# 官方源
# echo \
#   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
#   $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
echo \
  "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 安装 docker

```Shell
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 启动 docker

```Shell
service enable docker
service start docker

# wsl2 ubuntu 22.04 启动报错，执行以下命令修复
# update-alternatives --set iptables /usr/sbin/iptables-legacy
# update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```

## 配置国内镜像源

创建或修改 /etc/docker/daemon.json 文件，修改为如下形式

```
{
    "registry-mirrors": [
        "http://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
}
```

重启 docker，`service docker restart`。

查看是否修改成功，`docker info`。

## 参考

https://docs.docker.com/engine/install/ubuntu/

https://www.runoob.com/docker/ubuntu-docker-install.html

https://yeasy.gitbook.io/docker_practice/install/ubuntu

