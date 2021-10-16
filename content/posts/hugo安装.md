---
title: "Hugo安装md"
date: 2021-10-16T16:45:33+08:00
tags: [hugo]
categories: [安装]
---

### STEP1安装HUGO:

https://github.com/gohugoio/hugo/releases

在github官网下载hugo客户端程序

Win10下，要把hugo.exe命令配置到环境变量，或者直接放到C:\Windows\System32

如果用到extended版本，给hugo.exe改个名字，比如hugoE.exe

### STEP2环境：

新站点先创建脚手架

```bash
hugo new site /path/to/site
```

旧站点把项目先用git同步，然后hugo命令编译

```bash
hugo
```

### STEP3新建文章

新建文章存放在content。如果使用了主题，要放在主题设定的文件夹，比如content/posts。

```bash
hugo new relative_path

本地启动预览
hugo server --theme=meme --buildDrafts
```

附赠元数据模板：

```
title: "Hugo安装md"
date: 2021-10-16T16:45:33+08:00
tags: [hugo]
categories: [安装]
```

### STEP4上传：

hugo的部署文件在public文件夹。

可以把public单独上传到你的主页仓库，hugo的源文件单独上传到其他仓库
