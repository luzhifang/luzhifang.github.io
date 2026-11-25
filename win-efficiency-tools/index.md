# Windows 11 效率工具推荐


## 总览

这篇整理一下我在 Windows 11 上装的效率工具。挑选的标准有两个：**能明显省时间**，**常驻时占用资源少**。

| 工具 | 用途 | 类似的 Mac 工具 | winget ID |
|---|---|---|---|
| Everything | 全盘文件名秒搜 | Spotlight（找文件） | `voidtools.Everything` |
| Flow Launcher | 启动器：开程序、搜文件、算数、跑命令 | Alfred / Raycast | `Flow-Launcher.Flow-Launcher` |
| QuickLook | 选中文件按空格预览 | 快速查看（Quick Look） | `QL-Win.QuickLook` |
| PixPin | 截图、贴图、OCR | CleanShot X / Snipaste | `PixPin.PixPin` |
| pot | 划词翻译、OCR 翻译 | Bob | `Pylogmon.pot` |
| AltSnap | 按住 Alt 拖动、缩放窗口 | — | `AltSnap.AltSnap` |
| NanaZip | 压缩解压 | The Unarchiver | `M2Team.NanaZip` |
| 右键菜单增强 | 右键打开 WSL / VS Code、复制路径等 | — | 见下文 |

除了最后一个，都可以用 Windows 自带的 winget 安装，例如：

```powershell
winget install voidtools.Everything Flow-Launcher.Flow-Launcher QL-Win.QuickLook
```

## Everything + Flow Launcher

这两个是提升最大的，搭配使用：Everything 负责建索引、搜索，Flow Launcher 负责输入框和显示结果。

### Everything：为什么这么快

Windows 自带的搜索要先扫描文件建立索引，又慢又不全。Everything 直接读取 NTFS 的主文件表（MFT），再通过 USN 日志实时感知文件变化，所以全盘几十万个文件名，打字的同时结果就出来了，常驻内存也只有几十 MB。

代价是它**只搜文件名和路径**，不搜文件内容。

### Flow Launcher：接入 Everything

Flow Launcher 官方支持 Everything，不用装插件，默认用的是 Windows 索引，在设置里切换一下即可：

1. 打开 Flow Launcher 设置 → 左侧「插件」。
2. 找到「文件管理器」（Explorer），点击展开。
3. 展开**「通用设置」**，有三个下拉框：
   - **索引搜索引擎**：改成 `Everything`。平时按文件名搜索走的就是它。
   - **目录递归搜索引擎**：改成 `Everything`。
   - **文件内容搜索引擎**：可以不改。Everything 1.4 不给文件内容建索引，用它搜内容会逐个读文件，很慢。

几个注意点：

- Everything 必须**一直在后台运行**。Flow Launcher 是通过进程间通信（IPC）把查询发给 Everything 的。
- Everything 要装**完整版**。Lite 版去掉了 IPC，Flow Launcher 连不上。

### Flow Launcher 常用关键字

默认呼出快捷键是 `Alt+Space`。直接输入就是全局搜索（程序、文件、计算器、网址都在里面），加前缀可以只用某个插件：

| 输入 | 作用 |
|---|---|
| `notepad` | 搜程序、文件 |
| `1024*3` | 计算器 |
| `> ping 1.1.1.1` | 运行命令 |
| `kill chrome` | 结束进程 |
| `s xxx` | 搜 Windows 设置项，回车直接打开对应页面 |
| `b xxx` | 搜浏览器书签 |
| `pm install xxx` | 安装插件 |

### 让 Everything 也能搜到 WSL 里的文件

Everything 默认只扫描 NTFS 盘。WSL2 的文件在一个 ext4 格式的虚拟磁盘里，Windows 只能通过 `\\wsl.localhost\...` 这种网络路径访问，所以默认搜不到。

在 Everything 里加一个文件夹索引就行：

1. **工具 → 选项 → 索引 → 文件夹 → 添加**，填 `\\wsl.localhost\Ubuntu\home\<用户名>`。
   - 只加 home 目录。加整个 `\\wsl.localhost\Ubuntu` 的话，`/usr`、`/proc` 这些系统目录也会被扫进去。
2. 给这个文件夹设置**定时重新扫描**，我设的是每 6 小时一次。
   - 网络路径上的文件变化 Everything 基本感知不到，只能靠定时重扫，所以新建的文件要等下一次扫描才搜得到。
   - 每次扫描都会访问 `\\wsl.localhost`，WSL 没开时会被唤醒。间隔不要设得太短。
3. **排除没用的目录**：工具 → 选项 → 索引 → 排除，在「排除文件夹」列表旁点**「添加筛选器」**，逐个添加目录名：

```
node_modules
.git
target
.venv
.cache
```

第 3 步很值得做。我的 home 目录下有将近 69 万个文件，其中 `.git`、`node_modules`、`target` 这几类目录就占了七成。排除之后只剩 20 万左右，扫描快很多，搜索结果也干净。

筛选器按目录名匹配、全局生效，C 盘上同名的文件夹也会一起排除，正好把这类没必要搜的目录一次清掉。

验证方法：在 Everything 里搜 `\\wsl.localhost\ node_modules`，没有结果，说明排除生效了；再搜 WSL 里一个确定存在的文件名，能搜到，说明索引生效了。

