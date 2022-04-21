---
title: snippet
weight: 2
---

## 延时异步加载

用QTimer::singleShot单次定时器和QMetaObject::invokeMethod可以解决意想不到的问题。比如在窗体初始化的时候加载一个耗时的操作，很容易卡主界面的显示，要在加载完以后才会显示界面，这就导致了体验很卡不友好的感觉，此时你可以将耗时的加载（有时候这些加载又必须在主线程，比如用QStackWidget堆栈窗体加载一些子窗体），延时或者异步进行加载，这样就会在界面显示后去执行，而不是卡住主界面。

```
//异步执行load函数
QMetaObject::invokeMethod(this, "load", Qt::QueuedConnection);
//延时10毫秒执行load函数
QTimer::singleShot(10, this, SLOT(load()));
```

## 获取类的属性和方法

```C++
//拿到控件元对象
const QMetaObject *metaObject = widget->metaObject();

//所有属性的数量
int propertyCount = metaObject->propertyCount();
//propertyOffset是自定义的属性开始的位置
int propertyOffset = metaObject->propertyOffset();
//循环取出控件的自定义属性, int i = 0 表示所有属性
for (int i = propertyOffset; i < propertyCount; ++i) {
    QMetaProperty metaProperty = metaObject->property(i);
    const char *name = metaProperty.name();
    const char *type = metaProperty.typeName();
    QVariant value = widget->property(name);
    qDebug() << name << type << value;
}

//所有方法的数量
int methodCount = metaObject->methodCount();
//methodOffset是自定义的方法开始的位置
int methodOffset = metaObject->methodOffset();
//循环取出控件的自定义方法, int i = 0 表示所有方法
for (int i = methodOffset; i < methodCount; ++i) {
    QMetaMethod metaMethod = metaObject->method(i);
    const char *name = metaMethod.name();
    const char *type = metaMethod.typeName();
    qDebug() << name << type;
}
```

## inherits判断是否属于某种类

```
QTimer *timer = new QTimer;         // QTimer inherits QObject
timer->inherits("QTimer");          // returns true
timer->inherits("QObject");         // returns true
timer->inherits("QAbstractButton"); // returns false
```

使用弱属性机制，可以存储临时的值用于传递判断。可以通过widget->dynamicPropertyNames()列出所有弱属性名称，然后通过widget->property("name")取出对应的弱属性的值。

## sqlite数据库

1. 如果使用sqlite数据库不想产生数据库文件，可以创建内存数据库。

```
QSqlDatabase db = QSqlDatabase::addDatabase("QSQLITE");
db.setDatabaseName(":memory:");
```

清空数据表并重置自增ID

```SQL
truncate table table_name
```

## 废弃或者过时函数

从Qt4转到Qt5，有些类的方法已经废弃或者过时了，如果想要在Qt5中启用Qt4的方法，比如QHeadVew的setMovable，可以在你的pro或者pri文件中加上一行即可：DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0

## 编译环境和运行环境

x86/x64都编译环境和运行环境是编译环境和运行环境相同，没有或。带下划线的就是交叉编译，前面是编译环境，后面是运行环境。

| 名称      | 说明                                 |
| --------- | ------------------------------------ |
| x86       | 32/64位系统上编译在32/64位系统上运行 |
| x86_amd64 | 32/64位系统上编译在64位系统上运行    |
| x86_arm   | 32/64位系统上编译在arm系统上运行     |
| amd64     | 64位系统上编译在64位系统上运行       |
| amd64_x86 | 64位系统上编译在32/64位系统上运行    |
| amd64_arm | 64位系统上编译在arm系统上运行        |

## 窗口模态显示

很多时候用QDialog的时候会发现阻塞了消息，而有的时候我们希望是后台的一些消息继续运行不要终止，此时需要做个设置。

```
QDialog dialog;
dialog.setWindowModality(Qt::WindowModal);
```

## 无边框窗体输入焦点

在嵌入式linux上，如果设置了无边框窗体，而该窗体中又有文本框之类的，发现没法产生焦点进行输入，此时需要主动激活窗体才行。

