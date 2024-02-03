# centos6.6安装supervisor3.x方法


## 安装 meld (如果缺少的话)

```Shell
cd /usr/local
wget https://pypi.python.org/packages/0f/5e/3a57c223d8faba2c3c2d69699f7e6cfdd1e5cc31e79cdd0dd48d44580b50/meld3-1.0.1.tar.gz#md5=2f045abe0ae108e3c8343172cffbe21d
tar -zxvf meld3-1.0.1.tar.gz
cd meld3-1.0.1
python setup.py install
```

## 安装 supervisor3

```Shell
wget https://pypi.python.org/packages/source/s/supervisor/supervisor-3.0b2.tar.gz --no-check-certificate
tar -zxvf supervisor-3.0b2.tar.gz
cd supervisor-3.0b2
python setup.py install
yum install python-setuptools
```

## 创建日志文件

```Shell
touch /var/log/supervisord.log
chmod 755 /var/log/supervisord.log
```

## 创建默认的配置文件

```Shell
echo_supervisord_conf > /etc/supervisord.conf
```

## 修改/etc/supervisord.conf

```
# 添加到末尾
[include]
files = /etc/supervisord.d/\*.ini
```

