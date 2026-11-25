# Hugo 博客从 Algolia 换成 Fuse.js 本地搜索


## 背景

博客的站内搜索以前用的是 Algolia（见[《hugo添加Algolia搜索系统》](/hugo-loveit-algolia/)）：构建时生成索引文件，再用 `atomic-algolia` 上传到 Algolia，页面上的搜索框直接请求 Algolia 的接口。

这次升级 Hugo、换 FixIt 主题时（见[《Hugo 升级到 0.170 并从 LoveIt 迁移到 FixIt 主题》](/hugo-upgrade-fixit/)）发现，线上的搜索框**不报错，但什么都搜不到**。

## 1. 排查：Algolia 应用已经被回收

打开浏览器开发者工具的 Network 面板，搜索请求发往 `<应用ID>-dsn.algolia.net`，直接失败（`ERR_NAME_NOT_RESOLVED`）。在命令行确认一下：

```bash
# 自己的应用：解析不到
getent hosts <应用ID>-dsn.algolia.net

# 对照：Algolia 官方的演示应用能正常解析，说明不是网络问题
getent hosts latency-dsn.algolia.net
```

Algolia 每个应用都有独立的域名，域名都解析不到，说明应用已经不存在了。登录 Algolia 后台，账号下只剩一个空的 “My First Application”。

