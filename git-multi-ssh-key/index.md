# Git多账号SSH Key配置


## 背景

GitHub/GitLab 等平台不允许同一个 SSH 公钥绑定到多个账号。当你有多个账号（比如个人账号和工作账号）需要在同一台机器上使用 SSH 免密拉取代码时，就需要为每个账号生成独立的 SSH 密钥，并通过 `~/.ssh/config` 区分。

## 1. 为第二个账号生成新密钥

```bash
ssh-keygen -t ed25519 -C "work@example.com" -f ~/.ssh/id_ed25519_work
```

执行后会在 `~/.ssh/` 下生成两个文件：

- `id_ed25519_work` — 私钥
- `id_ed25519_work.pub` — 公钥

## 2. 配置 SSH Config

编辑 `~/.ssh/config`，为不同账号设置不同的 Host 别名：

```bash
# 个人账号（已有密钥 id_ed25519）
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/id_ed25519

# 工作账号（新密钥 id_ed25519_work）
Host github-work
  Hostname ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/id_ed25519_work
```

> 这里使用 443 端口是因为某些网络环境下 22 端口可能被封，如果网络没有限制也可以使用默认配置：
>
> ```bash
> Host github-work
>   Hostname github.com
>   User git
>   IdentityFile ~/.ssh/id_ed25519_work
> ```

## 3. 添加公钥到 GitHub

将新生成的公钥内容复制到剪贴板：

```bash
# macOS
pbcopy < ~/.ssh/id_ed25519_work.pub

# Linux
xclip -selection clipboard < ~/.ssh/id_ed25519_work.pub
```

然后到工作账号的 **Settings → SSH and GPG keys → New SSH key** 中添加。

## 4. 验证连接

```bash
# 验证个人账号
ssh -T git@github.com

# 验证工作账号
ssh -T git@github-work
```

成功时会返回类似：

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

## 5. 使用方式

### 克隆仓库

```bash
# 个人账号的仓库（不变）
git clone git@github.com:personal/repo.git

# 工作账号的仓库（使用别名）
git clone git@github-work:company/repo.git
```

### 已有仓库修改 remote

```bash
git remote set-url origin git@github-work:company/repo.git
```

### 为不同仓库设置不同的 Git 用户信息

在工作仓库目录下单独配置：

```bash
cd /path/to/work/repo
git config user.name "work-username"
git config user.email "work@example.com"
```

也可以利用 `includeIf` 按目录自动切换：

```ini
# ~/.gitconfig
[user]
    name = personal-username
    email = personal@example.com

[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
```

```ini
# ~/.gitconfig-work
[user]
    name = work-username
    email = work@example.com
```

这样 `~/work/` 目录下的所有仓库会自动使用工作账号的身份信息。

## 总结

| 步骤 | 操作 |
|------|------|
| 生成新密钥 | `ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_work` |
| 配置 SSH Config | 为新账号添加 Host 别名，指向新密钥 |
| 添加公钥 | 将 `.pub` 内容添加到对应 GitHub 账号 |
| 使用别名 clone | `git@github-work:company/repo.git` |
| 设置用户信息 | 仓库级别 `git config` 或 `includeIf` |

