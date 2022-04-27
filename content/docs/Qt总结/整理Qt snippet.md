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



# 常见类使用

## QTableWidget

关于QTableWidget等控件调用自带的removeRow、clearContents、clear函数删除了里面的item和内容，会自动调用item或者cellwidget的析构函数进行资源释放，不用自己手动再去释放。

```c++
//每次调用 clearContents 都会自动清理之前的item
ui->tableWidget->clearContents();
for (int i = 0; i < count; ++i) {
    ui->tableWidget->setItem(i, 0, new QTableWidgetItem("aaa"));
    ui->tableWidget->setItem(i, 1, new QTableWidgetItem("bbb"));
    ui->tableWidget->setCellWidget(i, 2, new QPushButton("ccc"));
}
```

QTabWidget选项卡控件，生成的tabbar选项卡宽度是按照文本自动设置的，文本越长选项卡的宽度越大，很多时候，我们需要的是一样的宽度或者等分填充，

```c++
//方法1：字符串空格填充
ui->tabWidget->addTab(httpClient1, "测    试");
ui->tabWidget->addTab(httpClient1, "人员管理");
ui->tabWidget->addTab(httpClient1, "系统设置");

//方法2：识别尺寸改变事件自动设置最小宽度
void MainWindow::resizeEvent(QResizeEvent *e)
{
    int count = ui->tabWidget->tabBar()->count();
    int width = this->width() - 30;
    QString qss = QString("QTabBar::tab{min-width:%1px;}").arg(width / count);
    this->setStyleSheet(qss);
}

//方法3：设置全局样式，不同选项卡个数的设置不同的宽度
QStringList list;
list << QString("QTabWidget[tabCount=\"2\"]>QTabBar::tab{min-width:%1px;}").arg(100);
list << QString("QTabWidget[tabCount=\"3\"]>QTabBar::tab{min-width:%1px;}").arg(70);
qApp->setStyleSheet(list.join(""));
//设置了tabCount弱属性自动去找对应的宽度设置
ui->tabWidget->setProperty("tabCount", 2);
ui->tabWidget->setProperty("tabCount", 3);

//方法4：强烈推荐-》使用内置的方法 setExpanding setDocumentMode 两个属性都必须设置
//Qt4的tabBar()是propected的，所以建议还是通过样式表设置
ui->tabWidget->tabBar()->setDocumentMode(true);
ui->tabWidget->tabBar()->setExpanding(true);
//样式表一步到位不用每个都单独设置
QString("QTabBar{qproperty-usesScrollButtons:false;qproperty-documentMode:true;qproperty-expanding:true;}");
//在5.9以前开启这个设置后，貌似选项卡个数按照真实个数+1计算宽度，也就是永远会留空一个tab的占位。
//5.9以后貌似修复了这个BUG，按照理想中的拉伸填充等分设置tab的宽度。
```

## QTableView

关于QTableView（采用model数据源）、QTableWidget列名列宽设置，有时候发现没有起作用，原来是对代码设置的顺序有要求，比如setColumnWidth前必须先setColumnCount，不然列数都没有，哪来的列宽，包括setHorizontalHeaderLabels设置列标题集合也是，前提都要先有列。

