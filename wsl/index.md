# Windows11 安装 Linux 子系统


## 启用相关功能

首先搜索【启用或关闭 Winsows 功能】

然后勾选【适用于 linux 的 Windows 子系统】和【虚拟平台】

<img src="/images/windows/enable-linux.jpg" alt="" width="50%" />

## 下载安装 Linux 内核更新包

[wsl_update_x64.msi](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

## 设置默认版本为 WSL 2

```Shell
wsl --set-default-version 2
```

## 安装 Linux

最后打开应用商店，选择你喜欢的 linux 分发版本安装。

安装成功之后，打开相关 Linux 版本。

设置默认 root 账号打开。

```Shell
ubuntu2204.exe config --default-user root
```

## Windows 访问 Linux 文件

通过 \\wsl$ 访问 Linux 文件时将使用 WSL 分发版的默认用户。

如果修改没有权限，可以更改文件或目录权限以支持修改。

```Shell
chmod 777 -R /data
```

## Linux 访问 Windows 文件

在从 WSL 访问 Windows 文件时，可以直接使用/mnt/{Windows 盘符}进入对应的盘中。

## 参考

https://zhuanlan.zhihu.com/p/386385348