```C++
//这种方式设置的无边框窗体在嵌入式设备上无法产生焦点
setWindowFlags(Qt::WindowStaysOnTopHint | Qt::FramelessWindowHint | Qt::X11BypassWindowManagerHint);

//需要在show以后主动激活窗体
w->show();
w->activateWindow();
```

## QMetaObject::invokeMethod

巧用QMetaObject::invokeMethod方法可以实现很多效果，包括同步和异步执行，很大程度上解决了跨线程处理信号槽的问题。比如有个应用场景是在回调中，需要异步调用一个public函数，如果直接调用的话会发现不成功，此时需要使用 QMetaObject::invokeMethod(obj, "fun", Qt::QueuedConnection); 这种方式来就可以。

- invokeMethod函数有很多重载参数，可以传入返回值和执行方法的参数等。
- invokeMethod函数不仅支持槽函数还支持信号，而且这逼居然是线程安全的，可以在线程中放心使用，牛逼！
- 测试下来发现只能执行signals或者slots标识的方法。
- 默认可以执行private(protected/public) slots下的函数，但是不能执行private(protected/public)下的函数。
- 必须是slots或者signals标注的函数，不是标注的函数不在元信息导致无法查找，执行之后会提示No such method。
- 如果要执行private(protected/public)下的函数，需要函数前面加上 Q_INVOKABLE 关键字

```C++
//头文件声明信号和槽函数
signals:
    void sig_test(int type,double value);
private slots:
    void slot_test(int type, double value);
private:
    Q_INVOKABLE void fun_test(int type, double value);

//构造函数关联信号槽
connect(this, SIGNAL(sig_test(int, double)), this, SLOT(slot_test(int, double)));

//单击按钮触发信号和槽,这里是同时举例信号槽都可以
void MainWindow::on_pushButton_clicked()
{
    QMetaObject::invokeMethod(this, "sig_test", Q_ARG(int, 66), Q_ARG(double, 66.66));
    QMetaObject::invokeMethod(this, "slot_test", Q_ARG(int, 88), Q_ARG(double, 88.88));
    QMetaObject::invokeMethod(this, "fun_test", Q_ARG(int, 99), Q_ARG(double, 99.99));
}

//会打印 66 66.66、88 88.88
void MainWindow::slot_test(int type, double value)
{
    qDebug() << type << value;
}

//会打印 99.99
void MainWindow::fun_test(int type, double value)
{
    qDebug() << type << value;
}
```

## Qt的定时器精度

Qt的默认定时器精度不够高（比如应用场景是1分钟保存一条记录或者文件，当你用默认的定时器的时候你会发现有些时候是60秒而有些是59秒随机的，如果客户有要求这就需要设置精度了。当然我们所做的绝大部分项目也不需要精度非常高的定时器，毕竟精度越高，占用的系统资源可能越大），如果需要设置更高的精度可以设置 setTimerType(Qt::PreciseTimer)。Qt有两种定时器处理，一种是QTimer类，还有一种是QObject类就内置的timeevent事件，如果是QObject类的定时器要设置的话调用 startTimer(interval, Qt::PreciseTimer);

- Qt::PreciseTimer 精确的定时器，尽量保持毫秒精度。
- Qt::CoarseTimer 粗略的定时器，尽量保持精度在所需的时间间隔5%范围内。
- Qt::VeryCoarseTimer 很粗略的定时器，只保留完整的第二精度。
- 精度再高，也依赖对应的操作系统中断，假设中断需要 5ms，则定时器精度不可能高于5毫秒。

## QRegExpValidator 正则表达

1. QLineEdit除了单纯的文本框以外，还可以做很多特殊的处理用途。

- 限制输入只能输入IP地址。
- 限制输入范围，强烈推荐使用 QRegExpValidator 正则表达式来处理。

