# Markdown指北


## Markdown 简介

- Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档。
- Markdown 语言在 2004 由约翰·格鲁伯（英语：John Gruber）创建。
- Markdown 编写的文档可以导出 HTML 、Word、图像、PDF、Epub 等多种格式的文档。
- Markdown 编写的文档后缀为 .md, .markdown。

## Markdown 应用

- Markdown 能被使用来撰写电子书，如：Gitbook。
- 当前许多网站都广泛使用 Markdown 来撰写帮助文档或是用于论坛上发表消息。例如：GitHub、简书、reddit、Diaspora、Stack Exchange、OpenStreetMap 、SourceForge 等。

## Markdown 编辑器

- Typora 编辑器：Typora 支持 MacOS 、Windows、Linux 平台，且包含多种主题，编辑后直接渲染出效果。支持导出 HTML、PDF、Word、图片等多种类型文件。
  官网：https://typora.io/
- 在线编辑器
- 其他的文本编辑器(例：VSCODE + 插件)，IDE(idea + 插件)

## Markdown 标题

1. 使用 # 号可表示 1-6 级标题，一级标题对应一个 # 号，二级标题对应两个 # 号，以此类推。

```Markdown
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

## Markdown 换行和段落

1. 段落的换行是使用两个以上空格加上回车
2. 在段落后面使用一个空行来表示重新开始一个段落

```Markdown
段落段落段落段落段落段落段落段落段落段落段落段落段落段落段落段落段落段落段落段落
段落

段落
```

## Markdown 字体样式

```Markdown
_斜体文本_
_斜体文本_
**粗体文本**
**粗体文本**
**_粗斜体文本_**
**_粗斜体文本_**
```

_斜体文本_
_斜体文本_
**粗体文本**
**粗体文本**
**_粗斜体文本_**
**_粗斜体文本_**

## Markdown 分隔线

你可以在一行中用三个以上的星号、减号、下划线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。

```Markdown
***
---
___
```

---

---

---

## Markdown 删除线，两边加~~

```Markdown
~~删除线~~
```

~~删除线~~

## Markdown 下划线

```Markdown
<u>下划线</u>
```

<u>下划线</u>

## Markdown 列表

1 无序列表使用星号(\*)、加号(+)或是减号(-)作为列表标记，这些标记后面要添加一个空格

```Markdown
- 第一项
- 第二项

* 第一项
* 第二项

- 第一项
- 第二项
```

- 第一项
- 第二项

* 第一项
* 第二项

- 第一项
- 第二项

2 有序列表使用数字并加上 . 号

```Markdown
1. 第一项
2. 第二项
3. 第三项
```

1. 第一项
2. 第二项
3. 第三项

3 列表嵌套只需在子列表中的选项前面添加四个空格即可：

```Markdown
1. 第一项：
   - 第一项嵌套的第一个元素
   - 第一项嵌套的第二个元素
2. 第二项：
   - 第二项嵌套的第一个元素
   - 第二项嵌套的第二个元素
```

1. 第一项：
   - 第一项嵌套的第一个元素
   - 第一项嵌套的第二个元素
2. 第二项：
   - 第二项嵌套的第一个元素
   - 第二项嵌套的第二个元素

## Markdown 区块

Markdown 区块引用是在段落开头使用 > 符号 ，然后后面紧跟一个空格符号：

```Markdown
> 区块引用
> 区块引用
> 区块引用

> 最外层
>
> > 第一层嵌套
> >
> > > 第二层嵌套
```

> 区块引用
> 区块引用
> 区块引用

> 最外层
>
> > 第一层嵌套
> >
> > > 第二层嵌套

## Markdown 代码

段落上的一个函数或片段的代码可以用反引号把它包起来（`）

```Markdown
`printf()`函数
```

`printf()`函数

你也可以用 ``` 包裹一段代码，并指定一种语言（也可以不指定）

```javascript
$(document).ready(function () {
  alert("Hello world!");
});
```

## Markdown 链接

```Markdown
[百度](https://baidu.com)

<https://baidu.com>
```

[百度](https://baidu.com)

<https://baidu.com>

```Markdown
这个链接用 a 作为网址变量 [百度][a]，然后在文档的结尾为变量赋值（网址）

[a]: http://www.baidu.com/
```

这个链接用 a 作为网址变量 [百度][a]，然后在文档的结尾为变量赋值（网址）

[a]: http://www.baidu.com/

## Markdown 图片

```Markdown
![alt 属性文本](https://demo.png "可选标题")

这个链接用 mark 作为网址变量 ![Markdown][mark]
然后在文档的结尾为变量赋值（网址）

[mark]: https://demo.png

<img src="https://demo.png" width="50%" />
```

## Markdown 表格

Markdown 制作表格使用 | 来分隔不同的单元格，使用 - 来分隔表头和其他行。

```Markdown
| 表头   | 表头   |
| ------ | ------ |
| 单元格 | 单元格 |
| 单元格 | 单元格 |

我们可以设置表格的对齐方式：

-: 设置内容和标题栏居右对齐。
:- 设置内容和标题栏居左对齐。
:-: 设置内容和标题栏居中对齐。

| 左对齐左对齐 | 右对齐右对齐 | 居中对齐居中对齐 |
| :----------- | -----------: | :--------------: |
| 单元格       |       单元格 |      单元格      |
| 单元格       |       单元格 |      单元格      |
```

| 表头   | 表头   |
| ------ | ------ |
| 单元格 | 单元格 |
| 单元格 | 单元格 |

我们可以设置表格的对齐方式：

-: 设置内容和标题栏居右对齐。
:- 设置内容和标题栏居左对齐。
:-: 设置内容和标题栏居中对齐。

| 左对齐左对齐 | 右对齐右对齐 | 居中对齐居中对齐 |
| :----------- | -----------: | :--------------: |
| 单元格       |       单元格 |      单元格      |
| 单元格       |       单元格 |      单元格      |

## Markdown 高级技巧

- 支持的 HTML 元素

不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。

目前支持的 HTML 元素有：`<kbd> <b> <i> <em> <sup> <sub> <br> <hr> <p> <img> <div> <h1> <video>`等 ，如：

```Markdown
使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑

视频

<video controls width="250">
    <source src="/media/examples/flower.mp4"
            type="video/mp4">

    Sorry, your browser doesn't support embedded videos.

</video>
```

使用 <kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>Del</kbd> 重启电脑

视频

<video controls width="250">

<source src="/media/examples/flower.mp4"
            type="video/mp4">
    Sorry, your browser doesn't support embedded videos.

</video>

- 转义

Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号

````Markdown
\\ 反斜线
\` 反引号 \* 星号
\_ 下划线
\{\} 花括号
\[\] 方括号
\(\) 小括号
\# 井字号
\+ 加号
\- 减号
\. 英文句点
\! 感叹号
\```
\```
````

\\ 反斜线
\` 反引号 \* 星号
\_ 下划线
\{\} 花括号
\[\] 方括号
\(\) 小括号
\# 井字号
\+ 加号
\- 减号
\. 英文句点
\! 感叹号
\```
\```

