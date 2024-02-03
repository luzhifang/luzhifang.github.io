# Git使用指北


## 1. 起步

1. [下载地址](https://git-scm.com/download)，各平台安装方式不一样，可以点进去细看下
2. [中文文档](https://git-scm.com/book/zh/v2)

## 2. 运行前配置

```Shell
git config --list --show-origin # 查看所有配置
git config --global user.name "luzhifang" # 配置用户名
git config --global user.email "luzhifang@example.com" # 配置邮件地址
git config --global core.editor vim # 配置文本编辑器
git config --global color.ui auto # git config --global color.ui auto
git help config # 查看config的命令手册
cat ~/.gitconfig # 查看配置文件
```

## 3. 获取帮助

```Shell
git help config # git help <verb>，查看命令手册， 和git <verb> --help 、 man git-<verb> 等价
git add -h # git <verb> -h，不需要全面的手册，可以用这个查看可用选项的快速参考
```

## 4. 基础

### 4.1 获取仓库

1. 在已存在目录中初始化仓库

```Shell
mkdir -p /home/user/my_project
cd /home/user/my_project
git init # 初始化 .git 文件夹
```

2. 克隆现有的仓库

```Shell
git clone https://github.com/luzhifang/luzhifang.github.io
```

### 4.2 基础命令

```Shell
touch README.md
touch .gitignore # 添加忽略文件
git add . # 添加到暂存区
git mv README.md README # git mv 需要添加到版本控制中才能重命名
git rm --cached README # 从暂存区移除
git rm README # 非暂存区移除
git status # 检查当前文件状态
git diff # 列出具体哪个文件哪一行修改了，加--staged将比对已暂存文件与最后一次提交的文件差异
git commit # 不带-m则可以添加换行的commit message，建议使用，Linux默认会进去vim编辑模式，按crtl+x再按y进行保存
git commit -m 'initial project version' # 提交到本地仓库，git commit -a等同于git add + git commit
git add forgotten_file
git commit --amend # 修正提交，如果上次有提交则覆盖为一次
git reset HEAD forgotten_file # 取消暂存
git checkout -- forgotten_file # 撤销文件的修改
git remote # 查看已经配置的远程仓库，-v会显示URL
git remote add origin https://github.com/luzhifang/luzhifang.github.io # git remote add <shortname> <url> 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写，可添加多个
git push origin main # git push <remote> <branch> 推送到远程仓库
git remote show origin # git remote show <remote> 查看某个远程仓库
git remote remove origin # 删除远程仓库
git remote rename origin new # 远程仓库的重命名
git config --global alias.br branch # 设置别名
```

### 4.3 标签

```Shell
git tag # 列出所有标签
git tag -l "v1.8.5" # 通配符匹配标签
git tag -a v1.5 -m "标签信息" # 打附注标签，没有-a 打轻量标签
git show tagname # 查看标签信息和与之对应的提交信息
git tag -a tagname 9fceb02 # 指定提交ID打标签
git push origin v1.5 # 推送指定标签
git push origin --tags # 推送所有标签
git tag -d tagname # 删除本地标签
git push origin --delete <tagname> # 删除远程标签
git checkout tagname # 检出标签
git checkout -b branch tagname # 检出标签到分支
```

### 4.4 文件修改

```Shell
git log # 查看commit的历史
git log --pretty=format:"%h - %an, %ar : %s" --graph # 图标显示并格式化，更多的细节可以查看官方文档和手册
git log -p # 查看某个文件的修改历史
git log -p -2 # 查看最近2次的更新内容
git log --name-status # 每次修改的文件列表, 显示状态
git log --name-only # 每次修改的文件列表
git log --stat # 每次修改的文件列表, 及文件修改的统计
git whatchanged # 每次修改的文件列表
git whatchanged –stat # 每次修改的文件列表, 及文件修改的统计
git show commitid # 查看某次commit的修改内容
git show # 显示最后一次的文件改变的具体内容
```

## 5. 分支

```Shell
git branch # 列出本地所有分支，--merged 与 --no-merged 这两个有用的选项可以过滤这个列表中已经合并或尚未合并到当前分支的分支
git checkout branch # 切换到已存在的分支
git checkout - # 切回上一个分支
git checkout -b branch # 新建分支并检出，等于 git branch branch + git checkout branch
git checkout -b branch origin/branch # 建立在远程跟踪分支之上
git branch -u origin/branch # 设置上游分支
git merge branch # 合并分支，注意合并前需要切换到你想合并入的分支
git branch -d branch # 删除分支
git push origin --delete branch # 删除远程分支
git branch -vv # 查看设置的所有跟踪分支
git branch -m oldbranch newbranch # 修改本地分支名称
git fetch origin # 拉取 origin 的仓库中有但你没有的信息
git pull # 大多数情况下它的含义是一个 git fetch 紧接着一个 git merge 命令，推荐用fetch + merge 代替pull
```

## 6. 更改提交

```Shell
git reset --hard 9fceb02 # 强制回退到某个提交ID，慎用
git push origin HEAD --force # 远端同步回滚
git rebase -i HEAD~2 # 可以选定当前分支中包含HEAD（最新提交）在内的两个最新历史记录为对象，并。
# 变基，推荐只对尚未推送或未分享给别人的本地修改执行变基操作清理历史，不对已推送的提交执行变基操作
git rebase -i # 在编辑器中打开，加上HEAD~2修正最新提交两次提交并在，可以指定commit id指定修改开始和截止区段
git rebase --continue # 修改提交信息
git push # 推送
```

## 7. 提交规范

遵循 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/v1.0.0/#summary) 来规范化 commit message

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### 7.1 type

提交的 commit 类型主要有以下几种:

> 主要类型

fix 修复 bug  
feat 新增功能  
deps 依赖修改  
break 不兼容修改

> 其他类型

perf 性能优化  
docs 文档修改  
refactor 重构  
style 代码格式  
test 测试用例  
chore 日常工作，如文档修改，示例等  
ci 构建脚本

### 7.2 scope

提交的代码修改的代码文件范围：

```
transport
examples
middleware
config
cmd
etc.
```

### 7.3 description

用简短的话语清晰的描述提交的代码做了什么事。

### 7.4 body

补充说明，用于描述原因、目的、实现逻辑等可以省略。

### 7.5 footer

当存在不兼容(breaking change)更新时，需要描述原因以及影响范围。
关联相关的 issue，如 Refs #133。
可能涉及到的文档更新和其他模块的更新的 PR 关联。
Commit Examples
只有提交信息

```
fix: The log debug level should be -1
```

包含全部结构

```
fix(log): [BREAKING-CHANGE] unable to meet the requirement of log Library

Explain the reason, purpose, realization method, etc.

Close #777
Doc change on doc/#111
BREAKING CHANGE:
  Breaks log.info api, log.log should be used instead
```

## 8. 版本规范

格式为 x.y.z

1. x 代表主版本号，在重大功能变更或新版本不向下兼容时加 1，此时 y 与 z 的数字归 0
2. y 代表次版本号，在添加新功能或者删除已有功能时加 1，此时 z 的数字归 0
3. z 代表修订号，z 只在进行内部修改后加 1

## 9. Git Flow

```
   /add-user           /1.0.0  /1.0.1
    feature  develop  release  hotfix  master
       |        ●         |       |       |
       |      / |         |       |       |
       |    /   |         |       |       |
       |  /     |         |       |       |
       |/       |         |       |       |
start  ●        |         |       |       |
       ↓        |         |       |       |
       ●        |         |       |       |
       ↓        |         |       |       |
finish ●\       |         |       |       |
       |  \     |         |       |       |
       |PR  \   |         |       |       |
       |merge \ |         |       |       |
       |        ●         |       |       |
       |        | \       |       |       |
       |        |   \     |       |       |
       |        |     \   |       |       |
       |        |       \ |       |       |
       |        |         ● start |       |
       |        |         ↓       |       |
       |        |         ●       |       |
       |        |         ↓       |       |
       |        |         ●finish |       |
       |        |       / | \     |       |
       |        |     /   |   \   |       |
       |        |   /     |     \ |       |
       |        | /       |       \       |
       |        ●         |       | \     |
       |        |         |       |   \   |
       |        |         |       |     \ |
       |        |         |       |       ● tag 1.0.0
       |        |         |       |     / |
       |        |         |       |   /   |
       |        |         |       | /     |
       |        |         | start ●       |
       |        |         |       ↓       |
       |        |         |       ●       |
       |        |         |       ↓       |
       |        |         | finish●       |
       |        |         |     / | \     |
       |        |         |   /   |   \   |
       |        |         | /     |     \ |
       |        |         /       |       ● tag 1.0.1
       |        |       / |       |       |
       |        |     /   |       |       |
       |        |   /     |       |       |
       |        | /       |       |       |
       |        ●         |       |       |
       |        |         |       |       |
```