```c++
void frmSimple::initForm()
{
    //实例化数据模型
    model = new QStandardItemModel(this);

    //设置行数列数
    row = 100;
    column = 10;
    //设置列名列宽
    for (int i = 0; i < column; ++i) {
        columnNames << QString("列%1").arg(i + 1);
        columnWidths << 60;
    }
}

void frmSimple::on_btnLoad1_clicked()
{
    //先设置数据模型,否则 setColumnWidth 不起作用
    ui->tableView->setModel(model);

    //设置列数及列标题和列宽
    model->setColumnCount(column);
    //简便方法设置列标题集合
    model->setHorizontalHeaderLabels(columnNames);
    for (int i = 0; i < column; ++i) {
        ui->tableView->setColumnWidth(i, columnWidths.at(i));
    }

    //循环添加行数据
    QDateTime now = QDateTime::currentDateTime();
    model->setRowCount(row);
    for (int i = 0; i < row; ++i) {
        for (int j = 0; j < column; ++j) {
            QStandardItem *item = new QStandardItem;
            //最后一列显示时间区别开来
            if (j == column - 1) {
                item->setText(now.addSecs(i).toString("yyyy-MM-dd HH:mm:ss"));
            } else {
                item->setText(QString("%1_%2").arg(i + 1).arg(j + 1));
            }
            model->setItem(i, j, item);
        }
    }
}

void frmSimple::on_btnLoad2_clicked()
{
    //设置列标题和列数及列宽
    ui->tableWidget->setColumnCount(column);
    //简便方法设置列标题集合
    ui->tableWidget->setHorizontalHeaderLabels(columnNames);
    for (int i = 0; i < column; ++i) {
        ui->tableWidget->setColumnWidth(i, columnWidths.at(i));
    }

    //添加数据
    QDateTime now = QDateTime::currentDateTime();
    ui->tableWidget->setRowCount(row);
    for (int i = 0; i < row; ++i) {
        for (int j = 0; j < column; ++j) {
            QTableWidgetItem *item = new QTableWidgetItem;
            //最后一列显示时间区别开来
            if (j == column - 1) {
                item->setText(now.addSecs(i).toString("yyyy-MM-dd HH:mm:ss"));
            } else {
                item->setText(QString("%1_%2").arg(i + 1).arg(j + 1));
            }
            ui->tableWidget->setItem(i, j, item);
        }
    }
}
```

Qt表格控件一些常用的设置封装，QTableWidget继承自QTableView，所以下面这个函数支持传入QTableWidget。

```C++
void QUIHelper::initTableView(QTableView *tableView, int rowHeight, bool headVisible, bool edit)
{
    //奇数偶数行颜色交替
    tableView->setAlternatingRowColors(false);
    //垂直表头是否可见
    tableView->verticalHeader()->setVisible(headVisible);
    //选中一行表头是否加粗
    tableView->horizontalHeader()->setHighlightSections(false);
    //最后一行拉伸填充
    tableView->horizontalHeader()->setStretchLastSection(true);
    //行标题最小宽度尺寸
    tableView->horizontalHeader()->setMinimumSectionSize(0);
    //行标题最大高度
    tableView->horizontalHeader()->setMaximumHeight(rowHeight);
    //默认行高
    tableView->verticalHeader()->setDefaultSectionSize(rowHeight);
    //选中时一行整体选中
    tableView->setSelectionBehavior(QAbstractItemView::SelectRows);
    //只允许选择单个
    tableView->setSelectionMode(QAbstractItemView::SingleSelection);

    //表头不可单击
#if (QT_VERSION > QT_VERSION_CHECK(5,0,0))
    tableView->horizontalHeader()->setSectionsClickable(false);
#else
    tableView->horizontalHeader()->setClickable(false);
#endif

    //鼠标按下即进入编辑模式
    if (edit) {
        tableView->setEditTriggers(QAbstractItemView::CurrentChanged | QAbstractItemView::DoubleClicked);
    } else {
        tableView->setEditTriggers(QAbstractItemView::NoEditTriggers);
    }
}
```

## QSqlTableModel

QSqlTableModel大大简化了对数据库表的显示、添加、删除、修改等，唯独对数据库分页操作有点绕弯。

