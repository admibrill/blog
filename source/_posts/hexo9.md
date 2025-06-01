---
title: 使用Qt实现一个简易时钟应用
date: 2024-12-8 9:39:37
top_img: /img/clock.png
cover: /img/clock.png
type: "post"
categories:
  - Qt
tags:
  - 应用程序
  - 简易
---
# 前言

我使用Qt制作了一个时钟应用，现在来分享一下实现的过程。

这个小项目包括了四个部分：
- 时钟（显示当前时间）
- 秒表（正向计时器，支持记录时间）
- 倒计时
- 闹钟

# 各种准备工作

## 导入

制作这个应用，我使用了一大堆`QtWidgets`组件。同时，还有至关重要的`time`模块。

```python
from PySide6.QtWidgets import (QApplication,QWidget,QPushButton,QGroupBox,QLabel,
                                QTableWidget,QTableWidgetItem,QProgressBar,QScrollArea,
				                QDialog,QGridLayout,QVBoxLayout,QHBoxLayout,QSpinBox,QLineEdit,QComboBox,QSystemTrayIcon,QCheckBox)
from PySide6.QtGui import QIcon,QFont
from PySide6.QtCore import QObject,QTimer,Qt,QSize
import time
```

然后，建立一个`QApplication`，定义一些常量。

```python
qapp=QApplication([])
icon=QIcon('../media/images/1.png') #窗口图标
widgetslist=[] #每个模式对应一个窗格
musicUrlList=['../media/audio/1.wav'] #闹铃音乐
musicNameList=['音乐1'] #音乐名称

menubuttonnamelist=['时钟','秒表','倒计时','闹钟']
menubuttonlist=[]
mode=0 #当前的模式

#下面这段永远放在最后
qapp.exec()
```
## 建立窗口

建立一个类，继承`QtWidgets.QWidget`。

```python
class Window(QWidget):
    def __init__(self,width,height,title,icon):
        super().__init__()
        self.setWindowTitle(QObject.tr(title))
        self.resize(width,height)
        self.setWindowIcon(icon)
```

实例化窗口对象。

```python
mainWindow=Window(1000,600,'时钟',icon)
mainWindow.show()
```
## 功能窗格与切换按钮

我在窗口左侧做了四个按钮，分别对应四种功能。点击按钮就可以切换窗格的显示状态。

```python
#定义按钮类
class MenuButton(QPushButton):
    def __init__(self,parent,num):
        super().__init__(parent)
        self.num=num
        self.clicked.connect(self.toggleMode)
    def toggleMode(self):
        global mode
        widgetslist[mode].hide()
        widgetslist[self.num].show()
        mode=self.num
#显示
for i in range(len(menubuttonnamelist)):
    button=MenuButton(mainWindow,i)
    button.setText(menubuttonnamelist[i])
    button.move(10,10+i*30)
    button.show()
    menubuttonlist.append(button)
    group=QGroupBox(parent=mainWindow,title=menubuttonnamelist[i])
    group.resize(890,580)
    group.move(100,10)
    widgetslist.append(group)
```

## 窗格尺寸自适应

当缩放窗口时，窗格的大小不会改变，因此我们要自动设置窗格的尺寸。

在`Qt`中，`QWidget`被缩放大小就会自动执行`resizeEvent`函数，重写其内容即可。

```python
# class Window(QWidget):
    def resizeEvent(self,event):
        size=self.size()
        for i in widgetslist:
            i.resize(size.width()-110,size.height()-20)
```
# 时间显示

这是整个项目最最简单的一部分。
## 组件

```python
nowtime=time.strftime('%H:%M:%S')
labelTime=QLabel(widgetslist[0])
font=QFont('Arian',100,10)
labelTime.setFont(font)
labelTime.setText(nowtime)
```
## 更新显示

除了刷新时间，还设置了窗口自适应

```python
def update():
    size1_1=widgetslist[0].size()
    nowtime=time.strftime('%H:%M:%S')
    labelTime.setText(nowtime)
    labelTime.move(size1_1.width()/2-size2.width()/2,size1_1.height()/2-size2.height()/2)

timer=QTimer()
timer.start(0.05)
timer.timeout.connect(update)
```
## 效果

