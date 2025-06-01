---
title: Moonlark（Nonebot2+Python）命令式聊天机器人插件开发记录
date: 2025-5-25 21:58:06
top_img: /img/2025-05-25-22-07-47.jpg
cover: /img/2025-05-25-22-07-47.jpg
type: "post"
categories:
  - Python
tags:
  - 机器人
  - Nonebot2
---
# 往事

一直~~沉迷~~在lingmo-webbrowser的开发中无法自拔。
但是，由于Qt WebEngine 一个致命的问题——如果想要支持html5视频元素，就得重新编译qt
这个问题很难解决
后来，~~由于种种原因~~，webbrowser最终archive掉了，于是我有了时间去看别的。
在接触[IT Craft Development Team]([IT Craft Development Team](https://github.com/ITCraftDevelopmentTeam))后，我开始想去给由Python+Nonebot2写出来的[Moonlark](https://github.com/Moonlark-Dev/Moonlark)做一些贡献。

# 参考的资料
{% link 'Nonebot' 'https://nonebot.dev/' 'https://nonebot.dev/' %}
{% link 'Moonlark开发文档' 'https://moonlark-docs.itcdt.top/' 'https://moonlark-docs.itcdt.top/' %}

# 搭建开发环境
(基于moonlark文档修改)

1. 确保您的 Python 版本 `>= 3.11`。
2. 安装包管理工具 [Poetry](https://python-poetry.org/docs/)（建议安装 `2.1.0` 及以上的版本）。
3. 克隆仓库并安装依赖，可以使用以下指令：

```sh
git clone https://github.com/Moonlark-Dev/Moonlark
cd Moonlark
pip install pipx
pipx install nb-cli
pipx install poetry
poetry install
```

## 配置[​](https://moonlark-docs.itcdt.top/quick-start/create-develop-environment.html#%E9%85%8D%E7%BD%AE)

将`.env.template` 复制为 `.env` 并在 `.env` 文件中填写相关环境变量。
（不测试对应的功能不用填）

## 搭建测试环境[​](https://moonlark-docs.itcdt.top/quick-start/create-develop-environment.html#%E6%90%AD%E5%BB%BA%E6%B5%8B%E8%AF%95%E7%8E%AF%E5%A2%83)

为了测试 Moonlark 的代码，您需要运行一个 OneBot 实现并使其连接到 `ws://localhost:8080/onebot/v11/ws`（默认状态下），我们提供了以下两个参考方案：

- 使用 [NapCat](https://napneko.github.io/guide/start-install): 需要有一个闲置的 QQ 号供 NapCat 登录。同时，您可能需要承担该账号被 TX 冻结的风险。
- 使用 [Matcha](https://github.com/A-kirami/matcha): 目前已知无法显示 Moonlark 发出的图片。

NapCat确实好用，但是在qq上面，所有用了两天就被tx封。
Matcha虽然不能直接显示图片，但是在测试时可以通过nonebot的logger和with open(path,"wb")将bytes类型的图片保存到本地。
# 开发

一个moonlark插件基本上要有这些内容
假设插件的名字叫`nonebot_plugin_[name]`。
## `src/plugins/nonebot_plugin_[name]`

首先建立插件文件夹

```sh
nb plugin create nonebot_plugin_[name]
```

修改Moonlark目录下的`pyproject.toml`
```toml
...
[tool.nonebot]
...
plugins=[
	...,
	"nonebot_plugin_[name]",#加入这一行，确保插件会被加载
	...
]
```
## `src/plugins/nonebot_plugin_[name]/__init__.py`

这个文件包含了一些基本信息

```python
from nonebot import require
from nonebot.plugin import PluginMetadata

from .config import Config

__plugin_meta__ = PluginMetadata(
    name="nonebot_plugin_[name]",
    description="描述",
    usage="",
    config=Config,
)

require("nonebot_plugin_[othername]")


from . import __main__

```

## `src/lang/[lang]/[name].yaml`

这是moonlark用于本地化（翻译）的文件
目前支持`zh_hans`（简体中文，大陆）、`zh_tw`（繁体中文，台湾）、`en_us`（英语，美国）三种语言。
每个语言文件夹下都要创建一个对应的yaml，才能识别成功，并且每个对应的yaml中的键都要相同
例如，在`zh_hans/[name].yaml`中有一个键`key`,那么在`zh_tw`和`en_us`下的`[name].yaml`都要有`key`这个键。

键的命名满足以下规则：
- 不以数字开头；
- 由26个大写、26个小写、10个阿拉伯数字或者减号（`-`）组成
- 子键要缩进

## `src/templates/[name].html.jinja`
有时候，机器人需要发送图片。这时需要一个图片模版来图片
引用Moonlark开发文档：

> ## 模板编写
> Render 读取的模板储存在 `src/templates` 中，后缀为 `*.jinja`。
> 
> 一个模板的格式如下：
> 
> html
> 
> ```
> {% extends base %}
> {% block body %}
> {{ content }}
> {% endblock body %}
> ```
> 
> ### 基模板
> 
> html
> 
> ```
> {% extends base %}
> ```
> 
> 这里使用了一个模板变量 `base` 作为基模板的名称，这个变量在渲染时会自动填充为对应主题的基模板。
> 
> ### 内容块
> 
> #### `header`
> 
> 拓展页面页面的头部。
> 
> #### `body`
> 
> 卡片主体内容。
> 
> #### `card`
> 
> 卡片。
> 
> ### 保留变量
> 
> 这些变量会在渲染时被自动填充，请避免使用这些模板变量名。
> 
> - `base`: 主题基模板的相对路径。
> - `main_title`: 页面主标题。
> - `footer`: 页面页脚（版权信息）。
> 
> ## 主题[​](https://moonlark-docs.itcdt.top/plugins/render.html#%E4%B8%BB%E9%A2%98)
> 
> Render 支持主题，主题基模板储存在 `src/templates/base` 中，主题列表储存在 `src/plugins/nonebot_plugin_render/themes.json` 中。
> 
> ### 基模板[​](https://moonlark-docs.itcdt.top/plugins/render.html#%E5%9F%BA%E6%A8%A1%E6%9D%BF-1)
> 
> 一个主题的基模板需要定义以下变量和内容块：
> 
> #### 模板变量[​](https://moonlark-docs.itcdt.top/plugins/render.html#%E6%A8%A1%E6%9D%BF%E5%8F%98%E9%87%8F)
> 
> - `main_title`: 页面主标题。
> - `footer`: 页面页脚（版权信息）。
> 
> #### 内容块[​](https://moonlark-docs.itcdt.top/plugins/render.html#%E5%86%85%E5%AE%B9%E5%9D%97-1)
> 
> - `header`: 页面拓展头部。
> - `body`: 卡片主体内容。
> - `card`: 卡片。
> 
> TIP
> 
> 一般来说，`card` 会覆盖 `body` 块。
> 
> ### 主题配置[​](https://moonlark-docs.itcdt.top/plugins/render.html#%E4%B8%BB%E9%A2%98%E9%85%8D%E7%BD%AE)
> 
> 主题配置是一个 JSON 文件，格式为 `"主题ID": "主题模板相对于 src/templates 的路径"`。
> 
> ## 本地化[​](https://moonlark-docs.itcdt.top/plugins/render.html#%E6%9C%AC%E5%9C%B0%E5%8C%96)
> 
> 所有向用户展示的文本都要被本地化。
> 
> ## 本地文件引用[​](https://moonlark-docs.itcdt.top/plugins/render.html#%E6%9C%AC%E5%9C%B0%E6%96%87%E4%BB%B6%E5%BC%95%E7%94%A8)
> 
> 可以使用 `{% include "xx" %}` 块或 `src=xxx` 引入本地文件。 使用相对引入时，基路径为 `src/templates`。


## `src/plugins/nonebot_plugin_[name]/__main__.py`

一般nonebot的插件主要部分都是在这里写的
里面的内容由插件的功能而定，由开发者~~自由发挥~~
但是有需要注意的几点：

### 创建命令
用户通过命令来使用nonebot2的插件。
Moonlark提供了一系列相关工具。（注意：这些都是nonebot2本身没有的）
首先在`__init__.py`里加上
```python
require("nonebot_plugin_alconna")
require("nonebot_plugin_larklang")
require("nonebot_plugin_larkuser")
```
然后就可以在`__main__.py`里导入工具
```python
from nonebot_plugin_alconna import Alconna, Args, on_alconna, Subcommand
from nonebot_plugin_larklang.__main__ import LangHelper
from nonebot_plugin_larkuser.utils.matcher import patch_matcher
```
接着就可以创建插件的命令
nonebot本身有`on_command(`)方法，但是不方便获取参数，Moonlark使用Alconna让获取命令的参数更方便。
```python
[command] = on_alconna(
    Alconna(
        "[command]",
        Subcommand("[subcommand]", Args["[argument name]", [argument type],[default value]])        
    )
)
```
期中`[command]`指命令名称
`[subcommand]`指子命令名称
`[argument name]`指参数名称
`[argument type]`参数类型
`[default value]`默认值（可选）
```python
lang = LangHelper() #本地化
patch_matcher(sudoku) #启动指令配对器
```

### 事件处理
Nonebot是用到了异步编程，所以很多函数都用了`async`和`await`关键字
我总结了一下使用规则：
- `def`,`for`,`with`等关键字前可以使用`async`，让其异步执行；
- 调用使用了`async`定义的函数，需要使用`await`;
- 函数体中调用其他函数使用了`await`的函数（函数调用了其他异步函数），这个函数定义时要使用`async`（这个函数也一定是异步函数）

主命令的事件处理
```python

#记得在__init__.py加入这个
require("nonebot_plugin_larkutils")

#记得在__main__.py开头导入这个
from nonebot_plugin_larkutils import get_user_id

@[command].assign("$main")
async def _(args : [type], user_id: str = get_user_id()): #参数不止一个，建议指定类型
	...#干你想干的事情
```

子命令就是把`$main`换成对应的子命令名称即可

### 向用户发送消息

事件处理的一个重要步骤就是让机器人发送消息。

#### Alconna发送消息
```python
[command].send([message])#发送消息
[command].finish([message])#发送消息（终止事件处理）
```
`[command]`就是`on_alconna()`返回的`AlconnaMatcher()`对象。


#### LangHelper发送消息
Moonlark开发工具，用于本地化。

```python
#lang就是刚才lang=LangHelper()创建的对象
lang.send("key",user_id,args)
lang.finish("key",user_id,args)
```

其中`key`是刚才本地化时添加的组件
访问子键时要先把父键写在前面，父键和子键用点号（`.`）连接。

#### 渲染图片模版并发送图片

##### 导入依赖

**在`__init__.py`中
```python
require("nonebot_plugin_htmlrender")
```

**在`__main__.py`中**
```python
from nonebot_plugin_render.render import render_template
from nonebot_plugin_render.cache import creator
```
##### 定义渲染模版函数
```python
@creator("[command].html.jinja")
async def render(content: dict, user_id: str = get_user_id()):
    return await render_template("[command].html.jinja", "", user_id, content, {}, True)
```

使用
```python
async def _(user_id: str = get_user_id()):
    ...
    image = await render([content], user_id)
    ...    
```

其中`content`是记录内容的字典，要提供jinja对应的信息。

##### 发送图片
```python
await UniMessage().image(raw=image).send(reply_to=True)
```
`reply_to`设置为`True`时，机器人会回复用户的消息。
# 测试

## 启动Nonebot

在moonlark文件夹下运行命令，启动机器人

```sh
poetry run nb run
```

## 使用Matcha测试

不推荐Napcat测试，容易被封。
先设置角色（角色唯一标识必须是纯数字）
![image](https://s1.imagehub.cc/images/2025/05/26/b321716b601f0ae56efb49be8bc42126.png)
在启动Nonebot后向机器人发送消息就可以收到消息了