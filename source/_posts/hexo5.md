---
title: 自己动手写二维物理引擎（零）
date: 2024-09-28 12:06:08
top_img: /img/2024-09-28-12-03-32.png
cover: /img/2024-09-28-12-03-32.png
type: "post"
tags: 
  - 物理引擎
categories: 物理引擎
---

# 前言

## 为什么要写物理引擎

其实在很久以前就用过别人的物理引擎了，可是这些引擎的功能不满足我的需求，比如说不支持流体的运动，以及实现把一个物体切割成两部分等，还可以制造燃烧，爆炸等效果，别人的物理引擎都没有。所以我决定自己先用python写一款物理引擎，后面再写成JavaScript来实现网页上的效果。

本文的编号是零，意思是这是写物理引擎前的准备工作。

## 物理引擎的结构

组成物理引擎的是窗口（Window），力（Force），物体（Object）和效果（Effect）。

窗口就是一个物理场，它记录了很多物理常量（如重力加速度等）

同时，它也是物体的载体，可以控制物体是否能跑到窗口外面去。

力可以使物体运动。会考虑重力、引力、浮力等。

物体分为三种：刚体（solid）、流体（liquid）、和关节（joint）

> 刚体是指在运动中和受力作用后，形状和大小不变，而且内部各点的相对位置不变的物体。

刚体可以分为线段（segment）球体（其实就是一个圆，circle）和多边形（polygon）。

每个刚体都会考虑碰撞。

流体分为牛顿流体和非牛顿流体。

关节可以约束两个刚体，可以伸缩。

效果则包含劈裂、燃烧等

# 开始

## 新建文件

在同一个目录下新建`physics.py`和`demo.py`。`physics.py`使我们的物理引擎。`demo.py`则用来测试

## 代码

physics.py

```python
import pygame
import threading
from pygame.locals import *
import sys
pygame.init()
# pygame.draw.circle(window, (0, 0, 255),[300,300], 170, 0)
# pygame.draw.polygon(window, (255,0,0), [[300,300],[100,400],[100,300]])
# pygame.draw.line(window, (0, 0, 0), [100, 300], [500, 300], 5)
class Window():
    def __init__(self,width,height,caption):
        self.width=width
        self.height=height
        self.caption=caption
        self.surface=pygame.display.set_mode((self.width,self.height))
        self.bodies=[]
    def run(self):
        pygame.display.set_caption(self.caption)
        flag=True
        while flag:
            pygame.display.update()
            for event in pygame.event.get():
                if event.type == QUIT:
                    pygame.quit()
                    flag=False
class Object():
    def __init__(self):
        pass
class Solid(Object):
    def __init__(self):
        pass
class Liquid(Object):
    def __init__(self):
        pass
class Joint(Object):
    def __init__(self):
        pass
```

demo.py

```py
import physics
window1=physics.Window(1000,600,'123')
window1.run()
```