```c++
//实例化数据库表模型
QSqlTableModel *model = new QSqlTableModel(this);
//指定表名
model->setTable("table");
//设置列排序
model->setSort(0, Qt::AscendingOrder);
//设置提交模式
model->setEditStrategy(QSqlTableModel::OnManualSubmit);
//立即查询一次
model->select();
//将数据库表模型设置到表格上
ui->tableView->setModel(model);

//测试发现过滤条件中除了可以带where语句还可以带排序及limit等
model->setFilter("1=1 order by id desc limit 100");

//如果在过滤条件中设置了排序语句则不可以再使用setSort方法
//下面的代码结果是执行出错，可能因为setSort又重新增加了order by语句导致多个order by语句冲突了。
model->setSort(0, Qt::AscendingOrder);
model->setFilter("1=1 order by id desc limit 100");

//通过setFilter设置单纯的where语句可以不用加1=1
model->setFilter("name='张三'");
//如果还有其他语句比如排序或者limit等则需要最前面加上1=1
//下面表示按照id升序排序，查询结果显示第5-15条记录。
model->setFilter("1=1 order by id asc limit 5,10");

//多个条件用and连接
//建议任何时候用了setFilter则最前面写1=1最末尾加上 ; 防止有些地方无法正确执行。
model->setFilter("1=1 and name='张三' and result>=70;");

//下面表示查询姓名是张三的记录，按照id字段降序排序，结果从第10条开始100条，相当于从第10条到110条记录。
model->setFilter("1=1 and name='张三' order by id desc limit 10,100;");

//在第3行开始添加一条记录
model->insertRow(2);
//立即填充刚刚新增加的行，默认为空需要用户手动在表格中输入。
model->setData(model->index(2, 0), 100);
model->setData(model->index(2, 1), "张三");
//提交更新
model->submitAll();

//删除第4行
model->removeRow(3);
model->submitAll();

//总之有增删改操作后都需要调用model->submitAll();来真正执行，否则仅仅是数据模型更新了数据，并不会更新到数据库中。

//撤销更改
model->revertAll();
```

## QList、QMap、QHash

Qt内置了一些QList、QMap、QHash相关的类型，可以直接用，不用自己写个长长的类型。

```c++
//qwindowdefs.h
typedef QList<QWidget *> QWidgetList;
typedef QList<QWindow *> QWindowList;
typedef QHash<WId, QWidget *> QWidgetMapper;
typedef QSet<QWidget *> QWidgetSet;

//qmetatype.h
typedef QList<QVariant> QVariantList;
typedef QMap<QString, QVariant> QVariantMap;
typedef QHash<QString, QVariant> QVariantHash;
typedef QList<QByteArray> QByteArrayList;
```

## QLocale

QDateTime可以直接格式化输出星期几周几，Qt6默认按照英文输出比如 ddd = 周二 Tue dddd = 星期二 Tuesday ，此时如果只想永远是中文就需要用到QLocale进行转换。

```c++
//格式化输出受到本地操作系统语言的影响

//英文操作系统
//这样获取到的是Mon到Sun，英文星期的3个字母的缩写。
QDateTime::currentDateTime().toString("ddd");
//这样获取到的是Monday到Sunday，英文星期完整单词。
QDateTime::currentDateTime().toString("dddd");

//中文操作系统
//这样获取到的是周一到周日。
QDateTime::currentDateTime().toString("ddd");
//这样获取到的是星期一到星期日。
QDateTime::currentDateTime().toString("dddd");

//主动指定语言转换
//如果没有指定本地语言则默认采用系统的语言环境。
QLocale locale;
//QLocale locale = QLocale::Chinese;
//QLocale locale = QLocale::English;
//QLocale locale = QLocale::Japanese;C

//下面永远输出中文的周一到周日
locale.toString(QDateTime::currentDateTime(), "ddd");
//下面永远输出中文的星期一到星期日
locale.toString(QDateTime::currentDateTime(), "dddd");
```

## QApplication

Qt中基本上有三大类型的项目，控制台项目对应QCoreApplication、传统QWidget界面程序对应QApplication、quick/qml项目程序对应QGuiApplication。有很多属性的开启需要在main函数的最前面执行才有效果，比如开启高分屏支持、设置opengl模式等。不同类型的项目需要对应的QApplication。

