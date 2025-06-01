---
title: Qt Troubleshoot（三）
date: 2025-1-20 10:21:09
top_img: /img/2024-12-30-20-37-23.png
cover: /img/2024-12-30-20-37-23.png
type: "post"
categories:
  - Qt Troubleshoot
tags:
  - Qt
---
# 1.`setWindowFlags(Qt.WindowType.WindowStayOnTopHint)`窗口隐藏

制作了一个可以自动显示的窗口类，实例化后自动显示，无需`window.show()`。但是，在`show()`之后又通过`setWindowFlags()`方法设置窗口置顶时：

```python
from PySide6.QtWidgets import *
from PySide6.QtCore import *
app=QApplication()
class MyWindow(QWidget):
    def __init__(self):
        super().__init__()
        self.show()
        self.setWindowFlags(Qt.WindowType.WindowStaysOnTopHint)
window=MyWindow()
app.exec()
```

这个窗口刚显示出来就隐藏了。
我想这也是为什么`Qt`的窗口不会自动显示的原因之一。但是如果一定要在构造函数里面自动显示，而且`setWindowFlags()`必须放在`show()`的后面的时候，就会出现这个问题。

当然，可以在设置完之后再显示一次。但是，这样窗口还是会隐藏，在用户看来就是窗口闪了一下。
## 解决

所以，以后在调用了`show()`之后就不要用`setWindowFlags()`来设置窗口置顶了。（经过测试，设置其他的还行，比如说设置无边框就没有这种问题）
改成用`window.windowHandle().setFlags()`，作用是相同的，但是窗口不会隐藏。

# 2.`window.hasFocus()`不能正确判断窗口焦点

其实，`window.hasFocus()`并不是不能正确判断窗口焦点，它只判断这个窗口内的这个元件是否有焦点。焦点也不是自动变化的，而是要通过`setFocus()`来转移焦点到当前原件上。当窗口有焦点（窗口被点击，被系统置于所有其他窗口的上方）时，并不会自动调用`setFocus()`。

在窗口的`mousePressEvent()`中调用`setFocus`不是一种正确的做法。因为这样确实让`hasFocus()`返回的值变为`True`了，但是这将影响到里面元件的焦点。窗口和里面的所有元件中只能有一个有焦点，与窗口与其他窗口的只有一个的焦点不是同一个。

所以，`QWidget`专门提供了一个函数来判断当前窗口和其他窗口中，只有一个窗口有的那个焦点，在不在窗口上。这个函数就是：
```python
window.isActiveWindow()
```

当然，对应地，`setFocus()`函数也是只对当前元件所在窗口内的，如果需要设置窗口的全局焦点，那么方法名字应改为：
```python
window.activateWindow()
```

# 3.`setWindowIcon()`不改变任务栏图标

开发应用程序时，常常需要把窗口的图标改成自己的图标。设置的方法很简单，就是`window.setWindowIcon()`。
所以写了一个设置图标的测试程序：
```python
from PySide6.QtWidgets import *
from PySide6.QtCore import *
from PySide6.QtGui import *
app=QApplication()
window=QWidget()
window.setWindowIcon(QPixmap('../icon.png'))
window.show()
app.exec()
```
运行效果如图：
![image](https://s1.imagehub.cc/images/2025/01/20/3a7403e181cf89c2e0a2040a591cf698.png)
可以看到，窗口左上角图标和任务栏图标悬停后显示的缩略图的左上角图标都被设置了。但是任务栏上的那个图标没有被设置。
这个是Windows系统的一个问题，Windows认为所有的Python程序都是同一个appid，所以所有打开的窗口都显示同意个图标，Python的图标。为了修改appid，我们只能利用`ctypes`库访问系统dll。
为了防止程序在其他操作系统报错无法执行，这段代码外面要加上一层`try...except`语句。
在导入库之后紧接着加入这段代码：
```python
try:
    from ctypes import windll
    windll.shell32.SetCurrentProcessExplicitAppUserModelID('<your appid>')
except ImportError:
    pass
```
把`<your appid>`替换为一个字符串，建议包含一些有关应用程序的信息，每条信息用`.`隔开。
最终效果：
![image](https://s1.imagehub.cc/images/2025/01/20/7a13b0e65efa7bd8b5936b830996df3f.png)