![image](https://s1.imagehub.cc/images/2024/12/03/5eb951eea3aace8e6aee72c6a5ba299b.png)

# 正向计时器

这是第二个功能，需要实现一个可以开始、暂停的计时器，计时状态下可以记录，非计时状态下可以清零。
## 创建组件

实现它，需要一个文本显示，三个按钮，以及一个表格。

```python
recorder=QLabel(widgetslist[1])
recorder.setText('00:00:00.000')
recorder.setFont(font)
startStopButton=QPushButton('开始',widgetslist[1])
recordButton=QPushButton('记录',widgetslist[1])
recordButton.setDisabled(True)
clearButton=QPushButton('清零',widgetslist[1])
clearButton.setDisabled(True)
recordTable=QTableWidget(0,3,widgetslist[1])
recordTable.setHorizontalHeaderLabels(['序次','间隔','总计'])
recordTable.verticalHeader().setVisible(False)
recordTable.setHorizontalScrollBarPolicy(Qt.ScrollBarAlwaysOff)
recordTable.setVerticalScrollBarPolicy(Qt.ScrollBarAlwaysOff)
```

## 变量

```python
startTimeStamp=0
lastEndStamp=0
lastRecordStamp=0
running=False
timeDelta=0
```

 - `startTimeStamp`：计时开始的时间戳
 - `lastEndStamp`：上次暂停时的时间戳
 - `lastRecordStamp`：上次记录时的时间戳，用于计算记录间隔
 - `running`：是否正在计时
 - `timeDelta`：暂停后，将产生时间差，在计算当前时间是要减去这个时间差
## 获取与格式化时间戳

变量已经齐全，下一步就是获取当前的时间。获取时间可以使用下面简单的公式：
$$
res=now-startTimeStamp-timeDelta
$$
这个公式要使用多次，所以封装成函数:

```python
def getTime():
    global startTimeStamp,timeDelta
    return time.time()-startTimeStamp-timeDelta 
```

获取到时间之后，将时间戳直接展示给用户肯定不现实，因此将其转换成时间字符串。同时，要传入一个参数，表示取整还是精确到小数点后三位。（后续倒计时和闹钟都不取后三位。）

```python
def formatTime(data,round=False):
    return '{:0>2d}'.format(int(data//3600))+':'+'{:0>2d}'.format((int(data)%3600)//60)+':'+'{:0>2d}'.format(int(data)%60)+('.'+'{:0>3d}'.format(int((data%1)*1000)) if not round else '')
```

## 计时控制

刚刚我创建了三个按钮组件，现在在要对应地写着三个按钮被点击是执行的函数。
### 开始和停止

这个函数可以控制`running`变量，以及做出对应的操作：
- 设置按钮位置；
- 改变`timeDelta`
- 设置按钮

```python
def startStop():
    global running,startTimeStamp,lastEndStamp,timeDelta,lastRecordStamp
    if running:
        lastEndStamp=time.time()
        startStopButton.setText('开始')
    else:
        if startTimeStamp==0:
            startTimeStamp=lastEndStamp=time.time()
        timeDelta+=time.time()-lastEndStamp
        startStopButton.setText('停止')
    recordButton.setDisabled(running)
    running=not running
    clearButton.setDisabled(running)
```
### 记录

记录按钮被按下时，就在表格中插入一行，第一列是次数，第二列是间隔，第三列是总计时间。最后，还要设置这次记录的时间戳，以计算这一次到下一次记录的时间间隔。

```python
def writeRecord():
    global lastRecordStamp
    recordTable.insertRow(0)
    recordTable.setItem(0,0,QTableWidgetItem(str(recordTable.rowCount())))
    if recordTable.rowCount()>1:
        recordTable.setItem(0,1,QTableWidgetItem(formatTime(getTime()-lastRecordStamp)))
    recordTable.setItem(0,2,QTableWidgetItem(formatTime(getTime())))
    lastRecordStamp=getTime()
```

### 清除记录

清除，要把各种变量都初始化，设置时间文本的文字也为初始的。因为表格控件没有提供清空函数，所以只能逐行删除直至表格为空为止。

```python
def clearRecord():
    global startTimeStamp,lastEndStamp,timeDelta,recorder
    startTimeStamp=lastEndStamp=timeDelta=0
    recorder.setText('00:00:00.000')
    clearButton.setDisabled(True)
    while(recordTable.rowCount()>0):
        recordTable.removeRow(0)
```

### 连接按钮与函数

使用`按钮控件.clicked.connect(函数名)`就可以在按钮被点击时调用函数。

```python
startStopButton.clicked.connect(startStop)  
recordButton.clicked.connect(writeRecord)     
clearButton.clicked.connect(clearRecord)
```

## 更新显示

窗口的自适应需要一些计算。

```python
# def update():
    global startTimeStamp,timeDelta
    # size1_1=widgetslist[0].size()
    # nowtime=time.strftime('%H:%M:%S')
    # labelTime.setText(nowtime)
    # labelTime.move(size1_1.width()/2-size2.width()/2,size1_1.height()/2-size2.height()/2)
    
    size1_2=widgetslist[1].size()
    size2=labelTime.size()
    size3=recorder.size()
    size4=startStopButton.size()
    size5=recordButton.size()
    size6=clearButton.size()
    recorder.move(size1_2.width()/2-size3.width()/2,10)
    startStopButton.move(size1_2.width()/2-size5.width()/2-10-size4.width(),size1_2.height()-10-size4.height())
    recordButton.move(size1_2.width()/2-size5.width()/2,size1_2.height()-10-size5.height())
    clearButton.move(size1_2.width()/2+size5.width()/2+10,size1_2.height()-10-size6.height())
    recordTable.resize(size1_2.width()-200,size1_2.height()-320)
    size7=recordTable.size()
    recordTable.move(size1_2.width()/2-size7.width()/2,220)
    recordTable.setColumnWidth(0,40)
    recordTable.setColumnWidth(1,size7.width()/2-20)
    recordTable.setColumnWidth(2,size7.width()/2-20)
    if running:
        stampDelta=getTime()
        rec=formatTime(stampDelta)
        recorder.setText(rec)
```
## 效果 

![image](https://s1.imagehub.cc/images/2024/12/08/d5680b89d31cdccd2edf6412960c29e6.png)

# 倒计时

倒计时不像正向计时那么简单，需要支持一些倒计时同时计时。这就需要建立一个倒计时类来完成这个任务。

## 创建组件

```python
newCountDown=QPushButton('新建',widgetslist[2])
deleteCountDown=QPushButton('编辑',widgetslist[2])
countDownScrollArea=QScrollArea(widgetslist[2])
countDownScrollArea.move(10,30)
countDownScrollArea.setHorizontalScrollBarPolicy(Qt.ScrollBarAlwaysOff)
countDownScrollAreaWidget=QWidget()
countDownScrollArea.setWidget(countDownScrollAreaWidget)
countDownList=[]
```

还是一些按钮，但是多了一个滚动区，这样可以滚动显示倒计时控件。

## 创建倒计时类

我将在`QtWidgets.QGroupBox`的基础上制作这个倒计时组件。
### 实例化

```python
class CountDown(QGroupBox):
    def __init__(self,parent,timelength,title,music):
        super().__init__(parent)
        self.timelength=timelength
        self.titlestr=title
        self.music=music
        self.restTime=0
        self.startTimeStamp=0
        self.lastEndStamp=0
        self.timeDelta=0
        self.running=False
```

### 启动

实例化方法内不能穿件组件，所以单独建一个启动函数，在对象实例化之后使用。
按钮绑定的函数后续都会定义。

```python
# class CountDown(QGroupBox):
    def launch(self):
        self.resize(400,300)
        self.restTime=self.timelength
        self.timelabel=QLabel(formatTime(self.restTime),self)
        font=QFont('Arian',40,10)
        self.timelabel.setFont(font)
        self.progressbar=QProgressBar(self)
        self.progressbar.setTextVisible(False)
        self.progressbar.setRange(0,10000)
        self.startStopButton=QPushButton('开始',self)
        self.startStopButton.clicked.connect(self.startStop)
        self.clearButton=QPushButton('还原',self)
        self.clearButton.clicked.connect(self.reset)
        self.clearButton.setDisabled(True)
        self.editButton=None
        self.deleteButton=None
        self.mode=True
```

### 其他函数

包括：
- 切换模式（开始与清除/编辑与删除）
- 开始停止
- 更新显示
- 清除
- 编辑
- 倒计时结束任务栏通知
- 删除

```python
# class CountDown(QGroupBox)
	def toggleMode(self):
        if self.mode:
            self.startStopButton.hide()
            self.clearButton.hide()
            self.editButton=QPushButton('编辑',self)
            self.editButton.clicked.connect(self.edit)
            self.editButton.show()
            self.deleteButton=QPushButton('删除',self)
            self.deleteButton.clicked.connect(self.deleteSelf)
            self.deleteButton.show()
        else:
            self.startStopButton.show()
            self.clearButton.show()
            self.editButton.hide()
            self.editButton=None
            self.deleteButton.hide()
            self.deleteButton=None
        self.mode=not self.mode
    def updating(self):
        self.setTitle(self.titlestr)
        selfsize=self.size()
        buttonsize=self.startStopButton.size()
        progressbarsize=self.progressbar.size()
        if abs(self.timelength-max(0,time.time()-self.startTimeStamp-self.timeDelta))<0.0005:
            self.showMessage()
            self.startStop()
            self.timelabel.setText('00:00:00:000')
        self.restTime=max(0,self.timelength-max(0,time.time()-self.startTimeStamp-self.timeDelta))
        if self.running:
            self.timelabel.setText(formatTime(self.restTime))
            self.progressbar.setValue(self.restTime/self.timelength*10000)
            self.progressbar.update()
        labelsize=self.timelabel.size()
        self.timelabel.move(selfsize.width()/2-labelsize.width()/2,selfsize.height()/2-labelsize.height()/2)
        self.progressbar.resize(selfsize.width()-20,20)
        self.progressbar.move(10,selfsize.height()-10-buttonsize.height()-10-progressbarsize.height())
        if self.mode:
            self.startStopButton.move(selfsize.width()/2-5-buttonsize.width(),selfsize.height()-10-buttonsize.height())
            self.clearButton.move(selfsize.width()/2+5,selfsize.height()-10-buttonsize.height())
        else:
            self.editButton.move(selfsize.width()/2-5-buttonsize.width(),selfsize.height()-10-buttonsize.height())
            self.deleteButton.move(selfsize.width()/2+5,selfsize.height()-10-buttonsize.height())
    def startStop(self):
        if self.running:
            self.lastEndStamp=time.time()
            self.startStopButton.setText('开始')
        else:
            if self.startTimeStamp==0:
                self.startTimeStamp=self.lastEndStamp=time.time()
            self.timeDelta+=time.time()-self.lastEndStamp
            self.startStopButton.setText('停止')
        self.running=not self.running
        self.clearButton.setDisabled(self.running)
    def reset(self):
        self.startTimeStamp=self.lastEndStamp=self.timeDelta=0
        self.timelabel.setText(formatTime(self.timelength))
        self.clearButton.setDisabled(True)
    def edit(self):
        editor=CountDownEdit(self.timelength,self.titlestr,self.music)
        editor.editData()
        self.timelength=editor.length
        self.restTime=self.timelength
        self.titlestr=editor.titlestr
        self.music=editor.music
    def showMessage(self):
        self.trayicon=QSystemTrayIcon()
        self.trayicon.setIcon(icon)
        self.trayicon.show()
        self.trayicon.showMessage('计时器到了','倒计时'+self.titlestr+'计时已完成。',icon)
    def deleteSelf(self):
        countDownList.remove(self)
        self.deleteLater()
        del self
```

## 倒计时编辑类

刚刚的`edit()`函数里，就有一个`CountDownEdit`。当用户编辑一个倒计时时，就要打开一个倒计时编辑窗口，因此，我单独再建一个类，打开一个`QtWidgets.QDialog`。

```python
class CountDownEdit(QDialog):
    def __init__(self,length,title,music):
        super().__init__()
        self.confirmed=False
        self.length=length
        self.titlestr=title
        self.music=music
    def editData(self):
        self.setWindowTitle('倒计时编辑')
        self.vlayout=QVBoxLayout()
        self.gridlayout=QGridLayout()
        self.hlayout=QHBoxLayout()
        self.gridlayout.setSpacing(10)
        self.label1=QLabel('时长')
        self.label2=QLabel('标题')
        self.label3=QLabel('音乐')
        self.content1=QHBoxLayout()
        self.content1_1=QSpinBox()
        self.content1_1.setMinimum(0)
        self.content1_1.setMaximum(99)
        self.content1_1.setValue(self.length//3600)
        self.content1_2=QLabel(':')
        self.content1_3=QSpinBox()
        self.content1_3.setMinimum(0)
        self.content1_3.setMaximum(59)
        self.content1_3.setValue((self.length%3600)//60)
        self.content1_4=QLabel(':')
        self.content1_5=QSpinBox()
        self.content1_5.setMinimum(0)
        self.content1_5.setMaximum(59)
        self.content1_5.setValue(self.length%60)
        self.content1.addWidget(self.content1_1)
        self.content1.addWidget(self.content1_2)
        self.content1.addWidget(self.content1_3)
        self.content1.addWidget(self.content1_4)
        self.content1.addWidget(self.content1_5)
        self.content2=QLineEdit()
        self.content2.setText(self.titlestr)
        self.content3=QComboBox()
        self.content3.addItems(musicNameList)
        self.content3.setCurrentIndex(self.music)
        self.gridlayout.addWidget(self.label1,0,0)
        self.gridlayout.addWidget(self.label2,1,0)
        self.gridlayout.addWidget(self.label3,2,0)
        self.gridlayout.addLayout(self.content1,0,1)
        self.gridlayout.addWidget(self.content2,1,1)
        self.gridlayout.addWidget(self.content3,2,1)
        self.confirm=QPushButton('确定')
        self.cancel=QPushButton('取消')
        self.confirm.clicked.connect(self.confirming)
        self.cancel.clicked.connect(self.close)
        self.hlayout.addWidget(self.confirm)
        self.hlayout.addWidget(self.cancel)
        self.vlayout.addLayout(self.gridlayout)
        self.vlayout.addLayout(self.hlayout)
        self.setLayout(self.vlayout)
        self.show()
        self.exec()
    def confirming(self):
        self.confirmed=True
        self.length=self.content1_1.value()*3600+self.content1_3.value()*60+self.content1_5.value()
        self.titlestr=self.content2.text()
        self.music=self.content3.currentIndex()
        self.close()
```

## 按钮绑定函数

接着，给刚才两个按钮创建并绑定函数。

```python
def onNewCountDown():
    global countDownList
    countDownEdit=CountDownEdit(0,'未命名',0)
    countDownEdit.editData()
    if countDownEdit.confirmed:
        countDown=CountDown(countDownScrollAreaWidget,countDownEdit.length,countDownEdit.titlestr,countDownEdit.music)
        countDown.launch()
        countDown.show()
        countDownList.append(countDown)
def onDeleteCountDown():
    global countDownList
    for i in countDownList:
        i.toggleMode()
newCountDown.clicked.connect(onNewCountDown)
deleteCountDown.clicked.connect(onDeleteCountDown)
```

## 更新显示

没错，又要更新

```python
# def update():
	size1_3=widgetslist[2].size()
    size8=newCountDown.size()
    size9=deleteCountDown.size()
    newCountDown.move(size1_3.width()/2-5-size8.width(),size1_3.height()-10-size8.height())
    deleteCountDown.move(size1_3.width()/2+5,size1_3.height()-10-size9.height())
    countDownScrollArea.resize(size1_3.width()-20,size1_3.height()-50-size8.height())
    size10=countDownScrollArea.size()
    size11=QSize(400,300)
    numPerRow=(size10.width()-size11.width())//(size11.width()+10)+1
    widthPerRow=numPerRow*size11.width()+(numPerRow-1)*10
    countDownScrollAreaWidget.resize(size10.width(),10+((len(countDownList))//numPerRow+1)*(10+size11.height()))
    for i in range(len(countDownList)):
        countDownList[i].move(size10.width()/2-widthPerRow/2+(i%numPerRow)*(10+size11.width()),10+(i//numPerRow)*(10+size11.height()))
        countDownList[i].updating()
```

## 文件读写

这个时钟应用还是支持保存和读取计时器的。实现就是文件读写。
### 读入

这个是程序每次打开就执行的。

```python
# deleteCountDown.clicked.connect(onDeleteCountDown)
with open('../data/countdown.txt','r',encoding='utf-8') as f:
    for i in f.readlines():
        timelength,title,music=i.split(':/:')
        countDown=CountDown(countDownScrollAreaWidget,int(timelength),title,int(music))
        countDown.launch()
        countDown.show()
        countDownList.append(countDown)

# def update():
```
### 保存

这个就是程序每次关闭执行的。
让我们回到程序一开始：

```python
# class Window(QWidget):
	def closeEvent(self,event):
        global countDownList,alarmList
        with open('../data/countdown.txt','w',encoding='utf-8') as f:
            for i in countDownList:
                f.write(str(i.timelength)+':/:'+i.titlestr+':/:'+str(i.music)+'\n')
```

## 效果

![image](https://s1.imagehub.cc/images/2024/12/08/62420788b13c81e6b3003003d2816649.png)
# 闹钟

闹钟的程序，就是把倒计时复制了一遍改的，所以直接给个代码
## 主要代码

```python
newAlarm=QPushButton('新建',widgetslist[3])
deleteAlarm=QPushButton('编辑',widgetslist[3])
alarmScrollArea=QScrollArea(widgetslist[3])
alarmScrollArea.move(10,30)
alarmScrollArea.setHorizontalScrollBarPolicy(Qt.ScrollBarAlwaysOff)
alarmScrollAreaWidget=QWidget()
alarmScrollArea.setWidget(alarmScrollAreaWidget)
alarmList=[]
class Alarm(QGroupBox):
    def __init__(self,parent,timelength,title,music,repeat,description,running):
        super().__init__(parent)
        self.timelength=timelength
        self.titlestr=title
        self.music=music
        self.running=running
        self.repeat=repeat
        self.description=description
    def launch(self):
        self.resize(400,300)
        self.restTime=self.timelength
        self.timelabel=QLabel(formatTime(self.restTime,round=True),self)
        font=QFont('Arian',40,10)
        self.timelabel.setFont(font)
        self.descriptionLabel=QLabel(self)
        self.descriptionLabel.setWordWrap(True)
        self.descriptionLabel.setAlignment(Qt.AlignmentFlag.AlignTop|Qt.AlignmentFlag.AlignLeft)
        self.repetitionLabel=QLabel(self)
        self.startStopButton=QPushButton('关闭' if self.running else '开启',self)
        self.startStopButton.clicked.connect(self.startStop)
        self.editButton=None
        self.deleteButton=None
        self.mode=True
    def toggleMode(self):
        if self.mode:
            self.startStopButton.hide()
            self.editButton=QPushButton('编辑',self)
            self.editButton.clicked.connect(self.edit)
            self.editButton.show()
            self.deleteButton=QPushButton('删除',self)
            self.deleteButton.clicked.connect(self.deleteSelf)
            self.deleteButton.show()
        else:
            self.startStopButton.show()
            self.editButton.hide()
            self.editButton=None
            self.deleteButton.hide()
            self.deleteButton=None
        self.mode=not self.mode
    def updating(self):
        hours,minutes,seconds=list(map(int,time.strftime('%H %M %S').split()))
        if abs(self.timelength-(hours*3600+minutes*60+seconds))<0.0005 and self.running:
            self.showMessage()
            self.startStop()
        self.setTitle(self.titlestr)
        selfsize=self.size()
        buttonsize=self.startStopButton.size()
        self.timelabel.setText(formatTime(self.timelength,round=True))
        labelsize=self.timelabel.size()
        self.timelabel.move(selfsize.width()/2-labelsize.width()/2,30)

        self.descriptionLabel.setText(self.description)
        self.descriptionLabel.setFixedSize(selfsize.width()-10,200)
        labelsize=self.descriptionLabel.size()
        self.descriptionLabel.move(selfsize.width()/2-labelsize.width()/2,150)
        text=['星期一','星期二','星期三','星期四','星期五','星期六','星期日']
        list1=[]
        for i in range(7):
            if self.repeat[i]:
                list1.append(text[i])
        self.repetitionLabel.setText('，'.join(list1))
        self.repetitionLabel.setFixedSize(selfsize.width()-10,20)
        labelsize=self.repetitionLabel.size()
        self.repetitionLabel.move(selfsize.width()/2-labelsize.width()/2,100)
        if self.mode:
            self.startStopButton.move(selfsize.width()/2-buttonsize.width()/2,selfsize.height()-10-buttonsize.height())
        else:
            self.editButton.move(selfsize.width()/2-5-buttonsize.width(),selfsize.height()-10-buttonsize.height())
            self.deleteButton.move(selfsize.width()/2+5,selfsize.height()-10-buttonsize.height())
    def startStop(self):
        if self.running:
            self.startStopButton.setText('开启')
        else:
            self.startStopButton.setText('关闭')
        self.running=not self.running
    def edit(self):
        editor=AlarmEdit(self.timelength,self.titlestr,self.music,self.repeat,self.description,self.running)
        editor.editData()
        self.timelength=editor.length
        self.restTime=self.timelength
        self.titlestr=editor.titlestr
        self.music=editor.music
        self.repeat=editor.repeat
        self.description=editor.description
        self.running=editor.running
    def showMessage(self):
        self.trayicon=QSystemTrayIcon()
        self.trayicon.setIcon(icon)
        self.trayicon.show()
        self.trayicon.showMessage('闹钟'+formatTime(self.timelength,round=True),self.description,icon)
    def deleteSelf(self):
        alarmList.remove(self)
        self.deleteLater()
        del self
class AlarmEdit(QDialog):
    def __init__(self,length,title,music,repeat,description,running):
        super().__init__()
        self.confirmed=False
        self.length=length
        self.titlestr=title
        self.music=music
        self.repeat=repeat
        self.description=description
        self.running=running
    def editData(self):
        self.setWindowTitle('闹钟编辑')
        self.vlayout=QVBoxLayout()
        self.gridlayout=QGridLayout()
        self.hlayout=QHBoxLayout()
        self.gridlayout.setSpacing(10)
        self.label1=QLabel('时间')
        self.label2=QLabel('标题')
        self.label3=QLabel('音乐')
        self.label4=QLabel('描述')
        self.label5=QLabel('重复')
        self.content1=QHBoxLayout()
        self.content1_1=QSpinBox()
        self.content1_1.setMinimum(0)
        self.content1_1.setMaximum(23)
        self.content1_1.setValue(self.length//3600)
        self.content1_2=QLabel(':')
        self.content1_3=QSpinBox()
        self.content1_3.setMinimum(0)
        self.content1_3.setMaximum(59)
        self.content1_3.setValue((self.length%3600)//60)
        self.content1_4=QLabel(':')
        self.content1_5=QSpinBox()
        self.content1_5.setMinimum(0)
        self.content1_5.setMaximum(59)
        self.content1_5.setValue(self.length%60)
        self.content1.addWidget(self.content1_1)
        self.content1.addWidget(self.content1_2)
        self.content1.addWidget(self.content1_3)
        self.content1.addWidget(self.content1_4)
        self.content1.addWidget(self.content1_5)
        self.content2=QLineEdit()
        self.content2.setText(self.titlestr)
        self.content3=QComboBox()
        self.content3.addItems(musicNameList)
        self.content3.setCurrentIndex(self.music)
        self.content4=QLineEdit()
        self.content4.setText(self.description)
        self.content5=QVBoxLayout()
        text=['星期一','星期二','星期三','星期四','星期五','星期六','星期日']
        self.content5s=[QCheckBox(text=i) for i in text]
        for i in self.content5s:
            self.content5.addWidget(i)
            p=self.content5s.index(i)
            i.setChecked(self.repeat[p])
        self.gridlayout.addWidget(self.label1,0,0)
        self.gridlayout.addWidget(self.label2,1,0)
        self.gridlayout.addWidget(self.label3,2,0)
        self.gridlayout.addWidget(self.label4,3,0)
        self.gridlayout.addWidget(self.label5,4,0)
        self.gridlayout.addLayout(self.content1,0,1)
        self.gridlayout.addWidget(self.content2,1,1)
        self.gridlayout.addWidget(self.content3,2,1)
        self.gridlayout.addWidget(self.content4,3,1)
        self.gridlayout.addLayout(self.content5,4,1)
        self.confirm=QPushButton('确定')
        self.cancel=QPushButton('取消')
        self.confirm.clicked.connect(self.confirming)
        self.cancel.clicked.connect(self.close)
        self.hlayout.addWidget(self.confirm)
        self.hlayout.addWidget(self.cancel)
        self.vlayout.addLayout(self.gridlayout)
        self.vlayout.addLayout(self.hlayout)
        self.setLayout(self.vlayout)
        self.show()
        self.exec()
    def setRepeat(self,p):
        self.repeat[p]=not self.repeat[p]
    def confirming(self):
        self.confirmed=True
        self.length=self.content1_1.value()*3600+self.content1_3.value()*60+self.content1_5.value()
        self.titlestr=self.content2.text()
        self.music=self.content3.currentIndex()
        self.description=self.content4.text()
        self.repeat=[i.isChecked() for i in self.content5s]
        self.close()
def onNewAlarm():
    global alarmList
    alarmEdit=AlarmEdit(0,'未命名',0,[False,False,False,False,False,False,False],'',True)
    alarmEdit.editData()
    if alarmEdit.confirmed:
        alarm=Alarm(alarmScrollAreaWidget,alarmEdit.length,alarmEdit.titlestr,alarmEdit.music,alarmEdit.repeat,alarmEdit.description,alarmEdit.running)
        alarm.launch()
        alarm.show()
        alarmList.append(alarm)
def onDeleteAlarm():
    global alarmList
    for i in alarmList:
        i.toggleMode()
newAlarm.clicked.connect(onNewAlarm)
deleteAlarm.clicked.connect(onDeleteAlarm)
with open('../data/alarm.txt','r',encoding='utf-8') as f:
    for i in f.readlines():
        timelength,title,music,repeat1,repeat2,repeat3,repeat4,repeat5,repeat6,repeat7,description,running1=i.split(':/:')
        alarm=Alarm(alarmScrollAreaWidget,int(timelength),title,int(music),list(map(bool,map(int,[repeat1,repeat2,repeat3,repeat4,repeat5,repeat6,repeat7]))),description,bool(int(running1)))
        alarm.launch()
        alarm.show()
        alarmList.append(alarm)
```

## 其他

别忘了还需要更新显示，关闭时的文件写入

```python
# update():
    size8=newAlarm.size()
    size9=deleteAlarm.size()
    newAlarm.move(size1_3.width()/2-5-size8.width(),size1_3.height()-10-size8.height())
    deleteAlarm.move(size1_3.width()/2+5,size1_3.height()-10-size9.height())
    alarmScrollArea.resize(size1_3.width()-20,size1_3.height()-50-size8.height())
    size10=alarmScrollArea.size()
    size11=QSize(400,300)
    numPerRow=(size10.width()-size11.width())//(size11.width()+10)+1
    widthPerRow=numPerRow*size11.width()+(numPerRow-1)*10
    alarmScrollAreaWidget.resize(size10.width(),10+((len(alarmList))//numPerRow+1)*(10+size11.height()))
    for i in range(len(alarmList)):
        alarmList[i].move(size10.width()/2-widthPerRow/2+(i%numPerRow)*(10+size11.width()),10+(i//numPerRow)*(10+size11.height()))
        alarmList[i].updating()
```

```python
# class Window(QWidget):
	# def closeEvent(self,event):
		with open('../data/alarm.txt','w',encoding='utf-8') as f:
            for i in alarmList:
                f.write(str(i.timelength)+':/:'+i.titlestr+':/:'+str(i.music)+':/:'+':/:'.join(map(str,map(int,i.repeat)))+':/:'+i.description+':/:'+str(int(i.running))+'\n')
```

## 效果

![image](https://s1.imagehub.cc/images/2024/12/08/8d9a81c4e9e72270e372cba66f1ba9ba.png)
# 结语

这个项目，虽然简单，但是代码有亿点点多。所以写这篇文章也写了很多。希望这个项目可以对学习Qt的小伙伴起到一些引导性作用。
其实，写这么长真的很不容易。
拜拜！See you next time!