### Flow Launcher 的内存只涨不降？

用一段时间后，任务管理器里 Flow Launcher 能到三四百 MB，而且降不下来。这多数不是内存泄漏：

- 它是 .NET 程序。.NET 的垃圾回收在回收对象之后，不急着把内存还给系统，而是留着下次复用。所以任务管理器里的数字会停在用到过的最高点。
- 「内存」那一栏包含 .NET 运行时和 WPF 的 DLL，这部分可以和其他进程共享。真正属于它自己的占用要看「专用工作集」或私有内存。
- 程序索引、浏览器书签、图标缓存都是常驻的，用得越多缓存越大。

判断方法是看几天内的趋势：涨到某个值后就稳定了，就是正常现象；每天持续上涨，才可能是泄漏。不用浏览器书签搜索的话，可以把 Browser Bookmarks 插件关掉，能省一些。

## QuickLook：按空格预览

在资源管理器里选中文件，按空格就弹出预览，再按一次空格关闭，用法和 Mac 的快速查看一样。图片、视频、PDF、Markdown、代码、压缩包都能直接看。

**预览 Office 文件要装插件。** 我没装 Office，用的是 [OfficeViewer-Native](https://github.com/QL-Win/QuickLook.Plugin.OfficeViewer-Native) 插件，它自己解析文件，不依赖 Office，Word、Excel、PowerPoint 都能看。安装方法是去 GitHub 下载 `.qlplugin` 文件，选中它按空格，点安装。复杂的图表和透视表可能显示得不完整，看个大概足够了。

安装时用 winget 的 `QL-Win.QuickLook`，这是 GitHub 上的开源原版，插件生态最全。

## PixPin：截图和贴图

默认 `F1` 截图，`F3` 把截图贴在屏幕上。贴图会一直浮在最上层，写代码时对照设计图、对照报错信息都很方便。另外支持长截图、录 GIF、OCR 识别文字、标注打码。

> 安装后 `winget list` 里会有两条 PixPin：除了主程序，还有一个 `PixPinContextMenu` 包。它是 PixPin 用来接入 Windows 11 新版右键菜单的组件，跟着主程序一起安装、卸载就好。

## pot：划词翻译

选中文字按快捷键（我设的是 `Alt+T`），弹出翻译窗口。可以同时开好几个翻译服务（DeepL、Bing、Google 等），结果放在一起对照。另外支持截图 OCR 再翻译，看英文文档、报错信息时用得很多。

## AltSnap：拖动窗口不用找标题栏

按住 `Alt`，在窗口任意位置：

- 左键拖动：移动窗口
- 右键拖动：调整大小
- 中键点击：弹出窗口菜单（置顶、最大化、关闭等）

窗口标题栏被遮住，或者窗口超出屏幕时特别好用。拖到屏幕边缘也会像系统一样自动分屏。程序很小，几乎不占资源。

## NanaZip：压缩解压

NanaZip 是从 7-Zip 分出来的开源项目，压缩引擎一样，主要区别在于适配了 Windows 11：

- 支持 Win11 的新版右键菜单，右键直接能看到“解压到…”，不用再点「显示更多选项」。
- 界面跟随系统的深色模式。

用法和 7-Zip 一样。原来用 7-Zip 的话，可以直接换。

## 右键菜单增强

这是我自己写的小工具：[win-context-menu](https://github.com/luzhifang/win-context-menu)。给资源管理器的右键菜单加上程序员常用的几项：

- 在此处打开 WSL；用 VS Code（本地或 WSL）、GoLand 打开
- 复制路径：Windows 路径、WSL 路径（`/mnt/c/...`）、文件名
- 新建文件模板：Markdown、Shell、JSON、YAML、Go 等
- 可选恢复 Win11 经典右键菜单

它只改当前用户的注册表，**不常驻、不往资源管理器里加载 DLL**，可以在「设置 → 应用」里卸载。安装：

```powershell
irm https://raw.githubusercontent.com/luzhifang/win-context-menu/main/get.ps1 | iex
```

## 系统自带，不用另装

有些功能 Windows 11 自己就有，只是默认没打开或者不常被提到：

| 功能 | 用法 |
|---|---|
| 剪贴板历史 | `Win+V`，第一次用要按提示打开。能看到最近复制过的内容，也能固定常用的 |
| 窗口分屏 | `Win+Z` 选布局，或者把窗口拖到屏幕顶部 |
| 文件管理器标签页 | `Ctrl+T` 新建标签页 |
| 截图 | `Win+Shift+S`。简单截图够用，要贴图、标注再用 PixPin |
| 表情和符号 | `Win+.` |
| 软件安装 | `winget`，类似 Mac 的 Homebrew，本文的工具都能用它安装和升级 |

## 总结

- 最值得装的是 **Everything + Flow Launcher**：把 Flow Launcher 的搜索引擎切到 Everything，再给 Everything 配好排除目录，找文件基本就不用再开资源管理器了。
- 其他几个都是解决某个具体痛点的小工具，按需安装。
- 装之前先想想系统自带的功能够不够用，能少一个常驻进程就少一个。


---

> 作者: [Jeff](https://luzhifang.github.io)  
> URL: https://luzhifang.github.io/win-efficiency-tools/  

