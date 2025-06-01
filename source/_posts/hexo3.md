---
title: Hexo博客0氪建站记录（下）
date: 2024-08-29 15:57:55
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

1. 我还没说过我域名是怎么来的

2. ~~建站不知道多少天了~~，我的朋友圈竟然还长这样：
![](https://source-admibrill.pages.dev/file/7bd9ae102aa38c6193185.png)
在看了清羽飞扬的[Friend-Circle-Lite:轻量友链朋友圈](https://blog.liushen.fun/posts/4dc716ec/)之后，手痒了，决定搞一个。

3. 还有，我还需要自建图床。于是使用[Telegraph Image](https://github.com/cf-pages/Telegraph-Image)做了一个。之前用github搞过一个，可是听别人说，github禁止把仓库当图床，轻则删库，重则封号，仓库大于1GB就要人工审核，所以就用了这种方法。


## 使用cloudflare Pages部署Github Pages的网站（？）

实在无法理解，之前那个`admibrill.github.io`蛮好，但过了几天就无法访问了（？）,但是我的一些朋友还能访问（？）,为了让自己能够访问，我把github上的项目仓库在cf上又部署了一次。

首先，打开cloudflare,找到`Workers和Pages`然后进入，点击创建，创建一个部署。

![](https://source-admibrill.pages.dev/file/06561d555b0e66e7ef27c.png)

![](https://source-admibrill.pages.dev/file/9e300d58129a590f871dc.png)

选择仓库，等程序跑完就可以了。

## 友链朋友圈

这一部分参考清羽飞扬的博文，~~清羽的里面有很多碎碎念~~，我精简一点。

### .json存储文件

这个文件用于记录朋友圈里会显示的友站。

我们的友链有很多，我们不能每次改了友链都要改这个json文件。于是清羽飞扬提供了生成.json文件的JavaScript：

在`[blogroot]`新建文件`link.js`，写入如下内容：（感谢代码）

```javascript
const YML = require('yamljs')
const fs = require('fs')

const blacklist = ["友站名称1", "友站名称2", "友站名称3"]; // 由于某种原因，不想订阅的列表

let friends = [],
    data_f = YML.parse(fs.readFileSync('source/_data/link.yml').toString().replace(/(?<=rss:)\s*\n/g, ' ""\n'));

data_f.forEach((entry, index) => {
    let lastIndex = 2;
    if (index < lastIndex) {
        const filteredLinkList = entry.link_list.filter(linkItem => !blacklist.includes(linkItem.name));
        friends = friends.concat(filteredLinkList);
    }
});

// 根据规定的格式构建 JSON 数据
const friendData = {
    friends: friends.map(item => {
        return [item.name, item.link, item.avatar];
    })
};

// 将 JSON 对象转换为字符串
const friendJSON = JSON.stringify(friendData, null, 2);

// 写入 friend.json 文件
fs.writeFileSync('./source/friend.json', friendJSON);
//在控制台提示
console.log('friend.json 文件已生成。');
```

然后我们每次更新博客，就得多加一条，保证没有友链漏掉：

```bash
node link.js;..... #...为其他指令
```

**现在就更新一次，确保friend.json上传到了网站里！**

### 后端部署

首先，前往清羽的仓库[willow-god/Friend-Circle-Lite: 🐱一个精简版，无后端，且仅利用github action运行的精简版友链朋友圈程序，兼容fc的json格式信息，同时支持推送友圈更新，支持他人订阅个人站点并在更新时发送邮箱推送](https://github.com/willow-god/Friend-Circle-Lite)把仓库Fork复制到自己的账号下：

![](https://source-admibrill.pages.dev/file/0457a14794145a11dcbb8.png)

然后到自己的账号里的这个仓库，找到`conf.yml`

![](https://source-admibrill.pages.dev/file/16d2402b0ef56f0b5ce19.png)

修改配置：

```yaml
# 爬虫相关配置
# 解释：使用request实现友链文章爬取，并放置到根目录的all.json下
#   enable:             是否启用爬虫
#   json_url:           请填写对应格式json的地址，仅支持网络地址
#   article_count:      请填写每个博客需要获取的最大文章数量
#   marge_result:       是否合并多个json文件，若为true则会合并指定网络地址和本地地址的json文件
#     enable:           是否启用合并功能，该功能提供与自部署的友链合并功能，可以解决服务器部分国外网站无法访问的问题
#     marge_json_path:  请填写网络地址的json文件，用于合并，不带空格！！！
spider_settings:
  enable: true
  json_url: "https://blog.qyadbr.top/friend.json" ########一定要改，不然是我的，改成"自己的网站/friend.json"
  article_count: 15
  merge_result:
    enable: false
    merge_json_url: "https://fc.liushen.fun"

# 邮箱推送功能配置，暂未实现，等待后续开发
# 解释：每天为指定邮箱推送所有友链文章的更新，仅能指定一个
#   enable:             是否启用邮箱推送功能
#   to_email:           收件人邮箱地址
#   subject:            邮件主题
#   body_template:      邮件正文的 HTML 模板文件
email_push:
  enable: false   ########### 我不需要，就没有启用
  to_email: recipient@example.com
  subject: "今天的 RSS 订阅更新"
  body_template: "rss_template.html"

# 邮箱issue订阅功能配置
# 解释：向在issue中提取的所有邮箱推送您网站中的更新，添加邮箱和删除邮箱均通过添加issue对应格式实现
#   enable:             是否启用邮箱推送功能
#   github_username:    GitHub 用户名，用于构建issue api地址
#   github_repo:        GitHub 仓库名，用于构建issue api地址
#   your_blog_url:      你的博客地址
rss_subscribe:
  enable:  false ############ 我不需要，就没有启用
  github_username: willow-god
  github_repo: Friend-Circle-Lite
  your_blog_url: https://blog.qyadbr.top/
  email_template: "./rss_subscribe/email_template.html"

# SMTP 配置
# 解释：使用其中的相关配置实现上面两种功能，若无推送要求可以不配置，请将以上两个配置置为false
#   email:              发件人邮箱地址
#   server：            SMTP 服务器地址
#   port：              SMTP 端口号
#   use_tls：           是否使用 tls 加密
smtp:
  email: 3162475700@qq.com
  server: smtp.qq.com
  port: 587
  use_tls: true
```

![](https://source-admibrill.pages.dev/file/8901ef3ed5250a9a41ada.png)

### 第一次运行测试

按图示操作，运行workflow：

![](https://source-admibrill.pages.dev/file/8babe6f6beed7e37405da.png)

然后我们点开一条记录，查看`Check RSS feeds`

![](https://source-admibrill.pages.dev/file/3b9b57ffb17f3a68af552.png)

有这样的输出即为正常。

### 使用cloudflare托管后端

按照前面cloudflare部署的方法，部署一个后端地址。这个地址将会被我们当做后端。

### 前端部署

在`Blogroot/source/fcircle/index.md`中添加如下内容（其实就是加html）：

```html
<div id="friend-circle-lite-root"></div>
<script>
    if (typeof UserConfig === 'undefined') {
        var UserConfig = {
            // 填写你的fc Lite地址
            private_api_url: 'https://fc.liushen.fun/',
            // 点击加载更多时，一次最多加载几篇文章，默认20
            page_turning_number: 20,
            // 头像加载失败时，默认头像地址
            error_img: 'https://pic.imgdb.cn/item/6695daa4d9c307b7e953ee3d.jpg',
        }
    }
</script>
<link rel="stylesheet" href="https://fastly.jsdelivr.net/gh/willow-god/Friend-Circle-Lite/main/fclite.min.css">
<script src="https://fastly.jsdelivr.net/gh/willow-god/Friend-Circle-Lite/main/fclite.min.js"></script>
```

ok,朋友圈功能到此完成。

## Telegraph Image图床

在写博客的时候，常常需要图片~~（比如说刚才那个部分就弄了n次图片）~~。

为了方便自己，自己建了个图床。

### 部署图床

首先还是fork仓库（不需要图了吧），[cf-pages/Telegraph-Image: Image Hosting solution, Flickr/imgur alternative, make it easy for users to share their images. Using Cloudflare Pages and Telegraph. (github.com)](https://github.com/cf-pages/Telegraph-Image)。

然后把把仓库部署的cloudflare上（不需要图了吧）。

### 使用PicGo方便Typora上传图片

然后下载安装PicGo，进入插件，安装插件telegraph-image-uploader

进入图床设置

![image-20240829221408459](https://source-admibrill.pages.dev/file/74f1f956e5cb8d939db46.png)

设置URL为你的图床地址，就可以使用PicGo上传啦！

接着使用Markdown编辑器Typora:

![](https://source-admibrill.pages.dev/file/6ac87f110fed6c040f5e5.png)

现在截图复制粘贴到Typora里就可以上传了。

### 使用Python来删除Typora创建的临时图片文件

但是有个问题：我们粘进Typora的文件，会先放到`C:\Users\<username>\AppData\Roaming\Typora\typora-user-images`这个文件夹里，久而久之，文件会越来越多，Typora却不自己删除。于是我决定用Python来删除这些文件。

#### 代码

在任意位置新建一个`delete.py`，写入如下代码：

```python
# 导入库，放心，都是Python标准库，不用pip
import os
import time
# 编辑配置
username='乐乐'   #这是我电脑用户名，需要改成自己的
path='C:\\Users\\'+username+'\\AppData\\Roaming\\Typora\\typora-user-images\\'  #检查的目录
checktime=5 #每次检查新文件和删除文件的间隔时间
deltime=50  #每个文件存在大于多少秒就要删除，足以等待图片上传完
# 启动
while True: 
    listd=os.listdir(path)
    for i in listd:
        if time.time()-os.path.getctime(path+i)>deltime:
            os.remove(path+i)
            print('removed '+path+i)
    time.sleep(checktime)
```

## 总结

到此hexo建站教程就完结了，886！