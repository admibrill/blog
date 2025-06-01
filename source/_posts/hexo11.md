---
title: Qt Troubleshoot（二）
date: 2025-01-18 10:47:34
top_img: /img/2024-12-30-20-37-23.png
cover: /img/2024-12-30-20-37-23.png
type: "post"
categories:
  - Qt Troubleshoot
tags:
  - Qt
---
最近在研究Qt的过程中，又积攒了一些问题，下面是解决方案。

# 1.`QColor.lighter()`与`QColor.darker()`

在Qml中，使用`Qt.lighter()`和`Qt.darker()`来让颜色变得更亮或更暗。括号里填的是相比原来颜色亮了或暗了多少倍，值在1左右。
但是，如果这样使用到`QtWidgets`中，`lighter`的效果会很暗，`darker`的效果会很亮，反而达到了相反的效果。

这是因为，在Python程序文件中，括号内的值要乘以100，QColor是按原来的百分之多少来把颜色变亮或变暗的。

# 2.Qt同时继承两个类，窗口闪退

有一次，我继承`QFrame`自己定义了一个`Frame`类，这个类带有自动显示、鼠标事件检测等功能。
但是，`QFrame`是没有显示文字的功能的，所以我就尝试，制作一个`Label`类，同时继承（即载括号里写两个父类）可不可以呢？

```python
class Label(Frame,QLabel):
	def __init__(self,parent=None,show=True,autoAdjust=True):
		global widgetCount
		super().__init__(parent)
		
```

当然，窗口刚显示就直接消失了，这个可能就是Qt的一个问题，所以建议以后不要同时继承两个Qt类（Frame是继承QFrame的，所以也算）。

# 3.`QWidget.mouseMoveEvent()`点击鼠标再移动才触发事件

这也是Qt的一个bug。
解决这个问题的方法很简单，在自己定义的类的构造函数（`__init__()`）中加入这么一行：
```python
# class YourWidget(QWidget):
	# def __init__(self):
		# 其他内容
		self.setMouseTracking(True)
```

这样，鼠标直接在窗口中移动，`mouseMoveEvent()`也能触发了。

# 4.如何设置窗口即无边框又圆角

制作组件的时候，常常需要自定义一些样式。比如我需要无边框圆角窗口。
于是就有了这样的代码：
```python
from PySide6.QtWidgets import *
from PySide6.QtCore import *
app=QApplication()
class MyWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowFlags(Qt.WindowType.FramelessWindowHint)
        self.setStyleSheet('border-radius: 4')
window=MyWindow()
window.show()
app.exec()
```

但是，效果并不好，没有圆角：

![image](https://s1.imagehub.cc/images/2025/01/18/0a44771ae8c21cc9aebca90f95f14e48.png)

这是因为Windows窗口不能设置圆角。但是可以把窗口透明，然后自己做一个背景控件再设置背景控件的圆角。因为背景是控件，所以是可以设置圆角样式的。

最终代码：
```python
from PySide6.QtWidgets import *
from PySide6.QtCore import *
app=QApplication()
class MyWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowFlags(Qt.WindowType.FramelessWindowHint)
        self.setAttribute(Qt.WidgetAttribute.WA_TranslucentBackground)
        self.setStyleSheet('background-color:transparent')
        self.background1=QWidget(self)
        self.background1.resize(self.size())
        self.background1.setStyleSheet('background-color:#FFFFFF;border-radius: 10')
window=MyWindow()
window.show()
app.exec()
```

效果：
![image](https://s1.imagehub.cc/images/2025/01/18/0a74223a771be8410bcf10f1e1d7b898.png)