```C++
//正在表达式限制输入
QString str = "\\b(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\b";
ui->lineEdit->setValidator(new QRegExpValidator(QRegExp(str)));
//用于占位
ui->lineEdit->setInputMask("000.000.000.000");

#if 0
//下面代码设置浮点数范围限制失败
ui->lineEdit->setValidator(new QDoubleValidator(20, 50, 1));
#else
//下面代码设置浮点数范围限制成功
QDoubleValidator *validator = new QDoubleValidator(20, 50, 1);
validator->setNotation(QDoubleValidator::StandardNotation);
ui->lineEdit->setValidator(validator);
#endif
//下面代码设置整数范围限制成功
ui->lineEdit->setValidator(new QIntValidator(10, 120));

//其实上面的代码缺陷很多，只能限制只输入小数，无法设定数值范围，很操蛋
//需要来个万能的牛逼的 QRegExpValidator

//限制浮点数输入范围为[-180,180]
QRegExp regexp("^-?(180|1?[0-7]?\\d(\\.\\d+)?)$");
//限制浮点数输入范围为[-90,90]并限定为小数位后4位
QRegExp regexp("^-?(90|[1-8]?\\d(\\.\\d{1,4})?)$");
QRegExpValidator *validator = new QRegExpValidator(regexp, this);
ui->lineEdit->setValidator(validator);
```

## Qt重载qDebug输出自定义的信息

```c++
struct FunctionInfo {
    QString function;
    QString name;
    QString groupEnabled;
    QString action;
    QString group;

    friend QDebug operator << (QDebug debug, const FunctionInfo &functionInfo) {
        QString info = QString("功能: %1  名称: %2  启用: %3  方法: %4  分组: %5")
                       .arg(functionInfo.function).arg(functionInfo.name).arg(functionInfo.groupEnabled)
                       .arg(functionInfo.action).arg(functionInfo.group);
        debug << info;
        return debug;
    }
};
```

## findChild使用

Qt中自带的很多控件，其实都是由一堆基础控件（QLabel、QPushButton等）组成的，比如日历面板 QCalendarWidget 就是 QToolButton+QSpinBox+QTableView 等组成，妙用 findChildren 可以拿到父类对应的子控件集合，可以直接对封装的控件中的子控件进行样式的设置，其他参数的设置比如设置中文文本（默认可能是英文）等。

```C++
//打印子类类名集合
void printObjectChild(const QObject *obj, int spaceCount)
{
    qDebug() << QString("%1%2 : %3")
             .arg("", spaceCount)
             .arg(obj->metaObject()->className())
             .arg(obj->objectName());

    QObjectList childs = obj->children();
    foreach (QObject *child, childs) {
        printObjectChild(child, spaceCount + 2);
    }
}

//拿到对话框进行设置和美化
QFileDialog *fileDialog = new QFileDialog(this);
fileDialog->setOption(QFileDialog::DontUseNativeDialog, true);
QLabel *lookinLabel = fileDialog->findChild<QLabel*>("lookInLabel");
lookinLabel->setText(QString::fromLocal8Bit("文件目录："));
lookinLabel->setStyleSheet("color:red;");

//设置日期框默认值为空
QLineEdit *edit = ui->dateEdit->findChild<QLineEdit *>("qt_spinbox_lineedit");
if (!edit->text().isEmpty()) {
    edit->clear();
}
```

巧妙的使用 findChildren 可以查找该控件下的所有子控件。 findChild 为查找单个。

```C++
//查找指定类名objectName的控件
QList<QWidget *> widgets = fatherWidget.findChildren<QWidget *>("widgetname");
//查找所有QPushButton
QList<QPushButton *> allPButtons = fatherWidget.findChildren<QPushButton *>();
//查找一级子控件,不然会一直遍历所有子控件
QList<QPushButton *> childButtons = fatherWidget.findChildren<QPushButton *>(QString(), Qt::FindDirectChildrenOnly);
```

## 动态加载资源文件

当Qt中编译资源文件太大时，效率很低，或者需要修改资源文件中的文件比如图片、样式表等，需要重新编译可执行文件，这样很不友好，当然Qt都给我们考虑好了策略，此时可以将资源文件转化为二进制的rcc文件，这样就将资源文件单独出来了，可在需要的时候动态加载。

