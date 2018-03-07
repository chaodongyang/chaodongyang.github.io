---
layout: post
title: pyqt4 旧式信号与槽详解
categories: pyqt4
description: pyqt4信号与槽详解
keywords: pyqt4, pyqt4信号与槽详解
---
GUI 的程序开发人员并非需要甚至根本不需要知道所有的控件实现的底层细节，我们只需要知道当按钮按下时能够适当的相应即可。基于这一原因，Qt 和 pyqt 提供了两种通信机制：低级事件处理机制和高级事件处理机制，前者与其他 GUI 库提供的功能类似，或者被称之为 “信号与槽”。

QT 的一个关键特性是它使用信号和槽来进行对象之间的通讯。当一个组件发出一个信号时，一个可用的插槽应应该做出相应。如果一个信号连接到一个插槽，那么当信号被发射时，该插槽应该被调用。如果信号没有连接，则不会发生任何事情。

## 信号与槽机制的特点

- 信号可能连接到很多插槽
- 信号也可能连接到另外一个信号
- 信号参数可以是任何 python 类型
- 一个插槽可能连接到许多信号
- 连接可以是直接的 (同步) 也可是是排队的 (异步)
- 连接可以跨线程
- 连接可能中断

## 绑定和未绑定信号
信号（特别是非绑定信号）是作为 QObject 子类的类的属性。一个绑定信号具有connect()，disconnect()和emit()实现相关联的功能的方法。要截取一个信号必须把它连接到槽上。

## 连接信号与槽的语法形式

```python
QtCore.QObject.connect(a, QtCore.SIGNAL('QtSig()'), pyFunction)
QtCore.QObject.connect(a, QtCore.SIGNAL('QtSig()'), pyClass.pyMethod)
QtCore.QObject.connect(a, QtCore.SIGNAL('QtSig()'), b, QtCore.SLOT('QtSlot()'))
QtCore.QObject.connect(a, QtCore.SIGNAL('PySig()'), b, QtCore.SLOT('QtSlot()'))
QtCore.QObject.connect(a, QtCore.SIGNAL('PySig'), pyFunction)
```
以上语法可以理解为：连接 a 对象的信号到槽函数或者某个 b 对象的某个函数。
## 发射信号与槽的语法
```python
a.emit(QtCore.SIGNAL('clicked()'))
a.emit(QtCore.SIGNAL('pySig'), "Hello", "World")
```
发射一个信号，也可以带参数。

## 内置的信号与槽
大部分的窗口控件都提前预置了一些槽，所以很多时候可以直接把预置的信号和预置的槽相连接，无需做任何事情就可以得到想要的行为效果。

下面我们看两个简单的窗口部件 Dial 和 SpinBox。这两个窗口部件都有 valueChange() 信号，当这个信号触发时就会带有新值。这两个窗口部件也都有 setValue() 槽，带有整数型参数。因此可以将两个部件的信号和槽连接起来，无论用户改变哪一个窗口部件，都会让另一个值做出改变。

```python
# -*- coding: utf-8 -*-

# Form implementation generated from reading ui file 'D:\pyqtSingAndSloft\SingSloftDialog.ui'
#
# Created by: PyQt4 UI code generator 4.11.4
#
# WARNING! All changes made in this file will be lost!

from PyQt4 import QtCore, QtGui

try:
    _fromUtf8 = QtCore.QString.fromUtf8
except AttributeError:
    def _fromUtf8(s):
        return s

try:
    _encoding = QtGui.QApplication.UnicodeUTF8
    def _translate(context, text, disambig):
        return QtGui.QApplication.translate(context, text, disambig, _encoding)
except AttributeError:
    def _translate(context, text, disambig):
        return QtGui.QApplication.translate(context, text, disambig)

class Ui_Dialog(object):
    def setupUi(self, Dialog):
        Dialog.setObjectName(_fromUtf8("Dialog"))
        Dialog.resize(400, 300)
        Dialog.setSizeGripEnabled(True)
        self.dial = QtGui.QDial(Dialog)
        self.dial.setGeometry(QtCore.QRect(60, 100, 50, 64))
        self.dial.setObjectName(_fromUtf8("dial"))
        self.spinBox = QtGui.QSpinBox(Dialog)
        self.spinBox.setGeometry(QtCore.QRect(190, 120, 54, 25))
        self.spinBox.setObjectName(_fromUtf8("spinBox"))

        self.retranslateUi(Dialog)
        QtCore.QMetaObject.connectSlotsByName(Dialog)

    def retranslateUi(self, Dialog):
        Dialog.setWindowTitle(_translate("Dialog", "Dialog", None))


if __name__ == "__main__":
    import sys
    app = QtGui.QApplication(sys.argv)
    Dialog = QtGui.QDialog()
    ui = Ui_Dialog()
    ui.setupUi(Dialog)
    Dialog.show()
    sys.exit(app.exec_())


```
继承上面的窗口来编写我们的信号与槽：
```python
# -*- coding: utf-8 -*-

"""
Module implementing Dialog.
"""
from PyQt4 import QtGui
from PyQt4.QtGui import *
from PyQt4.QtCore import *

from Ui_SingSloftDialog import Ui_Dialog

class Dialog(QDialog, Ui_Dialog):
    """
    Class documentation goes here.
    """
    def __init__(self, parent=None):
        """
        Constructor

        @param parent reference to the parent widget
        @type QWidget
        """
        QDialog.__init__(self, parent)

        self.setupUi(self)
        # 连接信号与槽
        self.connect(self.dial, SIGNAL('valueChanged(int)'), self.spinBox.setValue)
        self.connect(self.spinBox, SIGNAL('valueChanged(int)'), self.dial.setValue)

if __name__ == "__main__":
    import sys
    app = QtGui.QApplication(sys.argv)
    ui = Dialog()
    ui.show()
    sys.exit(app.exec_())
```
运行结果：

