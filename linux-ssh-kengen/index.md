# Linux使用ssh-kengen生成公私钥和免密登录


## 详细命令

```Shell
ssh-keygen \
    -t rsa \
    -m PEM \
    -b 4096 \
    -C "user@server" \
    -f /home/username/.ssh/keys/privatekey \
    -N passphrase
```

## 命令释义

1. ssh-keygen = 用于创建密钥的程序
2. -m PEM = 将密钥的格式设为 PEM
3. -t rsa = 要创建的密钥类型，本例中为 RSA 格式
4. -b 4096 = 密钥的位数，本例中为 4096
5. -C "user@server" = 追加到公钥文件末尾以便于识别的注释。 通常以电子邮件地址用作注释，但也可以使用任何最适合你基础结构的事物。
6. -f /home/username/.ssh/keys/privatekey = 私钥文件的文件名（如果选择不使用默认名称）。 追加了 .pub 的相应公钥文件在相同目录中生成。
   该目录必须存在。
7. -N passphrase = 用于访问私钥文件的其他密码。

## 使用示例：

```Shell
username@hostname:~# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa):   // 这里输入要生成的文件名
Created directory '/home/username/.ssh'.                            // 这里输入密码
Enter passphrase (empty for no passphrase):                         // 这里重复输入密码
Enter same passphrase again:
Your identification has been saved in /home/username/.ssh/id_rsa.
Your public key has been saved in /home/username/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:LIGNrf7jteWJtB6quaJgOwT4yLkbyIBhCTUOCW58Em0 /home/username/@hostname
The key's randomart image is:
+---[RSA 2048]----+
|=++              |
|=ooE =           |
|oBo.o +          |
|*.o  . o         |
|=o. . . S        |
|=+..   .         |
|++. .   + .      |
|oo+  o.+ B .     |
|.+o.+++o= o      |
+----[SHA256]-----+
```

## 免密登录

1. 复制 A 机公钥到 B 机/home/username/.ssh 目录下的 authorized_keys 文件里。

```
scp /home/username/.ssh/id_rsa.pub username@192.168.1.100:/home/username/.ssh/authorized_keys
```

2. B 机中给 authorized_keys 文件 600 权限

```
chmod 600 authorized_keys
```

3. 、A 机登录 B 机

```
ssh 192.168.1.100
```

