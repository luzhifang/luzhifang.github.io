# Go多版本管理器


## windows

1. 下载多版本管理工具 g，地址：https://github.com/voidint/g/releases，选择g*.windows-amd64.zip
2. 解压 g 压缩包到自己喜好的目录，并将该目录添加到 PATH 环境变量
3. 设置环境变量：G_MIRROR 值为https://gomirrors.org/
4. 设置/修改环境变量 GOROOT 值为 C:\Users\个人账户\.g\go，C:\Users\个人账户\.g\目录下 downloads 目录为已下载包、version 目录为解压的版本
5. 将之前环境变量 PATH 里面 go 的 bin 目录去掉
6. 重要：右键以管理员身份打开 dos 窗口
7. g ls 查询已安装的 go 版本
8. g ls-remote 查询可供安装的所有 go 版本
9. g ls-remote stable 查询当前可供安装的 stable 状态的 go 版本
10. g install 1.14.6 安装目标 go 版本 1.14.6
11. g use 1.14.6 切换至 1.14.6 版本
12. g uninstall 1.14.6 卸载一个已安装的 go 版本

## linux

1. curl -sSL https://raw.githubusercontent.com/voidint/g/master/install.sh | bash
   1. 没有 curl 命令可以把 install.sh 下载下来直接运行
2. source ~/.bashrc
3. 修改~/.bashrc 文件最下面 G_MIRROR 值为https://gomirrors.org/
4. 将之前环境变量 PATH 里面 go 的 bin 目录去掉
5. g ls 查询已安装的 go 版本
6. g ls-remote 查询可供安装的所有 go 版本
7. g ls-remote stable 查询当前可供安装的 stable 状态的 go 版本
8. g install 1.14.6 安装目标 go 版本 1.14.6
9. g use 1.14.6 切换至 1.14.6 版本
10. g uninstall 1.14.6 卸载一个已安装的 go 版本