```C++
//Qt中使用二进制资源文件方法如下
//将qrc编译为二进制文件rcc，在控制台执行下列命令 
rcc -binary main.qrc -o main.rcc
//在应用程序中注册资源，一般在main函数启动后就注册
QResource::registerResource(qApp->applicationDirPath() + "/main.rcc");
```

Qt默认不支持大资源文件，比如添加了字体文件，需要pro文件开启。 CONFIG += resources_big

## blockSignals阻塞信号

```C++
//方法1：先 disconnect 掉信号，处理好以后再 connect 信号，缺点很明显，很傻，如果信号很多，每个型号都要这么来一次。
disconnect(ui->cbox, SIGNAL(currentIndexChanged(int)), this, SLOT(on_cbox_currentIndexChanged(int)));
for (int i = 0; i <= 100; i++) {
    ui->cbox->addItem(QString::number(i));
}
connect(ui->cbox, SIGNAL(currentIndexChanged(int)), this, SLOT(on_cbox_currentIndexChanged(int)));

//方法2：先调用 blockSignals(true) 阻塞信号，处理号以后再调用 blockSignals(false) 恢复所有信号。
//如果需要指定某个信号进行断开那就只能用 disconnect 来处理。
ui->cbox->blockSignals(true);
for (int i = 0; i <= 100; i++) {
    ui->cbox->addItem(QString::number(i));
}
ui->cbox->blockSignals(false);
```

## QCustomPlot使用

```c++
//对调XY轴，在最前面设置
QCPAxis *yAxis = customPlot->yAxis;
QCPAxis *xAxis = customPlot->xAxis;
customPlot->xAxis = yAxis;
customPlot->yAxis = xAxis;

//移除图例
customPlot->legend->removeItem(1);

//合并两个曲线画布形成封闭区域
customPlot->graph(0)->setChannelFillGraph(customPlot->graph(1));

//关闭抗锯齿以及设置拖动的时候不启用抗锯齿
customPlot->graph()->setAntialiased(false);
customPlot->setNoAntialiasingOnDrag(true);

//多种设置数据的方法
customPlot->graph(0)->setData();
customPlot->graph(0)->data()->set();

//设置不同的线条样式、数据样式
customPlot->graph()->setLineStyle(QCPGraph::lsLine);
customPlot->graph()->setScatterStyle(QCPScatterStyle::ssDot);
customPlot->graph()->setScatterStyle(QCPScatterStyle(shapes.at(i), 10));

//还可以设置为图片或者自定义形状
customPlot->graph()->setScatterStyle(QCPScatterStyle(QPixmap("./sun.png")));
QPainterPath customScatterPath;
for (int i = 0; i < 3; ++i) {
    customScatterPath.cubicTo(qCos(2 * M_PI * i / 3.0) * 9, qSin(2 * M_PI * i / 3.0) * 9, qCos(2 * M_PI * (i + 0.9) / 3.0) * 9, qSin(2 * M_PI * (i + 0.9) / 3.0) * 9, 0, 0);
}
customPlot->graph()->setScatterStyle(QCPScatterStyle(customScatterPath, QPen(Qt::black, 0), QColor(40, 70, 255, 50), 10));

//更换坐标轴的箭头样式
customPlot->xAxis->setUpperEnding(QCPLineEnding::esSpikeArrow);
customPlot->yAxis->setUpperEnding(QCPLineEnding::esSpikeArrow);

//设置背景图片
customPlot->axisRect()->setBackground(QPixmap("./solarpanels.jpg"));
//画布也可以设置背景图片
customPlot->graph(0)->setBrush(QBrush(QPixmap("./balboa.jpg")));
//整体可以设置填充颜色或者图片
customPlot->setBackground(QBrush(gradient));
//设置零点线条颜色
customPlot->xAxis->grid()->setZeroLinePen(Qt::NoPen);
//控制是否鼠标滚轮缩放拖动等交互形式
customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom | QCP::iSelectPlottables);

//柱状分组图
QCPBarsGroup *group = new QCPBarsGroup(customPlot);
QList<QCPBars*> bars;
bars << fossil << nuclear << regen;
foreach (QCPBars *bar, bars) {
    //设置柱状图的宽度大小
    bar->setWidth(bar->width() / bars.size());
    group->append(bar);
}
//设置分组之间的间隔
group->setSpacing(2);
```

