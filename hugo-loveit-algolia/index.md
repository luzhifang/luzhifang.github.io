# hugo添加Algolia搜索系统


## 1. 注册 Algolia

1. 注册 [algolia](https://www.algolia.com/)
2. 添加索引 data sources
3. 点击 Settings -> API Keys，查看自己的 Application ID 和 Search-Only API Key

## 2. 生成索引文件

1. 修改 config.toml

注意 outputs 下面 home 的末尾要有"Algolia"，漏了生成的名字就不一样了

```
[outputs]
    home = ["HTML", "RSS", "Algolia"]

[outputFormats.Algolia]
  baseName = "algolia"
  isPlainText = true
  mediaType = "application/json"
  notAlternative = true

[params.algolia]
  appId = "Application ID"
  indexName = "索引名字"
  searchOnlyKey = "Search-Only API Key"
```

2. 根目录下 layouts/\_default (没有就新建) 文件夹中新建 list.algolia.json 文件，内容如下

注意如果索引文件太多，则需要过滤一些内容作为索引

```
// 把内容也添加索引
{{- range $index, $entry := .Site.RegularPages }}
{{- if $index }}, {{ end }}
{
  "objectID": {{ .File.TranslationBaseName }},
  "url": {{ .Permalink | jsonify }},
  "title": {{ .Title | jsonify }},
  "summary": {{ .Summary | jsonify }},
  "content": {{ .Plain | jsonify }},
  "pubDate": {{ .PublishDate | jsonify }}
}
{{- end }}
```

```
// 过滤内容
{{- $.Scratch.Add "index" slice -}}
{{- $section := $.Site.GetPage "section" .Section}}
{{- range .Site.AllPages -}}
  {{- if or (and (.IsDescendant $section) (and (not .Draft) (not .Params.private))) $section.IsHome -}}
    {{- $.Scratch.Add "index" (dict "objectID" .UniqueID "date" .Date.UTC.Unix "description" .Description "dir" .Dir "expirydate" .ExpiryDate.UTC.Unix "fuzzywordcount" .FuzzyWordCount "keywords" .Keywords "kind" .Kind "lang" .Lang "lastmod" .Lastmod.UTC.Unix "permalink" .Permalink "publishdate" .PublishDate "readingtime" .ReadingTime "relpermalink" .RelPermalink "summary" .Summary "title" .Title "type" .Type "url" .URL "weight" .Weight "wordcount" .WordCount "section" .Section "tags" .Params.Tags "categories" .Params.Categories "authors" .Params.Authors)}}
  {{- end -}}
{{- end -}}
{{- $.Scratch.Get "index" | jsonify -}}
```

3. 执行`hugo`命令，在 public 目录下会生成`algolia.json`文件

## 3. 上传索引文件

1. 手动上传，在 Algolia 网站内，点左下侧边栏 Data Sources -> Indices，再点 Upload record 按钮上传生成的 algolia.json 文件。
2. 自动上传

   1. 安装 node 和 npm，执行 `npm init`，再执行`npm install atomic-algolia --save`，再执行 `npm install`
   2. 修改 package.json 文件，scripts 下添加 `"algolia": "atomic-algolia"`

      ```
      "scripts": {
          "test": "echo \"Error: no test specified\" && exit 1",
          "algolia": "atomic-algolia"
      },
      ```

   3. 根目录下新建 .env 文件，内容如下：

      ```
        ALGOLIA_APP_ID=你的Application ID
        ALGOLIA_INDEX_NAME=你的索引名字
        ALGOLIA_INDEX_FILE=public/algolia.json
        ALGOLIA_ADMIN_KEY=你的Admin API Key
      ```

   4. 执行 `npm run algolia`

