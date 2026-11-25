# Hugo 升级到 0.170 并从 LoveIt 迁移到 FixIt 主题


## 背景

博客 2020 年用 Hugo + LoveIt 主题搭建（见[《基于Hugo + Github Pages搭建LoveIt主题个人博客》](/hugo-loveit/)），这几年基本没动过，这次想重新在本地写博客时发现几个问题：

1. **LoveIt 已经很久没有维护**，新版 Hugo 删掉了不少旧 API（`.Site.Author`、`.Site.IsServer`、内置的 `google_analytics_async` 模板等），旧主题用新 Hugo 很可能直接构建失败。
2. **源码和线上不一致**：线上有两篇文章在源码仓库里找不到，当时是在另一台机器上写完直接发布的，源码没推上去。
3. **站内搜索静默失效**：搜索框不报错，就是搜不到任何结果（原因和解决办法单独写在[《Hugo 博客从 Algolia 换成 Fuse.js 本地搜索》](/hugo-fuse-search/)）。

所以干脆一次性升级：Hugo 升到 0.170.1，主题换成 LoveIt 的维护分支 [FixIt](https://github.com/hugo-fixit/FixIt)，要求是**所有文章的网址保持不变**。

博客的结构和以前一样，两个仓库：

| 仓库 | 内容 |
|---|---|
| `myblog`（私有） | Hugo 源码：配置、`content/` 文章、`static/` 图片、主题子模块 |
| `<用户名>.github.io`（公开） | `hugo` 生成的静态文件，GitHub Pages 直接托管 |

## 1. 安装 Hugo 和 Dart Sass

### 1.1 Hugo

FixIt v1.0.0-rc.5 要求 Hugo **≥ 0.166.0**。WSL / Linux 下直接下载官方发布包，校验后放进 `~/.local/bin`，不需要 sudo：

```bash
V=0.170.1
D=$(mktemp -d) && cd $D
gh release download v$V -R gohugoio/hugo \
  -p "hugo_extended_${V}_linux-amd64.tar.gz" -p "hugo_${V}_checksums.txt"
grep " hugo_extended_${V}_linux-amd64.tar.gz$" hugo_${V}_checksums.txt | sha256sum -c -
tar -xzf hugo_extended_${V}_linux-amd64.tar.gz hugo
install -m755 hugo ~/.local/bin/hugo
hugo version
```

> 0.170 起普通版和 extended 版已经是**同一个二进制**（官方 checksum 文件里两个包的哈希完全相同），所以 `hugo version` 里不再显示 `+extended`，不用纠结下哪个。

### 1.2 Dart Sass

装好 Hugo 第一次构建 FixIt 会报：

```
TOCSS-DART: failed to transform "/scss/main.scss" (text/x-scss).
You need to install Dart Sass, see https://gohugo.io//functions/css/sass/#dart-sass
```

新版 Hugo 不再内置 LibSass，FixIt 的样式是 SCSS 写的，需要外部的 [Dart Sass](https://github.com/sass/dart-sass)：

```bash
V=1.108.2
D=$(mktemp -d) && cd $D
gh release download $V -R sass/dart-sass -p "dart-sass-${V}-linux-x64.tar.gz"
# 校验值见 GitHub 发布页每个文件旁的 sha256
mkdir -p ~/.local/share && rm -rf ~/.local/share/dart-sass
tar -xzf dart-sass-${V}-linux-x64.tar.gz -C ~/.local/share
ln -sf ~/.local/share/dart-sass/sass ~/.local/bin/sass
sass --version
```

## 2. 替换主题：LoveIt → FixIt

主题用 git 子模块管理，并**固定在某个 tag**，避免主题更新把博客搞坏：

```bash
# 移除 LoveIt 子模块
git rm themes/LoveIt

# 添加 FixIt 并切到指定版本
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
git -C themes/FixIt checkout v1.0.0-rc.5
git add .gitmodules themes/FixIt
```

**坑**：我的 `.gitignore` 里以前写了 `themes/`，`git submodule add` 会报 `The following paths are ignored by one of your .gitignore files`，把这一行删掉即可。

以后升级主题：

```bash
cd themes/FixIt && git fetch --tags && git checkout <新版本 tag> && cd -
hugo server   # 检查没问题再提交子模块指针
```

## 3. 迁移配置

### 3.1 改写成 hugo.toml

LoveIt 的 `config.toml` 有将近 1000 行，大部分是默认值。迁移时我**只写和主题默认值不同的项**，完整可选项看 `themes/FixIt/hugo.toml` 就行，新配置不到 200 行。

FixIt 的参数名从驼峰改成了下划线（snake_case），常用的对照：

| LoveIt | FixIt |
|---|---|
| `theme = "LoveIt"` | `theme = "FixIt"` |
| `paginate = 10` | `[pagination] pagerSize = 10` |
| `[Permalinks] posts = ":filename"` | `[permalinks.page] posts = "/:contentbasename/"` |
| `params.defaultTheme` | `params.default_theme` |
| `params.dateFormat` | `params.date_format` |
| `params.header.desktopMode` | `params.header.desktop_mode` |
| `params.home.profile.avatarURL` | `params.home.profile.avatar_url` |
| `params.page.toc` | `params.toc` |
| `params.page.code.maxShownLines` | `params.codeblock.max_shown_lines` |
| `params.search.algolia.appID` | `params.search.algolia.app_id` |

另外几个值得改的：

```toml
# 中文字数统计、摘要截断需要，以前一直是 false
hasCJKLanguage = true

[markup.goldmark.renderer]
  # 文章里直接写了 <img> 等 HTML
  unsafe = true

[markup.goldmark.extensions.passthrough]
  # 没用数学公式，关掉以免正文里的 $ 被当成公式分隔符
  enable = false
```

### 3.2 保持文章网址不变

旧配置是 `posts = ":filename"`，文章网址是 `/<文件名>/`。新版 Hugo 里对应的写法是：

```toml
[permalinks]
  [permalinks.page]
    posts = "/:contentbasename/"
```

### 3.3 最大的坑：主题配置是 shallow 合并

FixIt 自己的 `hugo.toml` 里写了 `outputs`、`taxonomies`、`markup` 等配置，带着 `_merge = "shallow"`。**shallow 合并只会补上项目里完全没有的顶层键**，而 `outputs`、`taxonomies`、`markup.highlight` 这些 Hugo 本身就有默认值，于是主题的设置**根本合并不进来**，表现是：

- 构建直接失败：

  ```
  failed to create page from pageMetaSource /authors/jeff:
  no taxonomy configuration found for "/authors/jeff"
  ```

  FixIt 需要 `authors` 分类，但用的是 Hugo 默认的 `tags`、`categories`。
- 构建成功，但 `public/` 里**没有 `search.json`**、`archives/` 等页面，因为 `outputs` 还是 Hugo 默认的 `["html", "rss"]`。
- 代码高亮用的是行内样式，因为 `markup.highlight.noClasses` 还是 Hugo 默认的 `true`。

解决办法是在项目配置里显式写出来：

```toml
[taxonomies]
  category = "categories"
  tag = "tags"
  collection = "collections"
  author = "authors"

[build]
  noJSConfigInAssets = true

[outputs]
  home = ["html", "rss", "archives", "search", "offline", "manifest"]
  page = ["html", "markdown"]
  section = ["html", "rss"]
  taxonomy = ["html"]
  term = ["html", "rss"]

[markup]
  [markup.highlight]
    codeFences = true
    noClasses = false
    guessSyntax = true
    lineNos = true
    lineNumbersInTable = true
```

排查这类问题可以用 `hugo config`，它会打印最终生效的完整配置，对比一下就知道哪些主题配置没合并进来。注意它输出的键名是**小写**的（比如 `[outputformats.search]`），grep 的时候别被大小写骗了。

### 3.4 清理 LoveIt 遗留文件

- `layouts/_default/list.algolia.json`：LoveIt 时代自己加的 Algolia 索引模板，FixIt 自带 `search.json`，删掉。
- `static/site.webmanifest`、`static/browserconfig.xml`：引用的图标文件根本不存在，FixIt 会按需生成，删掉。
- 站点只有 `favicon.ico`，没有 `apple-touch-icon.png`、`favicon-32x32.png` 等，主题却默认会输出这些 `<link>`，每个页面都有 4 个 404。设置 `params.app.no_favicon = true` 去掉，浏览器会自动加载根目录的 `favicon.ico`。

## 4. 找回只在线上存在的文章

这次比较幸运：LoveIt 的配置里开了 Markdown 输出：

```toml
[outputs]
  page = ["HTML", "MarkDown"]
```

所以线上仓库每篇文章目录下除了 `index.html`，还有一份 `index.md`，就是文章正文的原始 Markdown（只是开头多了一行 `# 标题`，没有 front matter）。

还原步骤：

1. 用一篇源码还在的文章验证：去掉开头的 `# 标题` 后，线上 `index.md` 和源码正文**逐行一致**，说明可以无损还原。
2. front matter 从 `index.html` 里的 `<script type="application/ld+json">` 取：`headline` 是标题，`datePublished` 是日期，`keywords` 是标签的原始写法；分类从页面上 `/categories/xxx/` 的链接取。
3. 按原有文章的格式拼回去。

顺便把其余 80 篇也和线上逐篇比了一遍，全部一致，确认源码只少了这两篇。

**经验**：FixIt 默认也有 Markdown 输出（`page = ["html", "markdown"]`），建议保留，相当于在线上多存了一份文章原文。

## 5. 验证网址没变

换主题最怕的是旧链接失效。分别列出新旧两个站点的所有页面，对比一下：

```bash
cd <旧站点目录> && find . -name index.html -not -path './.git/*' \
  | sed 's|/index.html$||' | LC_ALL=C sort > /tmp/old.txt
cd <新构建目录> && find . -name index.html \
  | sed 's|/index.html$||' | LC_ALL=C sort > /tmp/new.txt

# 旧站有、新站没有的页面
LC_ALL=C comm -23 /tmp/old.txt /tmp/new.txt
```

结果：所有文章、标签、分类、关于页的网址都没变。只少了 36 个列表分页（`/page/N/`），因为首页从每页 2 篇改成了每页 10 篇。新增了 FixIt 自带的 `/archives/`、`/authors/jeff/`。

再顺手扫一遍构建结果里失效的站内链接和图片，又发现两处文章本身引用了不存在的文件，旧站本来就是坏的。

## 6. 一键发布脚本

以前发布要手动：`hugo` 构建 → 把 `public/` 拷到 `github.io` 仓库 → 提交推送。这次写成了 `deploy.sh`：

```bash
./deploy.sh -n   # 预演：只构建，列出会新增/修改/删除多少文件
./deploy.sh      # 构建 → 同步 → 确认后提交并推送
```

核心逻辑：

```bash
# 构建
hugo --gc --minify --cleanDestinationDir --destination public

# 同步到发布仓库（--delete 删掉已经不存在的旧文件）
rsync -a --delete --exclude '.git' --exclude 'CNAME' --exclude '.nojekyll' \
  public/ "$PUBLISH_DIR/"

# 没有推送成功就退出（取消、出错、Ctrl+C）时，把发布仓库恢复成远端的样子
PUBLISHED=0
restore_publish() {
  [[ $PUBLISHED -eq 1 ]] && return
  git -C "$PUBLISH_DIR" reset -q --hard "origin/main"
  git -C "$PUBLISH_DIR" clean -fdq
}
trap restore_publish EXIT
trap 'exit 130' INT TERM HUP

cd "$PUBLISH_DIR" && git add -A
git diff --cached --shortstat
read -r -p "提交并推送？[y/N] " reply
[[ "$reply" =~ ^[Yy]$ ]] || exit 1
git commit -q -m "发布：$(date '+%Y-%m-%d %H:%M')（源码 $(git -C "$SRC_DIR" rev-parse --short HEAD)）"
git push -q origin main
PUBLISHED=1
```

`trap` 那段是后来加的：第一次发布时脚本在确认提示那一步被中断，发布仓库里留下一堆已暂存的改动，下次运行脚本就卡住了。

提交信息里带上源码的 commit，以后看线上某个版本对应哪份源码一目了然。

## 踩坑汇总

| 现象 | 原因 | 解决 |
|---|---|---|
| `TOCSS-DART ... You need to install Dart Sass` | 新版 Hugo 不再内置 LibSass | 安装 Dart Sass |
| `no taxonomy configuration found for "/authors/jeff"` | 主题的 `taxonomies` 是 shallow 合并，合并不进来 | 项目里显式写 `[taxonomies]` 并加上 `author = "authors"` |
| 没有生成 `search.json`、`archives/` | `outputs` 同上，合并不进来 | 项目里显式写 `[outputs]` |
| 代码高亮变成行内样式 | `markup.highlight.noClasses` 同上 | 显式写 `noClasses = false` |
| `git submodule add` 报路径被忽略 | `.gitignore` 里有 `themes/` | 删掉这一行 |
| 每个页面 4 个 favicon 404 | 主题默认输出多种图标链接 | `params.app.no_favicon = true` |
| （预防）开了 `core.autocrlf=true` 的机器上克隆后，`deploy.sh` 会被转成 CRLF，bash 报 `$'\r': command not found` | Windows/WSL 常见的 git 配置 | 加 `.gitattributes`：`*.sh text eol=lf` |
| 本地预览出现 `WARN 当前运行环境是“development”` | FixIt 在开发环境下不启用评论、统计等功能，只是提示 | 不用管，正式构建是 production |

## 总结

整个迁移其实没有改一行文章内容，主要的工作量在配置上，而最耗时间的坑就是**主题配置的 shallow 合并**：报错信息完全看不出和合并有关，要靠 `hugo config` 对比才能定位。

迁移完之后，写博客的流程变成：

```bash
hugo new content posts/<分类>/<文件名>.md
hugo server -D        # 本地预览，http://localhost:1313
./deploy.sh           # 发布
```


---

> 作者: [Jeff](https://luzhifang.github.io)  
> URL: https://luzhifang.github.io/hugo-upgrade-fixit/  