## 网络请求超时时间

在网络请求中经常涉及到超时时间的问题，因为默认是30秒钟，一旦遇到网络故障的时候要等好久才能反应过来，所以需要主动设置下超时时间，超过了就直接中断结束请求。从Qt5.15开始内置了setTransferTimeout来设置超时时间，非常好用。

```C++
//局部的事件循环,不卡主界面
QEventLoop eventLoop;

//设置超时 5.15开始自带了超时时间函数 默认30秒
#if (QT_VERSION >= QT_VERSION_CHECK(5,15,0))
manager->setTransferTimeout(timeout);
#else
QTimer timer;
connect(&timer, SIGNAL(timeout()), &eventLoop, SLOT(quit()));
timer.setSingleShot(true);
timer.start(timeout);
#endif

QNetworkReply *reply = manager->get(QNetworkRequest(QUrl(url)));
connect(reply, SIGNAL(finished()), &eventLoop, SLOT(quit()));
eventLoop.exec();

if (reply->bytesAvailable() > 0 && reply->error() == QNetworkReply::NoError) {
    //读取所有数据保存成文件
    QByteArray data = reply->readAll();
    QFile file(dirName + fileName);
    if (file.open(QFile::WriteOnly | QFile::Truncate)) {
        file.write(data);
        file.close();
    }
}
```

## Qt获取当前所用的Qt版本、编译器、位数等信息。

```C++
//详细的Qt版本+编译器+位数
QString compilerString = "<unknown>";
{
    compilerString = QLatin1String("GCC ") + QLatin1String(__VERSION__);
}

//拓展知识 查看 QSysInfo 类下面有很多好东西
// qVersion() = QT_VERSION_STR
QString version = QString("%1 %2 %3").arg(qVersion()).arg(compilerString).arg(QString::number(QSysInfo::WordSize));
```



## Lamda形式信号槽

```C++
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    //早期写法,通用Qt所有版本,只支持定义了slots关键字的函数
    //connect(ui->pushButton, SIGNAL(clicked()), this, SLOT(test_fun()));
    connect(ui->pushButton, SIGNAL(clicked()), this, SLOT(test_slot()));

    //新写法,支持Qt5及后期所有版本,支持所有函数,无需定义slots关键字也行
    //采用这种写法，如果编译的时候信号或槽不存在是无法编译通过的，相当于编译时检查，不容易出错；
    connect(ui->pushButton, &QPushButton::clicked, this, &MainWindow::test_fun);
    connect(ui->pushButton, &QPushButton::clicked, this, &MainWindow::test_slot);
        
    //按钮单击不带参数
    connect(ui->pushButton, &QPushButton::clicked, [] {
        qDebug() << "hello lambda";
    });

    //按钮单击带参数
    connect(ui->pushButton, &QPushButton::clicked, [] (bool isCheck) {
        qDebug() << "hello lambda" << isCheck;
    });

    //自定义信号带参数
    connect(this, &MainWindow::sig_test, [] (int i, int j) {
        qDebug() << "hello lambda" << i << j;
    });

    emit sig_test(5, 8);
}
```

## Qt延时方法

```c++
void QUIHelperCore::sleep(int msec)
{
    if (msec <= 0) {
        return;
    }

#if 1
    //非阻塞方式延时,现在很多人推荐的方法
    QEventLoop loop;
    QTimer::singleShot(msec, &loop, SLOT(quit()));
    loop.exec();
#else
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    //阻塞方式延时,如果在主线程会卡住主界面
    QThread::msleep(msec);
#else
    //非阻塞方式延时,不会卡住主界面,据说可能有问题
    QTime endTime = QTime::currentTime().addMSecs(msec);
    while (QTime::currentTime() < endTime) {
        QCoreApplication::processEvents(QEventLoop::AllEvents, 100);
    }
#endif
#endif
}
```

