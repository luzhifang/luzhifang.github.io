# k8s 安装


目前主流的搭建 k8s 集群的方式有 minikube、二进制包以及 kubeadm。

## minikube 方式安装

minikube 一般用于本地开发、测试和学习，不能用于生产环境，是一个工具，minikube 快速搭建一个运行在本地的单节点的 Kubernetes。

1. 需要提前安装[docker](docker-install)

2. docker 需要运行在非 root 用户下

3. 把运行用户添加到 docker 用户组下，`sudo usermod -aG docker ubuntu && newgrp docker`

4. 安装 minikube

```Shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

5. 启动`minikube start`

6. 交互

```Shell
kubectl get po -A
minikube kubectl -- get po -A
alias kubectl="minikube kubectl --"
minikube dashboard
```

## kubeadm 方式安装

### 禁用 swap

```Shell
# 临时禁止
sudo swapoff -a
# 永久禁止
# /etc/fstab 文件 注释掉 /swapfile 行
```

### 安装 CRI 容器运行时

安装[containerd](https://github.com/containerd/containerd/releases)

```Shell
tar Cxzvf /usr/local containerd-1.6.2-linux-amd64.tar.gz
bin/
bin/containerd-shim-runc-v2
bin/containerd-shim
bin/ctr
bin/containerd-shim-runc-v1
bin/containerd
bin/containerd-stress
```

拷贝 https://raw.githubusercontent.com/containerd/containerd/main/containerd.service 内容至 /etc/systemd/system/containerd.service

```Shell
systemctl daemon-reload
systemctl enable --now containerd
```

安装 [runc](https://github.com/opencontainers/runc/releases)

```Shell
install -m 755 runc.amd64 /usr/local/sbin/runc
```

### 安装和配置先决条件

转发 IPv4 并让 iptables 看到桥接流量

```Shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system
```

配置 systemd cgroup 驱动

```Shell
containerd config default > /etc/containerd/config.toml
# 修改 [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
#        SystemdCgroup = true
# 修改 sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.9"
systemctl daemon-reload
systemctl restart containerd
```

### 安装 kubeadm

```Shell
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

### 初始化集群

```Shell
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers
kubeadm init --pod-network-cidr=10.244.0.0/16 --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=IP --control-plane-endpoint=IP
```

```Shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 添加 worker node

```Shell
# 以下命令参见 init 成功之后的提示
kubeadm join 10.0.12.17:6443 --token llcvt5.3cd0pw2stgxrmbui \
	--discovery-token-ca-cert-hash sha256:b501bac3e340e833f726ccc9e2520c90a8cdb23e03424173bafa83c857e48f0b
```

### 添加网络组件

```Shell
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

### 验证

```Shell
# 后面加上 -v 9，可查看每条命令执行的日志
kubectl run nginx --image=nginx
```

### FAQ

#### 使用公网 IP 搭建 k8s 集群

创建虚拟网卡

```Shell
# 所有主机都要创建虚拟网卡，并绑定对应的公网 ip
# 该设置方式在重启服务器后失效，持久化需要将配置写入/etc/network/interfaces或/etc/netplan/50-cloud-init.yaml
sudo ifconfig eth0:1 139.198.108.103
```

#### x509: certificate is valid for

```Shell
kubeadm reset
# 需要删除配置再重新生成
rm -rf $HOME/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 加上参数：--apiserver-cert-extra-sans=IP
kubeadm init --pod-network-cidr=10.244.0.0/16 --image-repository registry.aliyuncs.com/google_containers --apiserver-cert-extra-sans=IP
```

#### kubeadm init 查看日志

```Shell
systemctl status kubelet
journalctl -f -u kubelet
```

## 参考

https://minikube.sigs.k8s.io/docs/start/

https://developer.aliyun.com/mirror/kubernetes?spm=a2c6h.13651102.0.0.472e1b11kbg5Ix

https://kubernetes.io/zh-cn/docs/setup/

https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

https://cloud.tencent.com/developer/article/2164600

