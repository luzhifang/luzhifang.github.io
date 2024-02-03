# Gitlab Ubuntu 安装


## 简介

GitLab 是一个基于 Git 的平台，提供对 Git 存储库的远程访问，并通过创建用于管理代码的私有和公共存储库，有助于软件开发周期。

GitLab 支持不同类型的操作系统，例如:Windows，Ubuntu，Debian，CentOS 等。

## 安装

1. apt 安装

```Shell
# 安装依赖
sudo apt update
sudo apt-get upgrade
sudo apt install ca-certificates curl openssh-server ca-certificates postfix

# 执行脚本
cd /tmp
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh |sudo bash

# 修改源
# sudo vim /etc/apt/sources.list.d/gitlab_gitlab-ce.list
# 修改成以下内容
# deb [signed-by=/usr/share/keyrings/gitlab_gitlab-ce-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu focal main
# deb-src [signed-by=/usr/share/keyrings/gitlab_gitlab-ce-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu focal main
# 更新
# sudo apt update
# 安装



sudo apt install gitlab-ce
```

2. 调整防火墙规则

```Shell
sudo ufw allow http
sudo ufw allow https
sudo ufw allow OpenSSH
```

3. 编辑 Gitlab 配置文件

```Shell
vim /etc/gitlab/gitlab.rb
```

修改 external_url

```
external_url 'https://example.com' // 此处修改为您的域名或ip地址
```

4. 开机启动和禁止

```Shell
sudo systemctl enable gitlab-runsvdir.service
sudo systemctl disable gitlab-runsvdir.service
```

## 常用命令

| 常用命令                    | 说明                                                     |
| --------------------------- | -------------------------------------------------------- |
| sudo gitlab-ctl reconfigure | 重新加载配置，每次修改/etc/gitlab/gitlab.rb 文件之后执行 |
| sudo gitlab-ctl status      | 查看 GitLab 状态                                         |
| sudo gitlab-ctl start       | 启动 GitLab                                              |
| sudo gitlab-ctl stop        | 停止 GitLab                                              |
| sudo gitlab-ctl restart     | 重启 GitLab                                              |
| sudo gitlab-ctl tail        | 查看所有日志                                             |
| sudo gitlab-ctl tail        | nginx/gitlab_acces.log 查看 nginx 访问日志               |
| sudo gitlab-ctl tail        | postgresql 查看 postgresql 日志                          |

## 配置密码

在 Web 浏览器中访问 GitLab 服务器的域名：

```
https://example.com // 您external_url配置的地址
```

启动成功后，默认有个管理员账号

登录名：root

登录密码：初始密码在这个文件中/etc/gitlab/initial_root_password (可更改)

## 设置克隆地址

进入 vim /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml

设置 gitlab:host

## 邮箱配置

1. 开启 qq 邮箱的 POP3/SMTP 服务并保存好授权码

这一步在 qq 邮箱的设置 -> 账户中

点击开启按照提示步骤操作会获得相应的授权码

2. 修改配置文件

```Shell
sudo vim /etc/gitlab/gitlab.rb
```

```
#配置邮箱来源， 与展示的名称
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = '您的qq邮箱地址'
gitlab_rails['gitlab_email_display_name'] = '您的邮箱显示名称'

#smtp配置
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "您的qq邮箱地址"
gitlab_rails['smtp_password'] = "您的授权码"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true
```

```Shell
# 重新加载配置
sudo gitlab-ctl reconfigure
```

3. 发送测试邮件

```Shell
sudo gitlab-rails console

#进入控制台，然后发送邮件
Notify.test_email('测试邮箱地址', '邮件标题', '邮件正文').deliver_now
```

## 修改端口

```Shell
sudo vim /etc/gitlab/gitlab.rb
```

```
nginx['listen_port'] = 9091 // GitLab端口，默认80端口
unicorn['port'] = 9092 // 可不修改，默认监听8080端口
```

```Shell
sudo gitlab-ctl reconfigure
```

## Git Pages 设置

```Shell
sudo vim /etc/gitlab/gitlab.rb
```

```
gitlab_pages[‘enable’] = true; 开启 Pages 服务
pages_external_url ‘您的GitLab Pages域名地址'; 替换成你自己的域名
gitlab_pages[‘inplace_chroot’] = true; 以Docker container 方式运行的 Gitlab 必须开启此项
pages_nginx[‘enable’] = true; 开启 Pages 服务的 vhost，该项开启后将会在 /var/opt/gitlab/nginx/conf 目录下生成独立的名为 gitlab-pages.conf Nginx 配置文件。
gitlab_pages['access_control'] = true 开启 Pages 访问控制。
```

```Shell
sudo gitlab-ctl reconfigure
```

## 参考

https://about.gitlab.com/install/#ubuntu

https://www.cnblogs.com/bymo/p/9046586.html

https://blog.csdn.net/OldDirverHelpMe/article/details/106536972

https://www.cnblogs.com/bymo/p/9046586.html

https://cloud.tencent.com/developer/article/1711802

https://www.webfunny.com/blog/post/207

