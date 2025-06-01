---
title: Qt Troubleshoot （一）
date: 2024-12-30 20:38:31
top_img: /img/2024-12-30-20-37-23.png
cover: /img/2024-12-30-20-37-23.png
type: "post"
categories:
  - Qt Troubleshoot
tags:
  - Qt
---
# 建立这个分类的目的

自从开始用Qt来编写应用程序之后，我发现应用的开发变得方便了许多。当然，图中难免遇到问题。我会在这个分类不断记录我遇到的问题以及我的解决过程。

# 1.`QLabel`自动换行

## 描述

我在使用`QtWidgets.QLabel`时，常常需要一个自动换行功能。查阅Qt官网，发现`QLabel`有个`setWordWrap()`方法。但是，有时即使设置了为`True`,它任然没有换行。

```python
from PySide6.QtWidgets import *
app=QApplication([])
window=QWidget()
label=QLabel('LongWordTestLongWordTestLongWordTestLongWordTestLongWordTest',window)
label.setWordWrap(True)
window.resize(500,500)
window.show()
app.exec()
```

## 解决

这是因为`QLabel`只会在有空格的地方换行。所以，纯数字和纯字母都会认为是只有一个单词，所以就不会换行了。
当然，纯汉字也是会正常换行的。

# 2.自定义类继承`QWidget`不能设置样式

## 描述

一次，我利用`QWidget`来制作一个自己的窗口类，让它的对象实例化后自动显示。这个实现也很简单，这是一个简化的例子：
```python
from PySide6.QtWidgets import *
class CustomWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.show()
app=QApplication([])
window=CustomWindow()
window.resize(500,500)
app.exec()
```

后来，我有想设置一个初始的样式，把它的背景色设成绿色，在`__init__(self)`里面加了一行：
```python
self.setStyleSheet('background-color: green;')
```

结果竟然是，`QWidget`还是那个`QWidget`，背景还是那个初始颜色，根本不是绿色
## 解决

经过了研究，我发现可能是有时`QWidget`的子类是不会自动设样式表的（在一个复杂的项目中我遇到了此问题，当我尝试简化时，这个问题消失了，但是这个问题一定存在）。
有两种解决方法。
### 第一种：改变父类

`QWidget`可能有问题，但是`QFrame`不会有问题。改成继承`QFrame`就可以了。
```python
class CustomWindow(QFrame):
```
### 第二种：设置控件属性

这个问题的根本就是`QWidget`的背景样式没有启用哦。可以设置控件样式（`QWidget.setAttribute()`）来启用。
```python
window.setAttribute(Qt.WidgetAttribute.WA_StyledBackground,True)
```
# 3.`QWidget.setStyleSheet()`覆盖之前的样式表

## 描述

刚才说了，`setStyleSheet()`方法可以设设置`QWidget`的样式表。然而，每次设置之后都会覆盖原有的。
## 解决

解决的方法不是很复杂，可以自己定义一个类，设置一个`styleSheets`属性（和`QWidget`原有的`styleSheet()`方法区分开）
在自己写一个`addStyleSheet()`方法来维护它。
```python
class LingmoFrame(QFrame):
	def __init__(self,parent=None,show=True):
		super().__init__(parent)
		self.timer=QTimer()
		self.timer.timeout.connect(self.updateEvent)
		self.timer.timeout.connect(self.__update__)
		self.timer.start(timerDelay)
		if show:
			self.show()
		self.styleSheets={}
	def __update__(self):
		self.update()
		styleSheetString=''
		for i in self.styleSheets:
			styleSheetString+=i+': '+self.styleSheets[i]+';'
		self.setStyleSheet(styleSheetString)
	def updateEvent(self):
		pass
	def addStyleSheet(self,name,style):
		self.styleSheets[name]=str(style)
```

这个方法还贴心地将`style`转换成字符串类型，这样每次数值就不需要强转了。
# 结束

篇幅有限，最近的问题也没收集多少，所以这篇文章到这里就结束了。
2024年还有两天（算今天）就要过去，2025年就要来了。祝大家元旦快乐！Bye！