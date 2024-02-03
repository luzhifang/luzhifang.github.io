# Go 安装、升级、卸载


## 卸载

如果以前安装过版本，可以先卸载

1. 如果是用 Ubuntu 用 apt 安装的，用以下命令卸载

```Shell
sudo apt-get remove golang-go
sudo apt-get remove --auto-remove golang-go
```

2. 如果是手动下载安装的，删除相应目录即可

## 安装

1. 下载 go

   1. 方法一：官网下载：https://go.dev/dl/
   2. 方法二：打开 ubuntu 输入：

   ```Shell
   wget https://dl.google.com/go/go1.15.3.linux-amd64.tar.gz
   ```

2. 解压安装包

```Shell
sudo tar -C /usr/local -xzf go1.15.3.linux-amd64.tar.gz
```

3. 设置环境变量

   1. 建立软链接

   ```Shell
   sudo ln -s /usr/local/go/bin/* /usr/bin/
   ```

   2. 修改/etc/profile

   ```Shell
   # 打开/etc/profile，添加以下内容
   export GOPATH="$HOME/go"
   export PATH="$PATH:/usr/local/go/bin:$GOPATH/bin"

   # 重新加载文件
   source /etc/profile
   ```

4. 验证是否生效

```Shell
go version
```

## 升级

先卸载再安装

