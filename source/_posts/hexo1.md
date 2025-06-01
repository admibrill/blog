---
title: Hexo博客0氪建站记录（上）
date: 2024-08-23 11:00:56
top_img: /img/hexo.jpeg
cover: /img/hexo.jpeg
type: "post"
categories:
  - Hexo
tags:
  - Hexo
  - 建站
  - 免费
---
## 引言

大家好，我叫Admibrill，~~建站十多天了，没有发过一次文章~~。

第一篇文章，我决定记录一下我个人的建站历程。

## 各种环境依赖的安装

跟着[Fomal大佬的教程](https://www.fomal.cc/posts/e593433d.html)，我开始了建站。 

### Github账号和仓库

我本来就有一个github的账号，于是建了个仓库叫`admibrill.github.io`。

使用`Github Pages`的源文件仓库要命名为`<username>.github.io`。

这代表网站源码要通过github托管，会使用`Github Pages`部署，别人可以通过`https://admibrill.github.io`来访问我的网站。

### 本地源文件存储文件夹

在你喜欢的位置，建一个文件夹，起一个喜欢的名字（建议不要有汉字）。

不同的人起的名字不同，这个文件夹就用`[blogroot]`来指代。

### node.js和git的安装

<https://nodejs.org/en/download/>

[Git - Downloads (git-scm.com)](https://git-scm.com/downloads)

下载安装之后，打开`[blogroot]`，右键单击空白处，即可发现菜单中多出来了`Git Bash Here`一项

 ![](https://image.m-c.top/?/images/2024/08/23/NuXMqVMoUZ/1111.png) 

打开它，然后后面的很多命令都使用bash输入。

### Hexo的安装

```bash
npm install -g hexo-cli
```

然后，因为我是第一次使用，配置用户名和邮箱

```bash
git config --global user.name "<your username>"
git config --global user.email "<your email>"
```

### 连接至github

```bash
ssh-keygen -t rsa -C "<your email>"
```

## Hexo，启动！！！

### 初始化Hexo

打开`[blogroot]`,右键选择‘`Git bash here`。

```bash
hexo init
```

这时，我们的`[blogroot]`中缓缓出现了很多文件。

Hexo会自动安装依赖包。

等待程序执行完成，命令行中会出现一句`Start blogging with Hexo!`就代表Hexo初始化成功了。

### 安装挂载依赖包

有一个包Hexo不会自动安装，叫做`hexo-deployer-git`。

我们就要使用npm的安装命令安装它(以后安装一些包通用)。

```bash
npm install hexo-deployer-git --save
```

等待运行完成就好了。

### 修改配置文件

打开`[blogroot]`目录下的`_config.yml`文件，修改以下设置：

```yaml
url: https://<your username>.github.io
root: /
```

```yaml
deploy:
	type: git
	repository: https://github.com/<your username>/<your username>.github.io.git
```

### 挂载博客到Github Pages 

常用运行命令：

```bash
hexo cl
```

这个命令用于清除之前生成的静态文件。第一次没有生成，所以不用

后面只要使用了`hexo g`就一定要使用，因为`hexo g`不会把之前生成过的文件再生成一遍，在`hexo g`

```bash
hexo g
```



```bash
hexo d
```



这样，我们就把网站上传了，如果你想在本地预览，就运行如下命令：

```bash
hexo s
```

这些就是常用命令。

## 总结

我通过这样的方式建立了这样的一个个人空间。

我知道网上有很多这样的教程，但是这是我的个人经验，总体上还是能算原创的，再次感谢[Fomal大佬的教程](https://www.fomal.cc/posts/e593433d.html)。

下一篇，我讲讲关于博客的美化。886！