原因是 Algolia **免费套餐的应用长期不活跃会被自动删除**：官方帮助中心的说法是连续两个月左右没有任何操作（搜索、上传索引、修改配置）就会被暂停并删除，不同文章给的时间略有出入（[How can I prevent my inactive application on a free plan from being deleted](https://support.algolia.com/hc/en-us/articles/26409293431185-How-can-I-prevent-my-inactive-application-on-a-free-plan-from-being-deleted)）。个人博客访问量小、更新又少，很容易触发。

而 LoveIt 主题的搜索代码在请求失败时不会提示错误，只显示没有结果，所以一直没发现。

## 2. 为什么换成本地搜索

FixIt 主题支持 4 种搜索：`fuse`、`algolia`、`pagefind`、`cse`。其中 [Fuse.js](https://www.fusejs.io/) 是纯前端的模糊搜索库：构建时生成 `search.json`，浏览器第一次搜索时下载下来，在本地检索。

| | Algolia | Fuse.js |
|---|---|---|
| 外部依赖 | 需要 Algolia 账号，发布时用 Admin Key 上传索引 | 无，索引随站点一起发布 |
| 发布流程 | 多一步 `npm install` + 上传 | 只要 Hugo |
| 会不会失效 | 不活跃会被回收 | 不会 |
| 首次搜索 | 立即返回 | 需要先下载索引 |
| 中文效果 | 分词、排序更好 | 需要调参（见下文） |

我的博客 80 多篇文章，`search.json` 约 738 KB，GitHub Pages 会 gzip 压缩，实际传输约 230 KB，完全可以接受。文章多到几百篇以后，可以考虑 FixIt 同样支持的 Pagefind（按需分块加载索引）。

## 3. 配置 Fuse.js

```toml
[outputs]
  # 首页输出里要有 search，才会生成 search.json
  home = ["html", "rss", "archives", "search", "offline", "manifest"]

[params.search]
  enable = true
  type = "fuse"
  snippet_length = 50
  anchorify = false
```

`anchorify` 默认是 `true`，会按文章的小标题把内容拆成多条索引，搜索结果能直接跳到对应的小标题。但我实测发现，一段内容下面有多个小标题时，主题会把**同一段内容重复收录**多次（只是对应的标题不同）：我的站点有 18 组重复，涉及 95 条索引。用 Fuse 的话表现为搜索结果里出现重复项；用 Algolia 的话更糟，重复的 `objectID` 上传时会互相覆盖。所以我关掉了，按整篇文章建索引，和以前 LoveIt 的行为一致。

## 4. 中文调参

换成 Fuse 后，搜英文没问题，搜中文却很奇怪：有的明明文章里有的词搜不到，有的又搜出一堆不相关的文章。

### 4.1 原因

FixIt 给 Fuse 的默认参数是：

```toml
[params.search.fuse]
  threshold = 0.3
  location = 0
  distance = 100
  ignore_location = false
  use_extended_search = false
  ignore_field_norm = false
```

这套参数适合英文短文本，用在中文长文章上有三个问题：

1. **只在内容开头附近找**：`ignore_location = false` 时，匹配位置离 `location`（0，即开头）越远扣分越多，超出大约 `threshold × distance` = 30 个字符基本就匹配不上了。而每条索引的内容最长 4000 字，词出现在后半部分就搜不到。
2. **模糊匹配太宽**：`threshold = 0.3` 对英文是容忍拼写错误，对中文就是零散的几个字凑上了也算匹配，结果里一堆不相关的文章。
3. **多个词不能组合**：`docker 网络` 会被当成一个包含空格的整体去模糊匹配，而不是“同时包含 docker 和 网络”。

### 4.2 实测对比

为了不凭感觉调，我写了个小脚本，用博客真实的 `search.json` 和主题自带的 Fuse.js，统计不同参数下的搜索结果，跟“正文里确实包含这个词的文章”做对比：

```js
// fusetest.mjs —— 放在博客根目录，先 hugo 构建，再运行：node fusetest.mjs public/search.json
import Fuse from './themes/FixIt/assets/lib/fuse/fuse.mjs';
import fs from 'node:fs';

const data = JSON.parse(fs.readFileSync(process.argv[2], 'utf8'));
// 标准答案：标题或正文里确实包含所有关键词的文章
const truth = q => new Set(data
  .filter(d => q.split(/\s+/).every(t => (d.title + d.content).toLowerCase().includes(t.toLowerCase())))
  .map(d => d.uri));

const base = { keys: ['content', 'title', 'heading'], includeMatches: true, shouldSort: true, minMatchCharLength: 2 };
const cfgs = {
  默认: { ...base, threshold: 0.3, location: 0, distance: 100, ignoreLocation: false },
  推荐: { ...base, threshold: 0.1, ignoreLocation: true, useExtendedSearch: true, ignoreFieldNorm: true },
};

const queries = ['多账号', '负载均衡', '链路追踪', '性能调优', '消息队列', '网络通信', 'docker 网络', 'kubernets', 'prometheus 告警'];
for (const q of queries) {
  const t = truth(q);
  const row = Object.entries(cfgs).map(([name, opt]) => {
    const r = [...new Set(new Fuse(data, opt).search(q).map(x => x.item.uri))].slice(0, 10);
    return `${name}: ${r.length} 条，正确 ${r.filter(u => t.has(u)).length} 条`;
  });
  console.log(q, `（实际 ${t.size} 篇）`, row.join(' | '));
}
```

结果（搜索框最多显示 10 条）：

| 关键词 | 实际包含该词 | 默认参数 | 调整后 |
|---|---|---|---|
| 负载均衡 | 4 篇 | 0 条 | 4 条，全对 |
| 消息队列 | 9 篇 | 1 条 | 9 条，全对 |
| 性能调优 | 1 篇 | 6 条，**全错** | 1 条，正确 |
| 网络通信 | 1 篇 | 0 条 | 1 条，正确 |
| docker 网络 | 5 篇 | 10 条，对 4 条 | 5 条，全对 |
| prometheus 告警 | 2 篇 | 2 条，全对 | 2 条，全对 |

我一共测了 9 组关键词，调整后的结果全部和标准答案一致。

### 4.3 调整后的参数

```toml
[params.search.fuse]
  # 在全文任意位置匹配
  ignore_location = true
  # 收紧模糊匹配，避免零散汉字凑出不相关结果
  threshold = 0.1
  # 空格分隔多个词 = 同时包含；支持 '词 精确匹配、!词 排除
  use_extended_search = true
  # 排序不受文章长短影响，避免短文章总排在前面
  ignore_field_norm = true
```

打开 `use_extended_search` 后，搜索框支持这些写法：

| 写法 | 含义 |
|---|---|
| `docker 网络` | 同时包含 docker 和 网络 |
| `'etcd` | 精确包含 etcd |
| `git !gitlab` | 包含 git，但不包含 gitlab |

代价是英文拼错时容错变差了：错一个字母一般还能搜到，错多了就搜不到。对技术博客来说，精确比模糊更重要。

### 4.4 繁体搜不到简体

还有一个限制：Fuse 是按字符匹配的，不知道“裝”和“装”是同一个字。博客文章都是简体，用繁体输入法搜“安裝”一条都搜不到，搜“安装”有 34 篇。

如果需要支持，可以在项目的 `assets/js/custom.js`（FixIt 会自动加载）里加一张繁转简对照表，搜索前先把关键词转成简体。我自己切到简体输入就行，暂时没做。

## 5. 清理 Algolia

- 删除配置里的 `[params.search.algolia]`，以及 LoveIt 时代的 `layouts/_default/list.algolia.json`。
- 删除只用来装 `atomic-algolia` 的 `package.json`。
- 发布脚本里去掉上传索引那一步，发布不再需要 Node。

> 注意顺序：如果 Algolia 应用还正常，应该**先发布用 Fuse 的新站点，再删除 Algolia**，否则线上搜索会在中间这段时间失效。

## 总结

- 免费的第三方服务对低频使用的个人博客来说并不可靠，会在你不知道的时候失效。
- Fuse.js 本地搜索零依赖，几十到上百篇文章的规模完全够用，但**默认参数对中文不友好**，至少要打开 `ignore_location` 并调低 `threshold`。
- 调参别凭感觉，用真实的索引写个脚本对比一下，几分钟就能看出差别。


---

> 作者: [Jeff](https://luzhifang.github.io)  
> URL: https://luzhifang.github.io/hugo-fuse-search/  