## 获取当前屏幕索引以及尺寸

```c++
//获取当前屏幕索引
int QUIHelper::getScreenIndex()
{
    //需要对多个屏幕进行处理
    int screenIndex = 0;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    int screenCount = qApp->screens().count();
#else
    int screenCount = qApp->desktop()->screenCount();
#endif

    if (screenCount > 1) {
        //找到当前鼠标所在屏幕
        QPoint pos = QCursor::pos();
        for (int i = 0; i < screenCount; ++i) {
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
            if (qApp->screens().at(i)->geometry().contains(pos)) {
#else
            if (qApp->desktop()->screenGeometry(i).contains(pos)) {
#endif
                screenIndex = i;
                break;
            }
        }
    }
    return screenIndex;
}

//获取当前屏幕尺寸区域
QRect QUIHelper::getScreenRect(bool available)
{
    QRect rect;
    int screenIndex = QUIHelper::getScreenIndex();
    if (available) {
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
        rect = qApp->screens().at(screenIndex)->availableGeometry();
#else
        rect = qApp->desktop()->availableGeometry(screenIndex);
#endif
    } else {
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
        rect = qApp->screens().at(screenIndex)->geometry();
#else
        rect = qApp->desktop()->screenGeometry(screenIndex);
#endif
    }
    return rect;
}
```

## 文本进行分散对齐显示

有时候需要对文本进行分散对齐显示，相当于无论文字多少，尽可能占满整个空间平摊占位宽度，但是在对支持对齐方式的控件比如QLabel调用 setAlignment(Qt::AlignJustify | Qt::AlignVCenter) 设置分散对齐会发现没有任何效果，这个时候就要考虑另外的方式比如通过控制字体的间距来实现分散对齐效果。

```C++
QString text = "测试分散对齐内容";
//计算当前文本在当前字体下占用的宽度
QFont font = ui->label->font();
int textWidth = ui->label->fontMetrics().width(text);
//显示文本的区域宽度=标签的宽度-两边的边距
int width = ui->label->width() - 12;
//需要-1相当于中间有几个间隔
int count = text.count() - 1;
//计算每个间距多少
qreal space = qreal(width - textWidth) / count;
//设置固定间距
font.setLetterSpacing(QFont::AbsoluteSpacing, space);
ui->label->setFont(font);
ui->label->setText(text);
```



## override关键字

```c++
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    void enterEvent(QEnterEvent *);
#else
    void enterEvent(QEvent *);
#endif

//后面经过JasonWong大佬的指点，从父类重新实现的virtual修饰的函数，建议都加上override关键字。
//这样的话一旦父类的函数或者参数变了则会提示编译报错，而不是编译通过但是运行不正常会一脸懵逼茫然，从而把锅扣给Qt。

//下面是父类函数
virtual void enterEvent(QEvent *event);
//子类建议加上override
void enterEvent(QEvent *event) override;
```

# 升级到Qt6

1. 源码中的double数据类型全部换成了qreal，和Qt内部数据类型高度一致和统一。

1. QFontMetricsF 中的 fm.width() 换成 fm.horizontalAdvance() ，从5.11开始用新函数。

1. QApplication::desktop()废弃了， 换成了 QApplication::primaryScreen()。

```c++
#if (QT_VERSION > QT_VERSION_CHECK(5,0,0))
#include "qscreen.h"
#define deskGeometry qApp->primaryScreen()->geometry()
#define deskGeometry2 qApp->primaryScreen()->availableGeometry()
#else
#include "qdesktopwidget.h"
#define deskGeometry qApp->desktop()->geometry()
#define deskGeometry2 qApp->desktop()->availableGeometry()
#endif
```
