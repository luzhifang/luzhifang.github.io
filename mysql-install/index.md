# mysql 安装


## Ubuntu

1. 查看 Ubuntu 版本 `cat /etc/lsb-release`

2. 添加源

```Shell
vim /etc/apt/sources.list.d/mysql-community.list
```

```Code
# Ubuntu 20.04 LTS
deb https://mirrors.tuna.tsinghua.edu.cn/mysql/apt/ubuntu focal mysql-5.7 mysql-8.0 mysql-tools

# Ubuntu 18.04 LTS
deb https://mirrors.tuna.tsinghua.edu.cn/mysql/apt/ubuntu bionic mysql-5.7 mysql-8.0 mysql-tools

# Ubuntu 16.04 LTS
deb https://mirrors.tuna.tsinghua.edu.cn/mysql/apt/ubuntu xenial mysql-5.6 mysql-5.7 mysql-8.0 mysql-tools

# Ubuntu 14.04 LTS
deb https://mirrors.tuna.tsinghua.edu.cn/mysql/apt/ubuntu trusty mysql-5.6 mysql-5.7 mysql-8.0 mysql-tools
```

3. 导入 GPG 密钥

```Shell
# apt-key adv --keyserver pgp.mit.edu --recv-keys 3A79BD29
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3A79BD29
```

4. 更新库 `apt update`

5. 查看具体版本 `apt-cache policy mysql-server`

6. apt 安装

```Shell
# 安装5.7
apt install mysql-community-client=5.7.41-1ubuntu18.04 mysql-client=5.7.41-1ubuntu18.04
apt install mysql-community-server=5.7.41-1ubuntu18.04 mysql-server=5.7.41-1ubuntu18.04

# 安装8.0
apt install mysql-client mysql-server
```

7. 验证安装是否成功

```Shell
mysql --version
service mysql start
netstat -lnp | grep mysql
```

## 参考

https://blog.csdn.net/simplemurrina/article/details/80088479

https://www.cnblogs.com/immaxfang/p/16804455.html

https://blog.csdn.net/weixin_44129085/article/details/105403674

https://downloads.mysql.com/archives/community/
