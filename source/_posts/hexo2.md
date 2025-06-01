---
title: Hexo博客0氪建站记录（中）
date: 2024-08-25 15:57:55
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

上次，我们完成了Hexo框架的搭建。这次，我讲如何美化博客。

一打开博客页面，是一个单调的LandScape主题，没有首页，还有一篇hello world的英文文章。

想自己装饰那么多肯定不容易，于是我们就要去[hexo官网](https://hexo.io/zh-cn)来选择一个好看的主题来装饰。

主题就是一个模版，有很多选项，这就是主题的配置，可以自己来设置。

看了一些教程，也改了很多，有一小部分是自己改的。

我使用的[安知鱼](https://github.com/anzhiyu-c/hexo-theme-anzhiyu)主题，是一款可用度较高的主题，支持许多功能，推荐使用。

## 启动！

众所周知，静态网页是由三门语言编写的：

- HTML:超文本标记语言，有许多标签，有不同的功能，可以生成一个网页框架。
- CSS：层叠样式表，用于给框架加上华丽的样式。
- JS：java网页脚本。用于交互。

而博客魔改的重点就在CSS和JS上。

博客有很多魔改都是看的[Fomal大佬](www.fomal.cc)整理的教程，就把采纳了的和自己改的写在这里。

安知鱼主题的末尾处有一个`inject: `配置项，这里可以用于在网页头和尾引入外链css和js文件。

```yaml
inject: 
  head:
  	...
  bottom:
  	...
```

CSS文件一般放在head内，因为样式需要再预加载完成前渲染，这样就可以给让网页访问者更好的体验。

而JS一般放在bottom，因为JS时常需要获取HTML中的元素，如果在head就引入，执行，那么就有可能造成元素获取不到，返回了null，然这个js脚本执行中止。

## 魔改开始！！！

~~从这里开始，有部分内容都是链接~~

### 黑夜霓虹灯、星空和流行特效重构

[Hexo博客魔改教程总结(一) | Fomalhaut🥝](https://www.fomal.cc/posts/eec9786.html)

### 外挂标签的引入

[Tag Plugins Plus | Akilarの糖果屋](https://akilar.top/posts/615e2dec/)

### 鼠标样式

[Hexo博客魔改教程总结(二)| Fomalhaut🥝](https://www.fomal.cc/posts/5389e93f.html)

### 右下角工具按钮显示阅读进度

[返回顶部显示网页阅读进度 | Leonus](https://blog.leonus.cn/2022/percent.html)

### Vue+Element样式弹窗

[Hexo博客魔改教程总结(三) | Fomalhaut🥝](https://www.fomal.cc/posts/2d7ac914.html)

### 文章统计图

[Hexo 博客文章统计图 | Eurkon](https://blog.eurkon.com/post/1213ef82.html)

### 公告栏欢迎信息

[给博客添加腾讯地图定位并制作个性欢迎 | ichikaの小窝](https://ichika.cc/Article/beautiful_IPLocation/)

你以为公告栏欢迎信息这就结束了?NO!NO!NO!

由于[腾讯定位服务](https://lbs.qq.com/)实在太抠门，每天就让一次请求，于是我找来了[ip-api.com](ip-api.com)（主打一个0氪）

之前的~~`txmap.js`~~改成这样：感谢ichika提供的代码（因为换了API，响应结果的结构都变了，于是有很多地方要改，于是全粘进来）

（是不是应该换个文件名？）

```javascript
//get请求
$.ajax({
    type: 'get',
    url: 'http://ip-api.com/json/?fields=status,message,continent,continentCode,country,countryCode,region,regionName,city,district,zip,lat,lon,timezone,offset,currency,isp,org,as,asname,reverse,mobile,proxy,hosting,query&lang=zh-CN',
    data: {
        callback: 'jsonp',
    },
    dataType: 'jsonp',
    success: function (res) {
        ipLoacation = res;
    }
})
function getDistance(e1, n1, e2, n2) {
    const R = 6371
    const { sin, cos, asin, PI, hypot } = Math
    let getPoint = (e, n) => {
        e *= PI / 180
        n *= PI / 180
        return { x: cos(n) * cos(e), y: cos(n) * sin(e), z: sin(n) }
    }

    let a = getPoint(e1, n1)
    let b = getPoint(e2, n2)
    let c = hypot(a.x - b.x, a.y - b.y, a.z - b.z)
    let r = asin(c / 2) * 2 * R
    return Math.round(r*1000)/1000;
}

function showWelcome() {
    let dist = getDistance(114.337006, 30.525186, ipLoacation.lon, ipLoacation.lat); //这里换成自己的经纬度
    let pos = ipLoacation.country;
    let ip;
    let posdesc;
    //根据国家、省份、城市信息自定义欢迎语
    switch (ipLoacation.country) {
        case "日本":
            posdesc = "よろしく，一起去看樱花吗";
            break;
        case "美国":
            posdesc = "Let us live in peace!";
            break;
        case "英国":
            posdesc = "想同你一起夜乘伦敦眼";
            break;
        case "俄罗斯":
            posdesc = "干了这瓶伏特加！";
            break;
        case "法国":
            posdesc = "C'est La Vie";
            break;
        case "德国":
            posdesc = "Die Zeit verging im Fluge.";
            break;
        case "澳大利亚":
            posdesc = "一起去大堡礁吧！";
            break;
        case "加拿大":
            posdesc = "拾起一片枫叶赠予你";
            break;
        case "中国":
            pos = ipLoacation.country+ " " +ipLoacation.regionName + " " + ipLoacation.city;
            ip = ipLoacation.query;
            switch (ipLoacation.regionName) {
                case "北京":
                    posdesc = "北——京——欢迎你~~~";
                    break;
                case "天津":
                    posdesc = "讲段相声吧。";
                    break;
                case "河北省":
                    posdesc = "山势巍巍成壁垒，天下雄关。铁马金戈由此向，无限江山。";
                    break;
                case "山西省":
                    posdesc = "展开坐具长三尺，已占山河五百余。";
                    break;
                case "内蒙古自治区":
                    posdesc = "天苍苍，野茫茫，风吹草低见牛羊。";
                    break;
                case "辽宁省":
                    posdesc = "我想吃烤鸡架！";
                    break;
                case "吉林省":
                    posdesc = "状元阁就是东北烧烤之王。";
                    break;
                case "黑龙江省":
                    posdesc = "很喜欢哈尔滨大剧院。";
                    break;
                case "上海市":
                    posdesc = "众所周知，中国只有两个城市。";
                    break;
                case "江苏省":
                    switch (ipLoacation.city) {
                        case "南京":
                            posdesc = "这是我挺想去的城市啦。";
                            break;
                        case "苏州":
                            posdesc = "上有天堂，下有苏杭。";
                            break;
                        default:
                            posdesc = "散装是必须要散装的。";
                            break;
                    }
                    break;
                case "浙江省":
                    posdesc = "东风渐绿西湖柳，雁已还人未南归。";
                    break;
                case "河南省":
                    switch (ipLoacation.city) {
                        case "郑州":
                            posdesc = "豫州之域，天地之中。";
                            break;
                        case "南阳":
                            posdesc = "臣本布衣，躬耕于南阳。此南阳非彼南阳！";
                            break;
                        case "驻马店":
                            posdesc = "峰峰有奇石，石石挟仙气。嵖岈山的花很美哦！";
                            break;
                        case "开封":
                            posdesc = "刚正不阿包青天。";
                            break;
                        case "洛阳":
                            posdesc = "洛阳牡丹甲天下。";
                            break;
                        default:
                            posdesc = "可否带我品尝河南烩面啦？";
                            break;
                    }
                    break;
                case "安徽省":
                    posdesc = "蚌埠住了，芜湖起飞。";
                    break;
                case "福建省":
                    posdesc = "井邑白云间，岩城远带山。";
                    break;
                case "江西省":
                    posdesc = "落霞与孤鹜齐飞，秋水共长天一色。";
                    break;
                case "山东省":
                    posdesc = "遥望齐州九点烟，一泓海水杯中泻。";
                    break;
                case "湖北省":
                    posdesc = "来碗热干面！";
                    break;
                case "湖南省":
                    posdesc = "74751，长沙斯塔克。";
                    break;
                case "广东省":
                    posdesc = "老板来两斤福建人。";
                    break;
                case "广西壮族自治区":
                    posdesc = "桂林山水甲天下。";
                    break;
                case "海南省":
                    posdesc = "朝观日出逐白浪，夕看云起收霞光。";
                    break;
                case "四川省":
                    posdesc = "康康川妹子。";
                    break;
                case "贵州省":
                    posdesc = "茅台，学生，再塞200。";
                    break;
                case "云南省":
                    posdesc = "玉龙飞舞云缠绕，万仞冰川直耸天。";
                    break;
                case "西藏自治区":
                    posdesc = "躺在茫茫草原上，仰望蓝天。";
                    break;
                case "陕西省":
                    posdesc = "来份臊子面加馍。";
                    break;
                case "甘肃省":
                    posdesc = "羌笛何须怨杨柳，春风不度玉门关。";
                    break;
                case "青海省":
                    posdesc = "牛肉干和老酸奶都好好吃。";
                    break;
                case "宁夏回族自治区":
                    posdesc = "大漠孤烟直，长河落日圆。";
                    break;
                case "新疆维吾尔自治区":
                    posdesc = "驼铃古道丝绸路，胡马犹闻唐汉风。";
                    break;
                case "台湾省":
                    posdesc = "我在这头，大陆在那头。";
                    break;
                case "香港特别行政区":
                    posdesc = "永定贼有残留地鬼嚎，迎击光非岁玉。";
                    break;
                case "澳门特别行政区":
                    posdesc = "性感荷官，在线发牌。";
                    break;
                default:
                    posdesc = "带我去你的城市逛逛吧！";
                    break;
            }
            break;
        default:
            posdesc = "带我去你的国家逛逛吧。Show me around your country.";
            break;
    }

    //根据本地时间切换欢迎语
    let timeChange;
    let date = new Date();
    if (date.getHours() >= 5 && date.getHours() < 11) timeChange = "<span>上午好</span>，一日之计在于晨！";
    else if (date.getHours() >= 11 && date.getHours() < 13) timeChange = "<span>中午好</span>，该摸鱼吃午饭了。";
    else if (date.getHours() >= 13 && date.getHours() < 15) timeChange = "<span>下午好</span>，懒懒地睡个午觉吧！";
    else if (date.getHours() >= 15 && date.getHours() < 16) timeChange = "<span>三点几啦</span>，一起饮茶呀！";
    else if (date.getHours() >= 16 && date.getHours() < 19) timeChange = "<span>夕阳无限好！</span>";
    else if (date.getHours() >= 19 && date.getHours() < 24) timeChange = "<span>晚上好</span>，夜生活嗨起来！";
    else timeChange = "夜深了，早点休息，少熬夜。";

    try {
        //自定义文本和需要放的位置
        document.getElementById("welcome-info").innerHTML =
            `<b><center>🎉 欢迎信息 🎉</center>&emsp;&emsp;欢迎来自 <span style="color:var(--theme-color)">${pos}</span> 的小伙伴，${timeChange}您现在距离站长约 <span style="color:var(--theme-color)">${dist}</span> 公里，当前的IP地址为： <span style="color:var(--theme-color)">${ip}</span>， ${posdesc}</b>`;
    } catch (err) {
        console.log("Pjax无法获取#welcome-info元素🙄🙄🙄");
    }
}
window.onload = showWelcome;
// 如果使用了pjax在加上下面这行代码
document.addEventListener('pjax:complete', showWelcome);

```

### FPS检测（Ariasaka）

[博客魔改日记（3） | Ariasakaの小窝 (yaria.top)](https://blog.yaria.top/posts/670e47f/)

### 综合美化窗口

[综合美化模块教程 | Fomalhaut🥝](https://www.fomal.cc/posts/9ac969bb.html)

Fomal参考了Leonus的教程，但是~~还是Fomal的更全面~~，所以用了Fomal的

但是有一个问题，Fomal他使用了自定义图标，按他的导航栏来，安知鱼图标不适配，于是`nav.pug`下面的部分改成这样：

```pug
#menus
      !=partial('includes/header/menu_item', {}, {cache: true})
    #nav-right
      if theme.nav.travelling
        .nav-button.only-home#travellings_button(title='随机前往一个开往项目网站')
          a.site-page(onclick='anzhiyu.totraveling()', title='随机前往一个开往项目网站', href='javascript:void(0);', rel='external nofollow', data-pjax-state='external')
            i.anzhiyufont.anzhiyu-icon-train
      .nav-button#randomPost_button
        a.site-page(onclick='toRandomPost()', title='随机前往一个文章', href='javascript:void(0);')
          i.anzhiyufont.anzhiyu-icon-dice
      if (theme.algolia_search.enable || theme.local_search.enable || theme.docsearch.enable)
        div.nav-button#search-button
          a.site-page.social-icon.search(href='javascript:void(0);', title='搜索🔍' accesskey="s")
            i.anzhiyufont.anzhiyu-icon-magnifying-glass
            span=' '+_p('search.title')
          
      //- 美化设置开始
      div.nav-button
        a.meihua.faa-parent.animated-hover(onclick="toggleWinbox()" title="美化设置-自定义你的风格" id="meihua-button" )
          i.anzhiyufont.anzhiyu-icon-pencil
      //- 修改结束
      if theme.centerConsole.enable
        input#center-console(type="checkbox")
        label.widget(for="center-console" title=_p("中控台") onclick="anzhiyu.switchConsole();")
          i.left
          i.widget.center
          i.widget.right

      !=partial('includes/anzhiyu/console', {}, {cache:true})

      div.nav-button#nav-totop
        a.totopbtn(href='javascript:void(0);')
          i.anzhiyufont.anzhiyu-icon-arrow-up
          span#percent(onclick="anzhiyu.scrollToDest(0,500)") 0

      #toggle-menu
        a.site-page(href='javascript:void(0);' title="切换")
          i.anzhiyufont.anzhiyu-icon-bars
```

### 粒子连线+鼠标交互特效

首先，感谢[序炁](www.ordchaos.com)给的`canvas-nest-mouse.min.js`（我也不清楚原作者是谁，但是是他给我的）

把它外链引入就可以了，这是JS代码：

```javascript
!function () {
    function o(w, v, i) {
        return w.getAttribute(v) || i
    }
    function j(i) {
        return document.getElementsByTagName(i)
    } function l() {
        var i = j("script"), w = i.length, v = i[w - 1];
        return { l: w, z: o(v, "zIndex", -1), o: o(v, "opacity", 0.5), c: o(v, "color", "0,0,0"), n: o(v, "count", 500) } /*经研究，发现count是粒子的数量，不要调太多，不然会粒子泛滥*/
    }
    function k() {
        r = u.width = window.innerWidth || document.documentElement.clientWidth || document.body.clientWidth, n = u.height = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight
    }
    function b() {
        e.clearRect(0, 0, r, n);
        var w = [f].concat(t);
        var x, v, A, B, z, y;
        t.forEach(function (i) {
            i.x += i.xa, i.y += i.ya, i.xa *= i.x > r || i.x < 0 ? -1 : 1, i.ya *= i.y > n || i.y < 0 ? -1 : 1, e.fillRect(i.x - 0.5, i.y - 0.5, 1, 1);

            for (v = 0; v < w.length; v++) {
                x = w[v];
                if (i !== x && null !== x.x && null !== x.y) {
                    B = i.x - x.x, z = i.y - x.y, y = B * B + z * z;
                    y < x.max && (x === f && y >= x.max / 2 && (i.x -= 0.03 * B, i.y -= 0.03 * z), A = (x.max - y) / x.max, e.beginPath(), e.lineWidth = A / 2, e.strokeStyle = "rgba(" + s.c + "," + (A + 0.2) + ")", e.moveTo(i.x, i.y), e.lineTo(x.x, x.y), e.stroke())
                }
            } w.splice(w.indexOf(i), 1)

        }), m(b)
    } 
    var u = document.createElement("canvas"), s = l(), c = "c_n" + s.l, e = u.getContext("2d"), r, n, m = window.requestAnimationFrame || window.webkitRequestAnimationFrame || window.mozRequestAnimationFrame || window.oRequestAnimationFrame || window.msRequestAnimationFrame || function (i) { window.setTimeout(i, 1000 / 45) }, a = Math.random, f = { x: null, y: null, max: 20000 };
    u.id = c;
    u.className="lizicanvas";/*为了方便开关粒子特效，自己给粒子的canvas加了一个类*/
    u.style.cssText = "position:fixed;top:0;left:0;z-index:" + s.z + ";opacity:" + s.o;
    j("body")[0].appendChild(u);
    k(), window.onresize = k;
    window.onmousemove = function (i) { i = i || window.event, f.x = i.clientX, f.y = i.clientY }, window.onmouseout = function () { f.x = null, f.y = null };
    for (var t = [], p = 0;s.n > p;p++) {
        var h = a() * r, g = a() * n, q = 2 * a() - 1, d = 2 * a() - 1;
        t.push({ x: h, y: g, xa: q, ya: d, max: 6000 })
    } 
    setTimeout(function () { b() }, 100);
}();
```

## 总结

好了，写了这么多，全是链接，下期写评论等系统的部署经验，拜拜。