![](/images/blog/20180307151305.png)

如果用户拖动拨号盘为 20，此时拨号盘就会发射一个 valueChange(20) 的信号，相应的就会对输入框的 setValue() 槽进行调整，并将 20 作为参数传递进去。相反输入框的值发生改变拨号盘的槽函数也会触发。貌似会发生死循环，其实不用担心，如果传递额值并未发生改变 valueChange() 就不会再次发射信号。

#### 第二种写法

在上边的拨号盘例子中我们用的是 instance.metnodName() 的语法。但是当槽函数实际上是个 Qt 槽而不是 Python 方法时，用 SLOT() 语法可能会更高效。

```python
self.connect(dial,SIGNAL("valueChanged(int)"),spinbox,SLOT("setValue(int)"))
self.connect(spinbox,SIGNAL("valueChanged(int)"),dial,SLOT("setValue(int)"))
```
使用 QObject.connect() 可以建立各类连接，也可以使用 QObject.disconnect() 来取消这些连接。实际应用中我们并不需要自己去取消这些连接，这是因为，当删除一个对象后，PyQt 就会自动断开改对象的所有连接。

## 发射自定义信号的组件
我们已经知道如何连接信号与槽函数，这些槽就是一些常规的函数或者方法。但是如果我们想创建一个可以发射自定义信号的组件该怎么办呢？使用 QObject.emit() 就可以轻松的实现这一点。
```python
class ZeroSpinBox(QSpinBox):
    zeros =0
    def __init__(self, parent=None):
        super(ZeroSpinBox, self).__init__(parent)
        # 首先是把 valueChange 信号连接到 checkzeor() 函数
        self.connect(self, SIGNAL('valueChanged(int)'),self.checkzeor )
    def checkzeor(self):
        if self.value() ==0:
            self.zeros += 1
            # 发射一个名为 amout 的信号，并且带上参数
            self.emit(SIGNAL('amount'), self.zeros)
```

继续修改刚才的例子：
```Python
class Dialog(QDialog, Ui_Dialog):
    """
    Class documentation goes here.
    """
    def __init__(self, parent=None):
        """
        Constructor

        @param parent reference to the parent widget
        @type QWidget
        """
        QDialog.__init__(self, parent)
        # 初始化刚刚自定义的控件
        zerospinbox = ZeroSpinBox()
        layout = QHBoxLayout()
        layout.addWidget(zerospinbox)
        self.setLayout(layout)
        self.setupUi(self)
        self.connect(self.dial, SIGNAL('valueChanged(int)'), self.spinBox.setValue)
        self.connect(self.spinBox, SIGNAL('valueChanged(int)'), self.dial.setValue)
        # 把控件里边定义的 amount 信号绑定到 announce 函数
        self.connect(zerospinbox, SIGNAL('amount'), self.announce)

    def announce(self, zeros):
        print '等于0的次数' + str(zeros)


if __name__ == "__main__":
    import sys
    app = QtGui.QApplication(sys.argv)
    ui = Dialog()
    ui.show()
    sys.exit(app.exec_())
```
执行结果：

![](/images/blog/20180307154005.png)

## 不同信号连接到同一个槽上
我们之前的例子把不同的信号连接到同一个槽上，也不关心是谁发射了这个信号。有时候我们需要将两个或者更多的信号连接到同一个槽上，并需要根据连接的不同信号做出不同的反应。

如图我们有 5 个按钮和 1 个标签：

![](/images/blog/20180307154951.png)

#### 定义不同的槽
先说最简单的连接，这个连接用在 button1 中。下面是 button1 的连接方式：
```Python
self.connect(button1, SIGNAL("clicked()"), self.one)
```
我们定义一个 one() 方法去改变标签的值：
```Python
def one(self):
        self.label.setText("You clicked button 'One'")
```
#### 使用 Python2.5 之后的高阶函数

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。
```Python
def partial(func, arg):
        def callme():
            return func(arg)
        return callme
```
使用这个高阶函数作为槽函数：
```Python
self.button2callback = partial(self.anyButton, "Two")
self.connect(button2, SIGNAL("clicked()"),self.button2callback)
```
在我们的定义函数里边改变标签的值：
```Python
def anyButton(self, who):
        self.label.setText("You clicked button '%s'" % who)
```

#### 连接到同一个槽函数
如果将不同的槽连接到同一个槽函数我们使用 self.sender() 来发现信号是来自哪个 QObject 对象。

绑定信号：
```Python
self.connect(button4, SIGNAL("clicked()"), self.clicked)
self.connect(button5, SIGNAL("clicked()"), self.clicked)
```
定义函数：
```Python
def clicked(self):
        button = self.sender()
        if button is None or not isinstance(button, QPushButton):
            return
        self.label.setText("You clicked button '%s'" % button.text())
```

## 总结
PyQt 信号与槽的机制需要好好的理解，更重要的是需要大量的实践才能理解。
