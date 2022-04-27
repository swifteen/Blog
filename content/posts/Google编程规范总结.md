+++
title = "Google编程规范总结"
description = ""
tags = [
    "HUGO",
    "blog",
]
date = "2022-04-17"
categories = [
    "工具",
]
menu = "main"

weight= 1

+++
# 版本说明

* Google编程规范总结

| 日期     | 版本 | 修改内容 |
| -------- | ---- | -------- |
| 20211029 | V0.1 | 创建     |

# 目的

使代码易于管理的方法之一是加强代码一致性. 让任何程序员都可以快速读懂你的代码这点非常重要.

项目主页:

- [Google Style Guide](https://google.github.io/styleguide/cppguide.html)
- [Google 开源项目风格指南 - 中文版](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/)

# 代码规范

## 头文件

### 前置声明

1. 尽可能地避免使用前置声明。使用 #include 包含需要的头文件即可

2. 前置声明的类是不完全类型（incomplete type），我们只能定义指向该类型的指针或引用，或者声明（但不能定义）以不完全类型作为参数或者返回类型的函数。毕竟编译器不知道不完全类型的定义，我们不能创建其类的任何对象，也不能声明成类内部的数据成员。

   > If a class only appears in the header as a pointer or reference, then a forward declaration is sufficient

也就是说，类在头文件中，只是以指针或者引用的形式存在时，使用前置声明就够了。但原则上还是尽量避免使用前置声明

### 内联函数

1. 只有当函数只有 10 行甚至更少时才将其定义为内联函数.

2. 内联那些包含循环或 switch 语句的函数常常是得不偿失

3. 有些函数即使声明为内联的也不一定会被编译器内联， 虚函数和递归函数就不会被正常内联

4. 类内部的函数一般会自动内联。所以某函数一旦不需要内联，其定义就不要再放在头文件里，而是放到对应的 .cc 文件里

###  #include 的路径及顺序

dir2/foo2.h (优先位置, 详情如下)，让别人的头文件先出错，避免首先怀疑自己的头文件出现错误

1. C 系统文件 

2. C++ 系统文件 

3. 其他库的 .h 文件 

4. 本项目内 .h 文件

在 #include 中插入空行以分割相关头文件, C 库, C++ 库, 其他库的 .h 和本项目内的 .h 是个好习惯

```c++
#include "foo/public/fooserver.h" // 优先位置

#include <sys/types.h>//C 系统文件
#include <unistd.h>

#include <hash_map>//C++ 系统文件 
#include <vector>

#include "base/basictypes.h"//其他库的 .h 文件
#include "base/commandlineflags.h"

#include "foo/public/bar.h" //本项目内 .h 文件
```

## 作用域

### 命名空间

1. 不应该使用 using 指示 引入整个命名空间的标识符号

2. 不要在命名空间 std 内声明任何东西

3. 不要在头文件中使用 命名空间别名

4. 禁止用内联命名空间 

### 匿名命名空间和静态变量

https://roachsinai.github.io/Cpp-learning-notes/declaration/namespace/anonymous.html

1. 在 .cc 文件中定义一个不需要被外部引用的变量时，可以将它们放在匿名命名空间或声明为 static

2. 单纯为了封装若干不共享任何静态数据的静态成员函数而创建类, 不如使用命名空间

   ```c++
   //推荐
   namespace myproject {
   namespace foo_bar {
   void Function1();
   void Function2();
   }  // namespace foo_bar
   }  // namespace myproject
   ```

   ```c++
   //不推荐
   namespace myproject {
   class FooBar {
    public:
     static void Function1();
     static void Function2();
   };
   }  // namespace myproject
   ```

### 非成员函数、静态成员函数和全局函数

如果你必须定义非成员函数, 又只是在 .cc 文件中使用它, 可使用匿名 2.1. 命名空间 或 static 链接关键字 (如 static int Foo() {...}) 限定其作用域.

### 局部变量

1. C++ 允许在函数的任何位置声明变量. 我们提倡在尽可能小的作用域中声明变量, 离第一次使用越近越好. 这使得代码浏览者更容易定位变量声明的位置, 了解变量的类型和初始值. 特别是，应使用初始化的方式替代声明再赋值

   ```c++
   int j = g(); // 好——初始化时声明
   
   int i;
   i = f(); // 坏——初始化和声明分离
   ```

2. 局部变量在声明的同时进行显式值初始化，比起隐式初始化再赋值的两步过程要高效，同时也贯彻了计算机体系结构重要的概念「局部性（locality）」

3. 注意别在循环犯大量构造和析构的低级错误
   ```c++
    Foo f;                      // 构造函数和析构函数只调用 1 次
    for (int i = 0; i < 1000000; ++i) {
        f.DoSomething(i);
    }
   ```
   
   ```c++
   // 低效的实现
   for (int i = 0; i < 1000000; ++i) {
       Foo f;                  // 构造函数和析构函数分别调用 1000000 次!
       f.DoSomething(i);
   }  
   ```

### 静态全局变量Static and Global Variables

1. 不推荐给非局部变量动态分配空间，通常禁止这样做

> Dynamic initialization of nonlocal variables is discouraged, and in general it is forbidden.
>
2. 允许给静态局部变量动态分配空间
> Dynamic initialization of static local variables is allowed (and common).

3. 当一个初始化指向另一个具有静态存储期限的变量时，有可能导致一个对象在其生命周期开始前（或在其生命周期结束后）被访问。此外，当一个程序启动的线程在退出时没有加入，如果对象的析构器已经运行，这些线程可能试图在其生命周期结束后访问对象。

> When one initialization refers to another variable with static storage duration, it is possible that this causes an object to be accessed before its lifetime has begun (or after its lifetime has ended). Moreover, when a program starts threads that are not joined at exit, those threads may attempt to access objects after their lifetime has ended if their destructor has already run.

4.  静态生存周期的对象，即包括了全局变量，静态变量，静态类成员变量和函数静态变量，都必须是原生数据类型 (POD : Plain Old Data): 即 int, char 和 float, 以及 POD 类型的指针、数组和结构体

5. 只允许 POD 类型的静态变量，即完全禁用 vector (使用 C 数组替代) 和 string (使用 const char [])来作为静态变量。

6. 多线程中的全局变量 (含静态成员变量) 不要使用 class 类型 (含 STL 容器), 避免不明确行为导致的 bug

7. 禁止使用类的 [静态储存周期](http://zh.cppreference.com/w/cpp/language/storage_duration#.E5.AD.98.E5.82.A8.E6.9C.9F) 变量：由于构造和析构函数调用顺序的不确定性，它们会导致难以发现的 bug 。

8. 尽量不用全局函数和全局变量, 考虑作用域和命名空间限制, 尽量单独形成编译单元;

>同一个编译单元内是明确的，静态初始化优先于动态初始化，初始化顺序按照声明顺序进行，销毁则逆序。不同的编译单元之间初始化和销毁顺序属于未明确行为 (unspecified behaviour)。

## 类

### 构造函数

1. 构造函数不得调用虚函数。 如果在构造函数内调用了自身的虚函数, 这类调用是不会重定向到子类的虚函数实现. 即使当前没有子类化实现, 将来仍是隐患

2. 不要在无法报出错误时进行可能失败的初始化。如果执行失败, 会得到一个初始化失败的对象, 这个对象有可能进入不正常的状态, 必须使用 bool IsValid() 或类似这样的机制才能检查出来, 然而这是一个十分容易被疏忽的方法

### 隐式类型转换

1. 转换运算符和单参数构造函数, 请使用 explicit 关键字

> This keyword is a declaration specifier that can only be applied to in-class constructor declarations . An explicit constructor cannot take part in implicit conversions. It can only be used to explicitly construct an object 。

 https://www.jianshu.com/p/af8034ec0e7a

2.  为避免隐式转换, 需将单参数构造函数声明为 explicit;

> explicit的作用是用来声明类构造函数是显示调用的，而非隐式调用，所以只用于修饰单参构造函数。因为无参构造函数和多参构造函数本身就是显示调用的

### 可拷贝类型和可移动类型

1. 拷贝 / 移动构造函数在某些情况下会被编译器隐式调用. 例如, 通过传值的方式传递对象.

2. 如果你的类不需要拷贝 / 移动操作, 请显式地通过在 public 域中使用 = delete 或其他手段禁用之

```c++
//MyClass is neither copyable nor movable. 
MyClass(const MyClass&) = delete; 
MyClass& operator=(const MyClass&) = delete;
```

3. 如果你的基类需要可复制属性, 请提供一个 public virtual Clone() 和一个 protected 的拷贝构造函数以供派生类实现.

4. 为避免拷贝构造函数, 赋值操作的滥用和编译器自动生成, 可将其声明为 private 且无需实现;

   ```c++
   private:
   	MyClass(const MyClass &other);
   	MyClass &operator = (const MyClass &other);
   ```

> If a class contains pointer-type member variables, the copy constructor and the Assignment operator should either be explicitly implemented or declared as "private" (and not implemented). This ensures that the compiler does not generate any defaults for these methods that produce a "flat" copy of such objects and thus possibly create memory problems.

对于有指针成员变量的类，拷贝构造和赋值构造必须显式实现或者声明为私有

### 结构体 VS. 类

仅当只有数据成员时使用 struct, 其它一概使用 class

### 继承

1. 如果使用继承的话, 定义为 public 继承.

2. 必要的话, 析构函数声明为 virtual. 如果你的类有虚函数, 则析构函数也应该为虚函数.

3. 子类重载的虚函数也要声明 virtual 关键字, 虽然编译器允许不这样做

4. 如果 Bar 的确 “是一种” Foo, Bar 才能继承 Foo.

5. 只有当所有父类除第一个外都是 纯接口类 时, 才允许使用多重继承. 为确保它们是纯接口, 这些类必须以 Interface 为后缀.

### 存取控制

将所有数据成员声明为 private, 除非是 static const 类型成员

### 声明顺序

1. 将相似的声明放在一起, 将 public 部分放在最前.

2. 类定义一般应以 public: 开始, 后跟 protected:, 最后是 private:

3. 建议以如下的顺序: 类型 (包括 typedef, using 和嵌套的结构体与类), 常量, 工厂函数, 构造函数, 赋值运算符, 析构函数, 其它函数, 数据成员

   ```c++
   MyClass
   {
   public:
      typedef xxx
      {
          int x,
      }yyy;
      
       const int kMyValue = 100;
   public:    
       MyClass();
       MyClass(const MyClass&);
       ~MyClass();
   protected:
       void func_a();
   private:  
       void func_b();
   private:   
       int m_iValue;
   };
   ```

4. 不要将大段的函数定义内联在类定义中. 通常，只有那些普通的, 或性能关键且短小的函数可以内联在类定义中

## 函数

### 参数顺序

1. 将所有的输入参数置于输出参数之前

2. 在加入新参数时不要因为它们是新参数就置于参数列表最后, 而是仍然要按照前述的规则, 即将新的输入参数也置于输出参数之前.

### 引用参数

1. 定义引用参数可以防止出现 (*pval)++ 这样丑陋的代码. 引用参数对于拷贝构造函数这样的应用也是必需的. 同时也更明确地不接受空指针

2. 大多时候输入形参往往是 const T&. 若用 const T* 则说明输入另有处理. 所以若要使用 const T*, 则应给出相应的理由, 否则会使得读者感到迷惑

### 函数重载

1. 如果打算重载一个函数, 可以试试改在函数名里加上参数信息. 例如, 用 AppendString() 和 AppendInt() 等, 而不是一口气重载多个 Append().

### 缺省参数

1. 对于虚函数, 不允许使用缺省参数, 因为在虚函数中缺省参数不一定能正常工作.

2. 我们不允许使用缺省函数参数，少数极端情况除外。尽可能改用函数重载。

## 智能指针

1. scoped_ptr 和 auto_ptr 已过时. 现在是 shared_ptr 和 uniqued_ptr 的天下了。不要使用 std::auto_ptr, 使用 std::unique_ptr 代替它

2. 如果必须使用动态分配, 那么更倾向于将所有权保持在分配者手中. 如果其他地方要使用这个对象, 最好传递它的拷贝, 或者传递一个不用改变所有权的指针或引用. 倾向于使用 std::unique_ptr 来明确所有权传递

  ```
   std::unique_ptr<Foo> FooFactory();
   void FooConsumer(std::unique_ptr<Foo> ptr);
  ```

3. 如果确实要使用共享所有权, 建议于使用 std::shared_ptr

## 其它C++特性

### 友元

如果你只允许另一个类访问该类的私有成员时

### 异常

1. Google 现有的大多数 C++ 代码都没有异常处理, 引入带有异常处理的新代码相当困难

2. 很多 C++ 书籍上都提到当构造失败时只有异常可以处理

3. https://www.zhihu.com/question/22889420

### 类型转换

C++ 采用了有别于 C 的类型转换机制, 对转换操作进行归类

1. 用 static_cast 替代 C 风格的值转换, 或某个类指针需要明确的向上转换为父类指针时.

2. 用 const_cast 去掉 const 限定符.

3. 用 reinterpret_cast 指针类型和整型或其它指针之间进行不安全的相互转换.

### 流

1. 不要使用流, 除非是日志接口需要. 使用 printf 之类的代替.

2. 使用流还有很多利弊, 但代码一致性胜过一切. 不要在代码中使用流.

### 前置自增和自减

对简单数值 (非对象), 两种都无所谓. 对迭代器和模板类型, 使用前置自增 (自减).

### const 用法

1. 关键字 mutable 可以使用, 但是在多线程中是不安全的, 使用时首先要考虑线程安全.

```c++
const int* foo; //提倡但不强制
int const *foo;
```

2. 注意初始化 const 对象时，必须在初始化的同时值初始化

### 整型

int 与 unsigned int 运算时，前者被提升为 unsigned int 而有可能溢出

```c++
//无限循环
for (unsigned int i = foo.Length()-1; i >= 0; --i) ...
```

使用断言来指出变量为非负数, 而不是使用无符号型!不要为了指出数值永不会为负, 而使用无符号类型. 相反, 你应该使用断言来保护数据.

### 预处理宏

1. 宏具有全局作用域

2. 用 # 字符串化, 用 ## 连接

3. 不要在 .h 文件中定义宏

4. 在马上要使用时才进行 #define, 使用后要立即 #undef

### 0, nullptr 和 NULL

1. 整数用 0, 实数用 0.0, 指针用 nullptr 或 NULL, 字符 (串) 用 '\0'.

2. C++11 项目用 nullptr; C++03 项目则用 NULL

### sizeof

尽可能用 sizeof(varname) 代替 sizeof(type)

```c++
//下面代码存在错误，只对data的前4个字节执行了memset
Struct* data = NULL; 
data = new Struct; 
memset(data, 0, sizeof(data));
```

###  auto

1. auto 只能用在局部变量里用。别用在文件作用域变量，命名空间作用域变量和类数据成员里。
2. 永远别列表初始化 auto 变量

3. auto 在涉及迭代器的循环语句里挺常用

## 命名约定

函数命名, 变量命名, 文件命名要有描述性; 少用缩写

### 文件命名

文件名要全部小写, 可以包含下划线 (_)

定义类时文件名一般成对出现, 如 foo_bar.h 和 foo_bar.cc, 对应于类 FooBar

### 类型命名

类型名称的每个单词首字母均大写, 不包含下划线。类, 结构体, 类型定义 (typedef), 枚举, 类型模板参数

```c++
// 类和结构体
class UrlTable { ...
class UrlTableTester { ...
struct UrlTableProperties { ...

// 类型定义
typedef hash_map<UrlTableProperties *, string> PropertiesMap;

// using 别名
using PropertiesMap = hash_map<UrlTableProperties *, string>;

// 枚举
enum UrlTableErrors { ...
```

### 变量命名

变量 (包括函数参数) 和数据成员名一律小写, 单词之间用下划线连接

### 常量命名

声明为 constexpr 或 const 的变量, 或在程序运行期间其值始终保持不变的, 命名时以 “k” 开头, 大小写混合

```c++
const int kDaysInAWeek = 7;
```

### 函数命名

常规函数使用大小写混合, 取值和设值函数则要求与变量名匹配。写作 StartRpc() 而非 StartRPC()。驼峰变量名

### 枚举命名

单独的枚举值应该优先采用 常量 的命名方式

```c++
enum UrlTableErrors 
{ 
    kOK = 0,  
    kErrorOutOfMemory,  
    kErrorMalformedInput, 
};
```

枚举名 UrlTableErrors (以及 AlternateUrlTableErrors) 是类型, 所以要用大小写混合的方式

## 注释

如果类的声明和定义分开了(例如分别放在了 .h 和 .cc 文件中), 此时, 描述类用法的注释应当和接口定义放在一起, 描述类的操作和实现的注释应当和实现放在一起。不要在 .h 和 .cc 之间复制注释, 这样的注释偏离了注释的实际意义.

### 函数声明

1. 函数是否分配了必须由调用者释放的空间.

2. 参数是否可以为空指针

### 实现注释

对于代码中巧妙的, 晦涩的, 有趣的, 重要的地方加以注释.

### 实现技巧

1. 如果某个函数有多个配置选项, 你可以考虑定义一个类或结构体以保存所有的选项, 并传入类或结构体的实例.

```c++
ProductOptions options;
options.set_precision_decimals(7);
options.set_use_cache(ProductOptions::kDontUseCache);
const DecimalNumber product =
    CalculateProduct(values, options, /*completion_callback=*/nullptr);
```

2. 你所提供的注释应当解释代码 为什么 要这么做和代码的目的, 或者最好是让代码自文档化.

```c++
if (!IsAlreadyProcessed(element))
{ 
    Process(element); 
}
```

### TODO 注释

```c++
// TODO(kl@gmail.com): Use a "*" here for concatenation operator.
// TODO(Zeke) change this to use relations.
// TODO(bug 12345): remove the "Last visitors" feature
```

## 格式

###  制表符

不要在代码中使用制表符. 你应该设置编辑器将制表符转为空格.

### 函数声明与定义

1. 左圆括号总是和函数名在同一行.

2. 函数名和左圆括号间永远没有空格

3. 圆括号与参数间没有空格

4. 左大括号总在最后一个参数同一行的末尾处, 不另起新行.

5. 右圆括号和左大括号间总是有一个空格

### 条件语句

if (condition) { // 好 - IF 和 { 都与空格紧邻.

### 预处理指令

预处理指令不要缩进, 从行首开始

```c++
// 好 - 指令从行首开始
  if (lopsided_score) {
#if DISASTER_PENDING      // 正确 - 从行首开始
    DropEverything();
# if NOTIFY               // 非必要 - # 后跟空格
    NotifyClient();
# endif
#endif
    BackToNormal();
  }
```

命名空间内容不缩进

```c++
namespace {

void foo() {  // 正确. 命名空间内没有额外的缩进.
  ...
}

}  /
```

### 垂直留白

 垂直留白越少越好,同一屏可以显示的代码越多, 越容易理解程序的控制流