```C++
//如果是控制台程序则下面的QApplication换成QCoreApplication
//如果是quick/qml程序则下面的QApplication换成QGuiApplication
int main(int argc, char *argv[])
{
    //可以用下面这行测试Qt自带的输入法 qtvirtualkeyboard
    qputenv("QT_IM_MODULE", QByteArray("qtvirtualkeyboard"));
    
    //设置不应用操作系统设置比如字体
    QApplication::setDesktopSettingsAware(false);

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    //设置高分屏缩放舍入策略
    QApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::Floor);
#endif
#if (QT_VERSION > QT_VERSION_CHECK(5,6,0))
    //设置启用高分屏缩放支持
    //要注意开启后计算到的控件或界面宽度高度可能都不对,全部需要用缩放比例运算下
    QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
    //设置启用高分屏图片支持
    QApplication::setAttribute(Qt::AA_UseHighDpiPixmaps);
#endif
#if (QT_VERSION > QT_VERSION_CHECK(5,4,0))
    //设置opengl模式 AA_UseDesktopOpenGL(默认) AA_UseOpenGLES AA_UseSoftwareOpenGL
    //在一些很旧的设备上或者对opengl支持很低的设备上需要使用AA_UseOpenGLES表示禁用硬件加速
    //如果开启的是AA_UseOpenGLES则无法使用硬件加速比如ffmpeg的dxva2
    //QApplication::setAttribute(Qt::AA_UseOpenGLES);
    //设置opengl共享上下文
    QApplication::setAttribute(Qt::AA_ShareOpenGLContexts);
#endif

    QApplication a(argc, argv);
    QWidget w;
    w.show();
    return a.exec();
}
```

## QRandomGenerator

Qt5.10以后提供了新的类 QRandomGenerator QRandomGenerator64 管理随机数，使用更方便，尤其是取某个区间的随机数。

```C++
//早期处理办法 先初始化随机数种子然后取随机数
qsrand(QTime::currentTime().msec());
//取 0-10 之间的随机数
qrand() % 10;
//取 0-1 之间的浮点数
qrand() / double(RAND_MAX);

//新版处理办法 支持5.10以后的所有版本包括qt6
QRandomGenerator::global()->bounded(10);      //生成一个0和10之间的整数
QRandomGenerator::global()->bounded(10.123);  //生成一个0和10.123之间的浮点数
QRandomGenerator::global()->bounded(10, 15);  //生成一个10和15之间的整数

//兼容qt4-qt6及以后所有版本的方法 就是用标准c++的随机数函数
srand(QTime::currentTime().msec());
rand() % 10;
rand() / double(RAND_MAX);

//通用公式 a是起始值,n是整数的范围
int value = a + rand() % n;
//(min, max)的随机数
int value = min + 1 + (rand() % (max - min - 1));
//(min, max]的随机数
int value = min + 1 + (rand() % (max - min + 0));
//[min, max)的随机数
int value = min + 0 + (rand() % (max - min + 0));
//[min, max]的随机数
int value = min + 0 + (rand() % (max - min + 1));

//如果在线程中取随机数，线程启动的时间几乎一样，很可能出现取到的随机数一样的问题，就算设置随机数为当前时间啥的也没用，电脑太快很可能还是一样的时间，同一个毫秒。
//取巧办法就是在run函数之前最前面将当前线程的id作为种子设置。时间不可靠，线程的id才是唯一的。
//切记 void * 转换到数值必须用 long long，在32位是可以int但是在64位必须long，确保万一直接用quint64最大
srand((long long)currentThreadId());
qrand((long long)currentThreadId());
```

## QString

QString的replace函数会改变原字符串，切记，他在返回替换后的新字符串的同时也会改变原字符串

QString内置了很多转换函数，比如可以调用toDouble转为double数据，但是当你转完并打印的时候你会发现精确少了，只剩下三位了，其实原始数据还是完整的精确度的，只是打印的时候优化成了三位，如果要保证完整的精确度，可以调用 qSetRealNumberPrecision 函数设置精确度位数即可。

```
QString s1, s2;
s1 = "666.5567124";
s2.setNum(888.5632123, 'f', 7);
qDebug() << qSetRealNumberPrecision(10) << s1.toDouble() << s2.toDouble();
```



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

# 升级到Qt6

1. 源码中的double数据类型全部换成了qreal，和Qt内部数据类型高度一致和统一。

1. QFontMetricsF 中的 fm.width() 换成 fm.horizontalAdvance() ，从5.11开始用新函数。

1. QApplication::desktop()废弃了， 换成了 QApplication::primaryScreen()。

```
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

