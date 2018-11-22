---
title: hexo+githubpages搭建个人博客
date: 2017-06-21 10:21:51
tags: 学习
---

## Hexo安装，hexo基于node，首先需要安装nodejs  
* 安装hexo  

`npm install -g hexo`  
`hexo version`查看hexo版本，确定是否安装完成
<!--more-->
## 有关_config.yml配置
### Site 站点配置

* title: Hexo-demo #网站标题
* subtitle: hexo is simple and easy to study #网站副标题　　
* description: this is hexo-demo #网栈描述
* author: pomy #你的名字
* language: zh-CN #网站使用的语言
* timezone: Asia/Shanghai #网站时区
### URL可以不用配置

* url: http://yoursite.com #网址，搜索时会在搜索引擎中显示
* root: / #网站根目录
* permalink: :year/:month/:day/:title/ #永久链接格式
* permalink_defaults: #永久链接中各部分的默认值
### Directory 目录配置
* source_dir: source #资源文件夹，这个文件夹用来存放内容
* public_dir: public #公共文件夹，这个文件夹用于存放生成的站点文件
* tag_dir: tags #标签文件夹
* archive_dir: archives #归档文件夹
* category_dir: categories #分类文件夹
* code_dir: downloads/code #Include code 文件夹
* i18n_dir: :lang #国际化文件夹
* skip_render: #跳过指定文件的渲染，您可使用 glob 来配置路径
### Writing 写作配置
* new_post_name: :title.md # 新文章的文件名称
* default_layout: post #默认布局
* titlecase: false # Transform title into titlecase
* external_link: true # Open external links in new tab
* filename_case: 0 #把文件名称转换为 (1) 小写或 (2) 大写
* render_drafts: false #显示草稿
* post_asset_folder: false #是否启动资源文件夹
* relative_link: false #把链接改为与根目录的相对位址
* future: true
* highlight: #代码块的设置
* enable: true
* line_number: true
* auto_detect: true
* tab_replace:
### Category & Tag 分类 & 标签
* default_category: uncategorized #默认分类
* category_map: #分类别名
* tag_map: #标签别名
### Date / Time format 时间和日期
* date_format: YYYY-MM-DD
* time_format: HH:mm:ss
### Pagination 分页
* per_page: 10 #每页显示的文章量 (0 = 关闭分页功能)
* pagination_dir: page #分页目录
### Extensions 扩展
#### Plugins: http://hexo.io/plugins/ 插件
#### Themes: http://hexo.io/themes/ 主题
* theme: landscape #当前主题名称
### Deployment #部署到github
#### Docs: http://hexo.io/docs/deployment.html
* deploy:
* type:
