---
title: How to use hexo
date: 2022-05-29 09:06:33
tags:
  - hexo
  - blog
  - 工具
---

# 参考  

https://hexo.io/docs/

<!--more-->

# 准备  

## Nodejs  

## Git

# 初始化  

```bash
node hexo init blog  # create dir blog and init
cd blog 
npm init
```

# 配置  

修改_config.yml文件，具体配置项参考文章顶部链接

# 日常使用  

```bash
node hexo clean  # clean
node hexo s  # start server to preview
```

修改主题：

下载主题到theme目录下，修改_config.yml下theme配置即可

# 配合github pages使用

_config.yml配置github仓库以及branch

一键deploy需要安装插件：hexo-github-dploy

search功能插件：hexo-search-db

具体参考文章顶部链接



# 插件

```bash
npm install --save hexo-filter-mermaid-diagrams  # markdown mermaid 在主题配置文件中找到mermaid选项，将enable设置为true hexo不支持markdown mermaid渲染. 该插件需要使用直接使用mermaid语法书写
```

