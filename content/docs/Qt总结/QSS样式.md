---
title: QSS样式
weight: 3
---

# QSS样式

直接调用控件的 setstylesheet, 结果是每个控件 style 返回的对象都是不同的(地址不同足以证明是不同的对象), 而只给 QApplication 对象 setStyleSheet, 每个控件的 style 函数返回的对象都是相同的. 基于以上原因, 在开发时, 无论是出于维护的便捷性, 还是节省内存资源的考虑, 都应该有 一个 qss 文件来存放所有的样式表, 而不应该将 setStyleSheet 写的到处都是.

默认程序中获取焦点以后会有虚边框，如果看着觉得碍眼不舒服可以去掉，设置样式即可：setStyleSheet("*{outline:0px;}");

outline （轮廓）是控件有焦点时, 绘制在边框边缘的外围,可起到突出作用,轮廓线不占据控 件, 也不一定是矩形

```css
outline: none;
```

width, height 两个属性, 设置的均是盒子的内容的宽高, 而我们在 c++ 代码中的窗口的 width 与 height 指的是整个盒子的宽度与高度,

## 指示器设置样式

1. 可以对整体的指示器设置样式，而不需要单独对每个控件的指示器设置，

```
*::down-arrow{}
*::menu-indicator{}
*::up-arrow:disabled{}
*::up-arrow:off{}
```

## QPushButton左对齐文字

1. QPushButton左对齐文字，需要设置样式表

   ```CSS
   QPushButton{text-align:left;}
   ```

## 三种设置文本的方法

QLabel有三种设置文本的方法，掌握好Qt的属性系统，举一反三，可以做出很多效果。

```C++
//常规办法
ui->label->setText("hello");
//取巧办法
ui->label->setProperty("text", "hello");
//属性大法
ui->label->setStyleSheet("qproperty-text:hello;");
```

## 样式表不起作用

1. Qt中继承QWidget之后，样式表不起作用，解决办法有三个。强烈推荐方法一。

- 方法一：设置属性 this->setAttribute(Qt::WA_StyledBackground, true);
- 方法二：改成继承QFrame，因为QFrame自带paintEvent函数已做了实现，在使用样式表时会进行解析和绘制。
- 方法三：重新实现QWidget的paintEvent函数时，使用QStylePainter绘制。

```
void Widget::paintEvent(QPaintEvent *)
{
    QStyleOption option;
    option.initFrom(this);
    QPainter painter(this);
    style()->drawPrimitive(QStyle::PE_Widget, &option, &painter, this);
}
```

## 直接传入样式表文件路径

设置样式表支持直接传入样式表文件路径，亲测4.7到5.15任意版本，通过查看对应函数的源码可以看到内部会检查是否是 'file:///' 开头，是的话则自动读取样式表文件进行设置，无需手动读取。

```C++
//以前都是下面的方法
QFile file(":/qss/psblack.css");
if (file.open(QFile::ReadOnly)) {
    QString qss = QLatin1String(file.readAll());
    qApp->setStyleSheet(qss);
    file.close();
}

//其实一行代码就行
qApp->setStyleSheet("file:///:/qss/psblack.css");
//特别说明，只支持qApp->setStyleSheet 不支持其他比如widget->setStyleSheet
```

## QCheckBox

对于QListView（QListWidget）、QTreeView（QTreeWidget）、QTableView（QTableWidget）这种类型的控件，可以通过setChecked来让对应的item产生复选框效果，很多人（包括曾经的自己）误以为这就是复选框控件，其实不是的，他是对应控件的indicator指示器，所以想要更换样式，不能说设置了QCheckBox的样式就有效果，而要单独对齐indicator指示器设置样式才行。

```css
QCheckBox::indicator,QGroupBox::indicator,QTreeWidget::indicator,QListWidget::indicator{
width:13px;
height:13px;
}

QCheckBox::indicator:unchecked,QGroupBox::indicator:unchecked,QTreeWidget::indicator:unchecked,QListWidget::indicator:unchecked{
image:url(:/qss/flatwhite/checkbox_unchecked.png);
}

QCheckBox::indicator:unchecked:disabled,QGroupBox::indicator:unchecked:disabled,QTreeWidget::indicator:unchecked:disabled,QListWidget::indicator:disabled{
image:url(:/qss/flatwhite/checkbox_unchecked_disable.png);
}

QCheckBox::indicator:checked,QGroupBox::indicator:checked,QTreeWidget::indicator:checked,QListWidget::indicator:checked{
image:url(:/qss/flatwhite/checkbox_checked.png);
}

QCheckBox::indicator:checked:disabled,QGroupBox::indicator:checked:disabled,QTreeWidget::indicator:checked:disabled,QListWidget::indicator:checked:disabled{
image:url(:/qss/flatwhite/checkbox_checked_disable.png);
}

QCheckBox::indicator:indeterminate,QGroupBox::indicator:indeterminate,QTreeWidget::indicator:indeterminate,QListWidget::indicator:indeterminate{
image:url(:/qss/flatwhite/checkbox_parcial.png);
}

QCheckBox::indicator:indeterminate:disabled,QGroupBox::indicator:indeterminate:disabled,QTreeWidget::indicator:indeterminate:disabled,QListWidget::indicator:indeterminate:disabled{
image:url(:/qss/flatwhite/checkbox_parcial_disable.png);
}
```

