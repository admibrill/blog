---
title: 自己动手写二维物理引擎（一）
date: 2024-10-20 21:33:20
top_img: /img/2024-09-28-12-03-32.png
cover: /img/2024-09-28-12-03-32.png
type: "post"
tags: 
  - 物理引擎
categories: 物理引擎
---

# 前言

这一篇，继续之前的想法，开始写物理引擎的代码。

由于**pygame不能同时打开多个窗口**，我决定换**Qt**来显示窗口。

# 开始

## 导入库

打开`physics.py`

```python
from PySide6 import * 
import math
```

- `PySide6`：这是Qt官方的PyQt6的库。这里我们用它来实现窗口以及内容的显示。
- `math`：Python数学库，用来处理一些高级的物理计算。

## 全局常量

```python
DYMANIC=0
STATIC=1
```

这些常量定义了物体的状态等信息，后学会有更多常量写在这里。

## Qt应用程序

```python
qapp=QtWidgets.QApplication([])
def run():
    qapp.exec()
```

之后在`demo.py`中运行`physics.run()`即可运行应用程序。

# 物理空间

编写一个`Space(QtWidegets.QWidget)`类来模拟一个物理空间，这将被显示为一个窗口。

## 实例化

```python
class Space(QtWidgets.QWidget):	
    def __init__(self,width,height,caption,g):
        super().__init__()
        self.width=width
        self.height=height
        self.caption=caption
        self.objects=[]
        self.g=g
        self.timer = QtCore.QTimer()
```

- `width`：窗口的宽度
- `height`：窗口的高度
- `caption`：窗口的标题
- `objects`：物理空间中所有物体列表
- `g`：重力加速度（常量）
- `timer`：使用qt计时器，用来定时运行一些函数

## 启动窗口

```python
# class Space(QtWidgets.QWidget):	
	def launch(self):
        self.resize(self.width,self.height)
        self.setWindowTitle(self.caption)
        self.timer.start(5)
        self.timer.timeout.connect(self.moveObjects)
        self.timer.timeout.connect(self.update)
        self.show()
```

该函数可以使物理空间窗口弹出并初始化。

## 移动与绘制

一个使窗口移动，并制造重力，另一个绘制。

```python
# class Space(QtWidgets.QWidget):
    def moveObjects(self):
        for i in self.objects:
            if i.stype==DYMANIC:
                gravity=Force(self.g,math.pi/2)
                i.forceCompound(gravity)
                i.move()
    def paintEvent(self,event):
        painter = QtGui.QPainter(self)
        for i in self.objects:
            if isinstance(i,Circle):
                painter.setPen(i.pen)
                painter.drawEllipse(i.cpoint,i.r,i.r)
                painter.drawLine(i.cpoint.x(),i.cpoint.y(),
                              i.cpoint.x()+i.r*math.cos(i.direction),i.cpoint.y()+i.r*math.sin(i.direction))
            elif isinstance(i,Segment):
                painter.setPen(i.pen)
                painter.drawLine(i.x1,i.y1,i.x2,i.y2)
```

# 力与物体对象

## 力

力是最简单的一个类，它是一个**向量**，只有两个成员：方向和大小。

特别注意，所有的角单位都用弧度（$\pi\space \text{rad}=180^{\circ}$）

```python
class Force():
    def __init__(self,size,angle):
        self.size=size
        self.angle=angle
```

## 物体对象

物体对象有很多种，如刚体、流体、关节

### 实例化

```python
class Object():
    def __init__(self,z,window,color,mass,angle):
        self.z=z
        self.window=window
        self.window.objects.append(self)
        self.color=QtGui.QColor()
        self.color.setRgb(color[0],color[1],color[2])
        self.mass=mass
        self.velocity=0
        self.angle=angle
```
- `z`：层坐标，`z`不同的两个元素不会发生碰撞。
- `window`：所在的窗口
- `color`：对象显示的颜色
- `mass`：对象的质量
- `velocity`：运动速度
- `angle`：运动方向

### 人为施力

![image-20241020192920919](https://image.qyadbr.top/file/97ff9678655c516d94095-c862322cf153358e6a.png)

可以看出，我们可以利用三角函数把物体当前的方向和速度分解为水平方向上的速度和垂直方向上的速度，对施加的力作同样的处理，将水平和垂直的分别相加再转换为新的速度和角度就可以实现人为施力。

于是又了下面的代码：

```python
# class Object():
	def forceCompound(self,add_force):
        cur_force_size=self.velocity
        cur_force_dx=cur_force_size*math.cos(self.angle)+add_force.size*math.cos(add_force.angle)/self.mass
        cur_force_dy=cur_force_size*math.sin(self.angle)+add_force.size*math.sin(add_force.angle)/self.mass
        self.velocity=math.sqrt(cur_force_dx**2+cur_force_dy**2)
        if cur_force_dx>=0:
            self.angle=math.atan(cur_force_dy/cur_force_dx)
        else:
            self.angle=math.atan(cur_force_dy/cur_force_dx)+math.pi
```

### 删除对象

把对象移除，不多赘述

```python
# class Object():
    def destroy(self):
        self.window.objects.remove(self)
        self.window=None
        del self
```

# 刚体

目前先实现圆、线段。之后会实现多边形。

```python
class Solid(Object):
    def __init__(self,z,window,color,stype,mass,angle,direction):
        super().__init__(z,window,color,mass,angle)
        self.stype=stype
        self.direction=direction
        self.pen = QtGui.QPen(self.color)
        self.pen.setWidth(2)
class Circle(Solid):
    def __init__(self,window,z,x,y,r,direction,stype,mass,angle=0,color=(0,0,255)):
        super().__init__(z,window,color,stype,mass,angle,direction)
        self.cpoint=QtCore.QPointF(x,y)
        self.r=r
    def move(self):
        self.cpoint.setX(self.cpoint.x()+self.velocity*math.cos(self.angle))
        self.cpoint.setY(self.cpoint.y()+self.velocity*math.sin(self.angle))
    def hit(self,c):
        pass
class Segment(Solid):
    def __init__(self,window,z,x1,y1,x2,y2,direction,stype,mass,angle=0,color=(0,0,255)):
        super().__init__(z,window,color,stype,mass,angle,direction)
        self.x1=x1
        self.y1=y1
        self.x2=x2
        self.y2=y2
    def move(self):
        self.x1+=self.velocity*math.cos(self.angle)
        self.y1+=self.velocity*math.sin(self.angle)
        self.x2+=self.velocity*math.cos(self.angle)
        self.y2+=self.velocity*math.sin(self.angle)
    def hit(self,c):
        pass
```

## `angle`与`direction`的区别

`angle`是指运动方向。而`direction`是只物体本身的朝向。

# 效果测试

`demo.py`

```python
import physics

window1=physics.Space(1000,600,'123',9.8)
window1.launch()
circle1=physics.Circle(window1,0,100,100,40,0,physics.DYMANIC,1000)
force1=physics.Force(1000,0)
circle1.forceCompound(force1)
segment1=physics.Segment(window1,0,50,550,950,550,0,physics.STATIC,1000)
physics.run()
```

![21-24-31](https://image.qyadbr.top/file/cb091da5c9e4ad39451f3-b40670a181745f9d8a.gif)

# 下期预告

下一期会实现多边形以及碰撞检测。886！

