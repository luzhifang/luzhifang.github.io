# GitLab 更换域名 + HTTPS 切换 HTTP 踩坑实录


## 背景

需要将自建 GitLab Omnibus 从 HTTPS 迁移到 HTTP，同时更换域名和端口。整体架构为：

```
浏览器 → CDN(:8080) → 反向代理 Nginx(:80) → GitLab 服务器(:8080)
```

| 项目 | 旧值 | 新值 |
|------|------|------|
| 域名 | git.old-domain.com | git.new-domain.com |
| 协议 | HTTPS | HTTP |
| 端口 | 8443 | 8080 |

## GitLab 配置修改

编辑 `/etc/gitlab/gitlab.rb`：

```ruby
external_url 'http://git.new-domain.com:8080'

# Puma 端口必须避开 nginx 的 8080（见下方踩坑）
puma['port'] = 8181

# Nginx 配置
nginx['listen_port'] = 8080
nginx['listen_https'] = false
nginx['redirect_http_to_https'] = false
nginx['ssl_certificate'] = ""
nginx['ssl_certificate_key'] = ""

# 关闭 Let's Encrypt
letsencrypt['enable'] = false
letsencrypt['auto_renew'] = false
```

```bash
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

## 踩坑记录

### 坑一：Puma 和 Nginx 端口冲突（502）

**现象**：`gitlab-ctl status` 显示 Puma 不断重启（pid 显示 0s），所有请求返回 502 Bad Gateway。

**原因**：当 `external_url` 使用非标准端口时，Puma 默认也会尝试绑定该端口的 TCP 监听。如果 Nginx 也监听同一端口，Puma 绑定失败后整个进程崩溃。

**关键日志**（`/var/log/gitlab/puma/current`）：

```
Address already in use - bind(2) for "127.0.0.1" port 8080 (Errno::EADDRINUSE)
```

而 Workhorse 日志（`/var/log/gitlab/gitlab-workhorse/current`）会显示：

```
badgateway: failed to receive response: dial unix /var/opt/gitlab/gitlab-rails/sockets/gitlab.socket: connect: connection refused
```

Puma 的 unix socket 绑定成功后，在绑定 TCP 端口时失败，导致整个进程退出，socket 随之消失。

**修复**：

```ruby
puma['port'] = 8181  # 改成任意未占用的端口
```

### 坑二：残留的 proxy_set_headers 导致 HTTPS 跳转

**现象**：`curl -I http://localhost:8080` 返回 302，Location 是 `https://` 开头。

**原因**：旧配置中有一段硬编码的 proxy headers：

```ruby
nginx['proxy_set_headers'] = {
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl" => "on",
  ...
}
```

即使把 `X-Forwarded-Proto` 改成 `http`，只要 `X-Forwarded-Ssl` 仍为 `on`，GitLab Rails 仍会生成 HTTPS 链接。**`X-Forwarded-Ssl` 的优先级高于 `X-Forwarded-Proto`**。

**修复**：注释掉整个 `proxy_set_headers` 块，让前端反代自行传入。

### 坑三：反向代理端口丢失

**现象**：访问 `http://git.new-domain.com:8080` 跳转到 `http://git.new-domain.com/users/sign_in`（端口丢失）。

**原因**：反代 Nginx 配置中 `Host` 设成了内网 IP，`X-Forwarded-Port` 用的是 `$server_port`（=80 而非 8080），且缺少 `proxy_redirect` 指令。

**修复**：反代 Nginx 的关键配置：

```nginx
location ^~ / {
    proxy_pass http://10.0.0.100:8080;

    # 1. Host 必须是域名+端口
    proxy_set_header Host git.new-domain.com:8080;

    # 2. X-Forwarded-Port 写死为外部端口
    proxy_set_header X-Forwarded-Port 8080;

    # 3. proxy_redirect 改写 Location header，补回端口
    proxy_redirect http://git.new-domain.com/ http://git.new-domain.com:8080/;

    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
}
```

## CDN 配置

如果使用 Cloudflare 等 CDN 代理：

- Cloudflare 支持代理 HTTP 8080 端口
- SSL/TLS 模式须设为 **Flexible**（CDN → 源站走 HTTP）
- 若设为 Full，CDN 会用 HTTPS 回源，源站是 HTTP 会返回 502

## 排查命令速查

```bash
# 服务状态
sudo gitlab-ctl status

# 本地测试
curl -I http://localhost:8080

# 端口监听
ss -tlnp | grep 8080

# Puma 日志（最关键）
tail -50 /var/log/gitlab/puma/current

# Workhorse 日志
tail -20 /var/log/gitlab/gitlab-workhorse/current

# Nginx 错误日志
tail -20 /var/log/gitlab/nginx/gitlab_error.log

# 内存和 OOM
free -h
dmesg | grep -i "oom\|killed" | tail -20
```

## 总结

1. **`external_url` 的端口会连带影响 Puma**：非标准端口时必须手动设置 `puma['port']` 避开冲突
2. **`X-Forwarded-Ssl: on` 优先级高于 `X-Forwarded-Proto`**：切换协议时两者都要改
3. **反代 Host header 必须传域名+端口**：传内网 IP 会导致 GitLab 生成错误的链接
4. **`proxy_redirect` 不可少**：反代监听端口与外部访问端口不一致时，需要它改写 Location header
5. **CDN SSL 模式必须匹配源站协议**：HTTP 源站对应 Flexible，HTTPS 源站对应 Full