## QTabBar

QTabWidget选项卡有个自动生成按钮切换选项卡的机制，有时候不想看到这个烦人的切换按钮，可以设置usesScrollButtons为假，其实QTabWidget的usesScrollButtons属性最终是应用到QTabWidget的QTabBar对象上，所以只要设置全局的QTabBar的这个属性关闭即可。为啥要设置全局的呢，因为如果只是对QTabWidget设置了该属性，而在QMainWindow窗体中QDockWidget合并自动形成的选项卡只有QTabBar对象导致依然是有切换按钮。

```
//对tabWidget设置无切换按钮
ui->tabWidget->setUsesScrollButtons(false);
//对tabBar设置无切换按钮
ui->tabWidget->tabBar()->setUsesScrollButtons(false);
//对整个系统的选项卡设置无切换按钮
QTabBar{qproperty-usesScrollButtons:false;}
//设置选项卡自动拉伸 这玩意居然之前自动计算来设置原来内置了哇咔咔
QTabBar{qproperty-expanding:false;}
//设置选项卡关闭按钮可见
QTabBar{qproperty-tabsClosable:true;}
//还有其他属性参见QTabBar头文件有惊喜
//依旧是万能大法所有可视化类的 Q_PROPERTY 包含的属性都可以这样设置
```

## 设置字体

设置字体，大概都会经历一个误区，本来是打算设置整个窗体包括子控件的字体大小的，结果发现只有主窗体自己应用了字体而子控件没有。

```C++
//假设窗体中有子控件，默认字体12px，父类类型是QWidget，父类类名是Widget

//下面几种方法只会设置主窗体的字体，子控件不会应用，需要按个调用setFont
QFont font;
font.setPixelSize(20);
this->setFont(font);
this->setStyleSheet("{font:26px;}");
this->setStyleSheet("QWidget{font:26px;}");
this->setStyleSheet("Widget{font:26px;}");

//下面才是通过样式表设置整个控件+子控件的字体
this->setStyleSheet("font:26px;");
this->setStyleSheet("*{font:26px;}");
this->setStyleSheet("QWidget>*{font:26px;}");
this->setStyleSheet("Widget>*{font:26px;}");

//下面设置全局字体
qApp->setFont(font);
```



```css
*{font: normal 20px “微软雅黑”;}

QPushButton{
color: blue;
}

namespace ns {
 class MyPushButton : public QPushButton {
 // ...
 }
}
// ...
qApp->setStyleSheet("ns--MyPushButton { background: yellow; }");

.QPushButton{
color: blue;
}

QAbstractSpinBox{
min-height: 30px;
max-height: 30px;
border-width: 1px;
rder-style: solid;
order-color: gray;
padding: 0px;
}

#button_1{
 color: red;
}

QPushButton#settings_popup_fileDialog_button{
 min-height: 31px;
 min-width: 70px;
 border: 1px solid black;
 color: #F0F0F0;
 min-height: 10px;
 border-radius:3px;
 background: qlineargradient(spread:pad, x1:0, y1:0, x2:0, y2:1,
stop:0 #454648, stop:1 #7A7A7A);
}

BaseDialog QPushButton{
 min-width: 120px;
 min-height: 40px;
 max-width: 120px;
 max-height: 40px;
 font-size: 20px;
 padding: 0px;
}

.QGroupBox > .QCheckBox{
 color: blue;
}

[objectName~="button"]{
 color: red;
}

.QLineEdit, .QComboBox{
 border: 1px solid gray;
 background-color:white;
}
```

## qss优先级

给控件直接设置的样式 > 给 QApplication 设置的样式

如果是间接选中,那么最终的样式就是离目标最近的那个

后面的样式会覆盖掉前面的样 式

Id > 类 > 类型 > 通配符 > 继承 > 默认
