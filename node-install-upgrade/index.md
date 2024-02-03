# Linux安装node和升级node


## 下载 Node

进入 Node 最新版下载 [https://nodejs.org/en/download/current/](https://nodejs.org/en/download/current/)
例：`https://nodejs.org/dist/v16.17.1/node-v16.17.1-linux-x64.tar.xz`

## 安装 Node

```Shell
cd /usr/local
wget https://nodejs.org/dist/v16.17.1/node-v16.17.1-linux-x64.tar.xz
tar -xvf  node-v16.17.1-linux-x64.tar.xz
cd node-v16.17.1-linux-x64/
ln -s /usr/local/node-v16.17.1-linux-x64/bin/node /usr/bin/node
ln -s /usr/local/node-v16.17.1-linux-x64/bin/npm /usr/bin/npm
node -v
npm -v
```

## 加速 npm

1. 使用淘宝的 npm `npm install cnpm -g --registry=https://registry.npm.taobao.org`
2. 使用 cnpm 去代替 npm 来执行，比如 cnpm install xxx
3. 不想安装 cnpm，也可以每次使用`npm install xxx --registry=https://registry.npm.taobao.org`直接安装

## 升级 Node

1. 清除缓存信息 `sudo npm cache clean -f`
2. 下载 node 安装包 `sudo npm install -g n`
3. 升级到 nodejs 最新稳定版本 `sudo n stable`
4. 查看当前版本 `node -v`

