# 2017-12-15

##### 条款一，视c++为一个语言联邦。
- c，object-oriented c++ ， template c++， stl
- c++高效编程守则视情况而变化，取决使用c++的哪一部分

##### 条款二，尽量以const，enum，inline代替#define
- define使用的名称之前已被移走，并未进入记号表，所以用一个常量代替宏（const），string对象const往往比char*更加合适，例如consts char* const authorName = "Scott Meyers"可以定义成const std:: autorName(" Scott Meyers")
- define只要被定义，其后的编译过程中有效，没有所谓的private #define，意味着define不能完成封装，enum行为来说比较像define，例如取一个const地址是合法的，但取一个enum地址就不合法，enum可以实现不让指针和引用指向某个常数变量，enum和define不会导致非必要的内存分配。
- define的边际效益，比如
```cpp
int a=5; 
int b=0; 
#define CALL_WITH_MAX(a,b) f((a)>(b)?(a):(b))
CALL_WITH_MAX(++a,b) //a被累加两次
CALL_WITH_MAX(++a,b+10) //a被累加一次
```
a的递增次数不确定，可以用template inline代替define
```cpp
template<typename T>
{
  inline void callwithmax(const T &a, const T &b)  
  {
    f(a>b?a:b);
  }
}
```
总结:
- 对于单纯的变量，最好以const对象或enum替换define
- 对于形似函数的宏，最好改用inline函数代替define

##### 条款三，尽量使用const
- const在星号左面修饰的是指针指向的是常量，在星号右面，修饰的是指针本身
- stl迭代器中
```cpp
const std::vector<int>::iterator iter //修饰的是迭代器本身，等同于T* const
```
```cpp
std::vector<int>::const_iterator //修饰的是迭代器的指向物，等同于const T*
```
- const用于函数的返回值,防止==打成=
- const成员函数，两个成员函数如果只是常量性不同，可以被重载。non-const operator[]返回的是refence，如果不是我们无法通过[]去改变。
- const成员函数不会改变任何non-static成员变量，multable修饰可以让non-static成员变量被const成员函数修改
总结:
- 将某些声明为const可帮助编译器侦测出错误的用法，const可被施加于任何作用域内的对象，函数参数，函数返回类型，成员函数本体。
- 编译器强制实施bitwise constness,但你编写程序时应该使用“概念上的常量性”。
- 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复。

##### 条款四，确定对象被使用前已先被初始化

- 对于无任何成员的内置类型，进行手工初始化。
```cpp
int x=0;
const char *text="A C-style string";
double d;
std::cin>>d;
```
- 确保每一个构造函数都将对象的每一个成员初始化
```cpp
abentry::abentry(const std::string& name, const std::string& address, const std::List<phonenumber>& phones)
:thename(name),
   theaddress(address),
     thephones(phones), 
        numtimesconsulted(0)
{}
//使用成员初值列代替赋值动作，如果成员变量是引用和const一定要初值，不能赋值。
```
- c++对定义与不同的编译单元内的non-local static对象的初始化相对次序并无明确的定义。
```cpp
//互联网上的文件看上去位于本机的程序，象征单一文件系统
class filesystem{
  public:
    ...
    std::size_t numdisks() const;
    ...
}
extern filesystem tfs;
```

```cpp
class directory
{
  public:
    directory(params);
    ....
};
directory::directory(params)
{
  ....
  std::size_t disks = tsf.numdisks();
  ....
}
directory tempdir(params)
//无法确定tfs在tempdir之前被初始化。
```

singletion设计模式解决
```cpp
//将non-local static对象搬到自己的专属函数内，这些函数返回一个引用指向它所含的对象
//然后用户调用这些函数，而不直接指涉这些对象。
class filesystem{....};
filesystem tfs()
{
  static filesystem fs;
  return fs;
}
class directory{...}
directory& tempdir()
{
  static directory td;
  return td;
}
```

总结:
- 对内置型对象进行手工操作初始化，因为c++不保证初始化它们。
- 构造函数最好使用成员初值列，而不要在构造函数本体内使用赋值操作，初值列列出的成员变量，其排列次序应该和它们在class中的声明次序相同。
- 为免除“跨编译单元之初始化次序”问题，轻易local static对象替换non-local static对象。

##### 条款五，了解c++默默编写了哪些函数
- 一个空类会默认生成copy构造函数，一个copy assignment操作符，一个析构函数，一个default构造函数
- 如果已经声明了一个构造函数，编译器将不再为它创建default构造函数，
- 要注意，一般而言，只有代码合法且证明它有意义编译器才会生出operator = （比如&，const就会出现问题）

##### 条款六，若不想使用编译器自动生成的函数，就该明确拒绝
- 为驳回编译器自动提供的功能（例如条款五中），可将相应的成员函数声明为private并且不予实现，使用像uncopyble这样的base class也是一种做法。
```cpp
//例如
class homeforsale{
public:
  ...
private:
  ...
  homeforsale(const homeforsale& );
  homeforsale& operator=(const homeforsale&);
}
```

##### 条款七，为多态基类声明virtual函数
- c++明确指出，当派生类对象经由一个基类对象指针被删除时，而该基类对象带着一个non-virtual析构函数，其执行时的结果是derived成分没被销毁
- 当class不被当做base class的时候，不可以将析构函数作为virtual
- 当class内含有至少一个virtual函数时，才为它声明virtual析构函数（心得）
- 标准string，还有stl等等都是不带virtual（不企图让你将它们当做base class，不要尝试去做）
- 当抽象类没有没有任何pure virtual函数时，可以将析构函数声明为pure virtual函数。
```cpp
class awov{
public:
    virtual ~awov()=0;
}
awov::~awov(){}//必须为其提供一份定义
```
总结:
- 带有多态性质的base classes应该为其声明一个virtual析构函数，如果class带有任何virtual函数，它应该拥有一个virtual析构函数
- classes的设计目的如果不是作为base classes使用，或不是为了具备多态性，就不该声明virtual函数

##### 条款八，别让异常逃离析构函数
- c++并不禁止析构函数吐出异常，但它并不鼓励你那样做
- 可以用以下方法将析构中异常责任转移给用户
```cpp
//例如数据库的关闭
class dbconn{
public:
  void close()
  {
    db.close();
    closed = true;
  }  
 ~dbconn()
  {
    if(!closed)
    {
      try{
        db.close();
      }
      catch(...){
        ...
      }
    }
  }
private:
  DBConnection db;
  bool closed;
}
```
总结:
- 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们或结束程序。
- 如果客户需要对某个操作函数运作期间抛出的异常做出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。

##### 条款九，绝不在构造和析构函数中调用virtual函数
- 在base class构造期间，virtual函数不是virtual函数。
- 可以令derived classes将必要的构造信息向上传递至base class构造函数
```cpp
class transaction{
  explicit transaction(const std::string &loginfo);
  void logtransaction(const std::string &loginfo);
  ...
};
transaction::transaction(const std::string &loginfo)
{
  ...
  logtransaction(loginfo)
}
class buytransaction::public transaction{
public:
  buytransaction(parameters)
    : transaction(createlogstring(parameters))
    {...}
...
private:
  static std::string createlogstring(parameters);
}
```
总结：
- 在构造和析构期间不要调用virtual函数，因为这类调用从不降至derived class。

##### 条款十，令operator=返回一个refrence to *this
- 赋值操作符必须返回一个reference指向操作符的左侧实参。
```cpp
class widget{
public:
  widget& operator= (const widget& rhs)
  {
    ...
    return rhs;
  }
 ...
};
```
- 同时也适用于所有赋值相关操作

总结:
- 令赋值操作符返回一个refrence to *this

##### 条款十一，在operator=中处理“自我赋值” 
- 防止自我赋值，出传统做法是加证同测试，达到自我赋值的验证目的。
```cpp
widget& widget::operator=(const widget& rhs)
{
  if(this == &rhs)return *this;//证同测试
  
  delete pd;
  pd = new bitmap(*rhs.pd);
  return thisl
}
```
- 其实让operator =具备异常安全性往往自动获得自我赋值安全的回报
- 可以使用copy and swap技术，就是拷贝一次后交换（ps:作者比较忧虑这种做法）

总结:
- 确保当前对象自我赋值时operator=有良好的行为，其中技术包括比较来源对象和目标对象地址，精心周到的语句顺序，以及copy-and-swap。
- 确定任何函数如果操作一个以上的对象，而其中多个对象是同一个对象时，其行为仍然正确。

##### 条款十二，复制对象时勿忘其每一个成分。
- 如果为class添加一个成员变量时，要同时修改copying函数，构造函数，operator=函数。
- 应该让derived class的copying函数调用相应的base class函数（初始化列表）。
- 不该令copy assignment操作符调用copy构造函数，相反也一样

总结:
- copying函数应该确保复制对象内的所有成员变量及所有base class成分。
- 不要尝试copying函数实现另一个copying函数，应该将共同技能放进第三个函数中，并由两个copying函数共同调用。

##### 条款十三，以对象管理资源
- 因为抛出异常或者提前return的情况可能会使申请的资源无法释放
```cpp
void f()
{
  investment *pinv = createinvestment();//工厂模式
  ...//在这里出现异常或者return的话，资源无法释放
  delete pinv;
}
```

- 为避免这种情况，可以使用智能指针，。
- 以对象管理资源的两个思想，获得资源后立刻放进管理对象（构造函数），管理对象运用析构函数确保资源被释放。
- auto_ptr和shared_ptr两者在析构函数内是delete，而不是delete[]。
总结:
- 为防止资源泄露，请使用RAII对象，它们在构造函数中获得资源并在析构函数中释放资源。
- 两个常被使用的RAII classes分别是tr1::shared_ptr和auto_ptr.前者通常是较佳的选择，因为其copy行为比较直观，若选择auto_ptr，复制动作会使它（被复制物物）指向null

##### 条款十四，在资源管理类中小心copying行为
- 当一个RAII class 对象被复制时我们可以:
- 我们可以禁止复制，将copying声明为private。
- 对底层资源用引用计数，可以使用shared_ptr。

```cpp
class Lock
{
public:
  explicit lock(mutex *pm)
      :mutexptr(pm, unlock)
  {
    lock(mutexptr.get());
  }
private:
  stdd::tr1::shared_ptr<mutex>mutexptr;
}
```

- 复制底部资源，即进行深度拷贝。
- 转移底部资源，可以看做auto_ptr的行为

总结:
- 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
- 普遍而唱见的RAII class copying行为是:抑制copying，施行引用计数法。不过其他行为也可能被实现。

##### 条款十五，在资源管理器类中提供对原始资源的访问。
- tr1::shared_ptr和auto_ptr都提供了一个get成员函数，可以返回其中的内部原始指针，条款十三中的工厂模式，必须同样返回类型也要是RAII，所以我们可以这样调用
```cpp
//显式转换
std::tr1::shared_ptr<investment>pint(createinvestment);
int days = datsheld(pinv.get());
```

- 这两个智能指针也重载了->和*操作符，可以隐式转换至底部原始指针。
- 有时候必须取得RAII对象内的原始资源，做法是提供一个隐式转换函数。
```cpp
//显式转换
class font{
public:
  ...
  fonthandle get() const{ return f; }
}
```

```cpp
//隐式转换
class font{
public:
  ...
  operator fonthandle() const
  { return f; }
}
```

- 显式转换的优点是防止被误用，而隐式转换的优点是用起来比较自然

总结:
- apis往往要求访问原始资源，所以每一个RAII class应该提供一个“取得其所管理之资源”的方法。
- 对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。

##### 条款十六，成对使用new和delete时要采取相同的形式。
- 如果调用new，则必须对应调用delete，如果调用new[]，则必须对应调用delete[]。
- 注意一下用法

```cpp
typedef std::string addresslines[4];

std::string *pal = new addresslines;

delete pal;//行为未定义
delete pal[];//很好
```

总结:
- 如果你在new表达式中使用[]。必须在相应的delete中也使用[]，如果在new表达式中没有使用[]，一定不要在相应的delete中使用[]。

##### 条款十七，以独立语句将newed对象置入智能指针
- c++编译器调用参数时顺序是不固定的。

```cpp
process (std::str1::shared_ptr<widget>(new widget), priority())
/*
如果是下列执行顺序
1，调用new
2，调用priority
3，调用智能指针构造函数
如果第二步出现问题可能会导致资源泄露
*/
```

- 避免上述问题很简单，使用分离语句，在单独语句内调用std::str1::shared_ptr<widget>(new widget)即可

总结:
- 以独立语句将newed对象存储于智能指针内，如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄露。

##### 条款十八，让接口容易被正确使用，不易被误用
- 导入外覆类型
```cpp
struct day{
  explicit day(int d):val(d){ }
  int val;
}
struct month{
  explicit month(int m):val(m){ }
  int val;
}
struct year{
  explicit year(int y):val(y){ }
  int val;
}
class data{
public:
  data(const month&m const day&d, const year&y)    
};
data d(month(3),day(30),year(1995));
```

- 尽量令你的type行为与内置的type一样（例如STL接口十分一致）。
- 令factory返回智能指针shared_ptr
- share_ptr比正常的指针大且慢，而且使用辅助动态内存。
- DLL问题即对象在动态链接程序库中被new创建，却在另一个DLL内被delete。shared_ptr会追踪记录引用次数，不会出现这种问题。
- 如果我们期许将一个工厂返回的指针放到另一个函数去删除，那么应该将另一个函数直接绑定。
```cpp
std::tr1::shared_ptr<investment>
  pinv(static_cast<investment*>(0)),
    getridofinvestment);
```

总结:
- 好的接口很容易被正确使用，不容易被误用，你应该在你所有接口中努力达成这些性质
- “促进正确使用”的方法包括接口的一致性，以及与内置类型的行为兼容
- “阻止误用”的办法包括建立心累系女，限制类型上的操作，束缚对象值，以及消除客户资源管理的责任。
- tr1::shared_ptr支持定制删除器，可以防范DLL问题，可被用来自动解除互斥锁等等。

##### 条款十九，设计class犹如type

- 新type的对象应该如何被创建和销毁？（第八章）
- 初始化和赋值决定了构造函数和赋值操作符的行为，不要弄混对象的初始化和对象的赋值，因为它们应该在不同的函数调用（条款4）
- copy构造函数用来定义一个type的pass-by-value该如何实现。
- 对class的成员的变量而言，通常只有某些数值集是有效的，决定了成员函数必须要进行错误检查工作。
- 如果继承自某些既有的classes，那么就会受到那些classes的设计的束缚。
- 如果允许类型t1对象被隐式转换成t2对象，就必须在t1内写一个类型转换函数或在t2中写一个可被单一实参调用的构造函数。
- 注意哪些声明为member函数和non-member函数。
- 注意应该驳回哪些标准函数，应把它声明为private。
- 注意哪个成员声明为public，private，protect。
- 如果定义非一个新type而是一整个type家族，那就应该定义一个新的class template
- 如果只是为了既有的class添加机能，说不定多定义一个non-member或empletes，更能达到目标

总结:
- class的设计就是type的设计，在定义一个新type之前，请确定已经考虑过本条所覆盖的所有内容。

##### 条款二十，宁以pass-by-reference-to-const替换pass-by-value
- pass-by-value会通过copy构造函数创建副本，并且函数结束后会析构，而pass-by-reference-to-const，reference底层是通过指针实现的，因此往往通常意味着真正传递的指针，而通过const不会担心值被改变，效率会高很多
- 以by reference方式传递也可以避免对象切割问题，直接上例子

```cpp
class window{
public:
  std::string name()const;
  virtual void display()const;
}
class windowwithscrollbars::public window{
public:
  ...
  virtual void display()const;
}

void PrintNameAndDisplay(window w)//参数可能被切割。
{
}
windowwithscrollbars wwsb;
PrintNameAndDisplay(wwsb);
//参数wwsb会被构造成一个window对象，继承下来的信息会被全部切除，
//而pass by reference就不会出现这种问题。
```

- 内置类型应该pass by value

总结
- 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可以避免切割问题
- 以上规则并不适用于内置类型，以及STL迭代器和函数对象，对他们而言pass by value往往比较恰当。

##### 条款二十一：必须返回对象时，别妄想返回其reference
- 绝对不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一
个local static对象而有可能同时需要对个这样的对象。

总结:
- 这个我懂

##### 条款二十三，宁以non-member，non-friend替换member函数
- 尽量用non-member替换member函数，例如：

```cpp
class webborwser{
public:
  ...
  void clearcachhe();
  void clearhistory();
  void removecookies();
  void cleareverything();//调用上面三个函数
  ...
}

void cleareverything(webborwser &wb)
{
  wb.clearcache();
  wb.clearhistory();
  wb.removecookies();
}
```

- 上述做法可以降低耦合度，增加可改变程度。
- 越多的东西被封装，我们改变那些东西的能力也就越大。
- 越多的函数可以访问成员变量，那么封装性就越差,所以non-member,non-friend函数封装性更好，因为它们没有增加能够访问class中private成分的函数的数量
- 让函数成为class的non-member并不意味它不可以是一个class的member。c++做法是让non-member函数和class位于同一个namespace里。

```cpp
namespace webbrowserstuff{
  class webbrowser{...}
  void clearborwser(webbrowser &wb);
}
```

- friend函数对class private成员的访问权利和member函数相同。
- 标准程序库并不是拥有单一，整体，庞大的<c++StandardLibrary>头文件并在其中内涵std命名空间内的每一样东西，而是有数十个头文件，每个头文件生命std的某些技能，我们设计也当如此，如下：

```cpp
namespace webbrowserstuff{
  class webbrowser{...};      //核心机能，几乎所有客户都需要
    ...           //non-member函数 
}
namespace webbrowserstuff{
    ...//书签相关便利函数
}
namespace webbrowserstuff{
      ...cookie相关便利函数
}
```

总结:
- 宁可拿non-member，non-friend函数替换member函数，这样做可以增加封装性，包裹弹性和机能扩充。

##### 条款二十四，若所有参数皆需类型转换，请为此采用non-member函数
- 在建立数值类型时，应当令class支持隐式类型转换。
- 例如虚数运算，我们在重载乘法时，应当考虑将两个参数全部支持隐式转换，而不能支持一个，例如乘法交换律。

```cpp
result = 2*onehalf; //无法通过编译
```

- 让operation成为一个non-member函数就可以允许编译器在每一个实参上执行隐式类型转换。

```cpp
class rational{
  ...
};
const rational operator*(const rational& lhs, const rational& rhs){
  ...
}
rational result
rational onefourth(1,4);
result = 2*onefourth      //可通过编译

```

- 不能只因为函数不该成为member，就自动让它成为friend。

总结:
- 如果你需要为某个函数的所有参数进行类型转化，那么这个函数必须是non-member。

##### 条款二十五：考虑写出一个不抛异常的swap函数
- 标准库swap三次对象赋值。
- 置换下列代码，我们只需置换其pimpl指针，swap效率低下。

```cpp
class widgetimpl{
public:
  ...
private:
  int a,b,c;
  std::vector<double>v
}
class widget{
public:
  widget(const widget& rhs);
  widget& operator=(const widget& rhs)
  {
    ...
    *pimpl = *(rhs.pimpl);
    ...
  }
  ...
private:
  widgetimpl* pimpl;
}
```

- 可以令widget声明一个名为swap的public成员函数做真正的置换工作，然后std::swap特化。

```cpp
class widget{
public:
  ...
  void swap(widget& other)
  {
    using std::swap;
    swap(pimpl,other, pimpl);
  }
  ...
};
namespace std{
  template<>         //修订后的swap特化版本
  void swap<widget>(widget& a, widget& b)     
  {
    a.swap(b);
  }
}
```

- c++只允许对class templates偏特化，而function templates身上偏特化是行不通的。 如果打算偏特化一个funciont templates时，通常做法是简单的为它添加一个重载模板。
- 请不要添加任何新东西到std里头
- 如果widget和widgetimpl都是class templates，可以声明一个non-member swap调用memberswap，不将non-member swap声明为std::swap的特化版本或重载版本。

```cpp
namespace widgetstuff{
  temlate<typename T>
  class widget{...};
 ...
  template<typename T>
  void swap(widget<T>&a,
                   widget<T>&b)
  {
    a.swap(b);
  }
}
```

- c++的名称查找法则确保将找到global作用域或T所在之命名空间内的任何T专属的swap，如果没有，则用std内的swap。
- 如果swap效率不足，试着做一下事情：

```
1，提供一个public swap成员函数，让它高效的置换你的类型的两个对象值，这个函数绝对不能抛出异常。
2，在你的class或template所在的命名空间内提供一个non-member swap，并令它调用上述swap函数。
3，如果你正在编写一个class，为你的class特化std::swap。并令它调用你的swap函数
```

- 如果调用swap，确定包含一个using声明式，以便让std::swap在你的函数内曝光可见。让后不加任何修饰符，直接swap。

总结:
- 当std::swap对你的类型效率不高时，提供一个swap函数，并确定这个函数不抛出异常。
- 如果你提供一个member swap，也该提供一个non-member swap用来调用前者，对classes（非templates），也请特化std::swap。
- 调用swap时应针对std::swap使用using声明式，然后调用swap并不带任何命名空间修饰
- 为用户类型进行std templates全特化是好的，但千万不要尝试在std内加入对std而言全新的东西

##### 条款二十六，尽可能延后变量定义式的出现时间
- 定义一个不使用的变量，可能会造成不必要的构造和析构成本。

```cpp
std::string encryptpassword(const std::string &password)
{
  using namespace std;
  string encrypted;
  if(password.length()<minimumpasswordlength)
  {
    throw logic_error("password is too short");
  }
  ...
  return encrypted;
}
//上述代码如果丢出异常，那么encrypted就没有被使用，仍要付出构造和析构成本
```

```cpp
std::string encryptpassword(const std::string &password)
{
  using namespace std;
  if(password.length()<minimumpasswordlength)
  {
    throw logic_error("password is too short");
  }
  string encrypted(password);//初始化
  ...
  return encrypted;
}
//应当这么做
```

- 你不只应该延后变量的定义，直到非得使用该变量的前一刻为止。
- 如果在循环内有两种做法，A，1构造+1个析构+n个赋值操作，B，n个构造+n个析构，第一个是在循环外，第二个是在循环内，当n很大时A好，否则B好。

总结:
- 尽可能的延后变量定义式出现的时间，这样做可增加程序的清晰度并改善程序效率。


##### 条款二十七，尽量少做转型动作
- c++提供四种类型的新式转型

```
const_cast<T>(expression)，通常被用来将对象的常量性移除。它也是唯一有此能力的c++style转型操作符
dynamic_cast<T>(expression)，主要用来执行安全向下转型。
reinterpret_cast<T>(expression)，执行低级转型，例如pointer to int转型为int。
static_cast<T>(expression)，强迫隐式转型。
```

- 我们唯一使用旧型转型的时机是，当我要调用一个explicit构造函数将一个对象传递给一个函数时，例如:

```cpp
class widget{
  public:
    explicit widget(int size);
    ...
};
void doSomeWork(const widget& W);
doSomeWork(widget(15));
```

- 任何一个类型转换往往会使编译器编译出运行期间的代码。
- 许多应用框架都要求derived classes内的virtual函数代码的第一个动作就先调用bass class的函数。但是会有问题

```cpp
class window{
public:
  virtual void onresize(){...}
  ...
class specialwindow:public window{
public:
  virtual void onresize(){
    static_cast<window>(*this).onsize();//这里调用的并不是对象身上的函数，而是稍早转型动作所建立的一个*this对象之base class成分的暂时副本身上的onresize 
    ...
  }
  ...
};
```

- 如果想调用base class版本的onresize函数，请这么做：

```cpp
class specialwindow:public window{
public:
  virtual void onresize(){
    window::onresize();
  }
}
```

- dynamic_cast的许多实现版本执行速度很慢，在注重效率时应注意
- 当认定为derived class 对象身上执行derived class操作函数，但你的手上却只有一个指向base的pointer或refrence，可能想到需要dynamic_cast，有两种方法可以避免这种问题

```cpp
/*
使用容器并在内存中直接指向derived class对象指针
*/
typedef std::vector<std::tr1::shared_ptr<specialwindow*> >vpsw;
vpsw winptrs;
...
for(vpsw::iterator iter = winptrs.begin();iter!=winptrs.end();++iter)
  (*iter)->blink();
```

```cpp
/*
通过base class接口处理所有可能之各种派生类，即在base class内提供virtual函数做你想对window派生类做的事
*/
class window{
public:
  virtual void blink(){}
}
class specialwindow::public window{
public:
  virtual void blink(){...};
  ...
};
typedef std::vector<std::tr1::shared_ptr<specialwindow*> >vpsw;
vpsw winptrs;
...
for(vpsw::iterator iter = winptrs.begin();iter!=winptrs.end();++iter)
  (*iter)->blink();
```

- 应当避免一连串的dynamic_cast
- 优良的c++代码很少使用转型，但是却无法摆脱它们

总结:
- 如果可以，尽量避免转型，特别时在注重效率的代码中避免dynamic_casts。如果有个设计需要转型，试着看无需转型的设计代替。
- 如果转型是必要的，试着将它隐藏于某个函数背后，客户随后可以调用该函数而不需将转型放进他们自己的代码内
- 宁可使用c++style新式，不要使用旧式。前者很容易辨别出来，而且也比较有着分门别类的职掌。

##### 条款二十八，避免返回handles指向对象内部成分
- 成员变量的封装性最多只等于返回其reference的函数的访问级别
- 如果const成员函数传出一个reference，后者所指数据与对象自身有关联，而它又被存储于对象之外。
- 返回一个对象内部数据的handle，随之带来的是降低对象封装性，还有会导致dangling（即指向的对象已经被虚构）

总结：
- 避免返回handles指向内部对象，遵守这条可增加封装性，帮助const成员函数行为像个const，并将发生虚吊的可能性降到最低

##### 条款二十九，为异常安全而努力是值得的

- 异常安全有两个条件，不泄露任何资源，不允许任何数据败坏。

```cpp
class prettymenu{
public:
  ...
  void changebackground(std::iostream imgsrc);
  ...
private:
  Mutex mutex;
  Image *bgimage;
  int imagechages;
};
void prettymenu changebackground(std::iostream imgsrc)
{
  lock(&mutex);
  delete bgimage;
  ++imagechanges;
  bgimage = new image(imgsrc);
  unlock(&mutex);
}
//如果new image出现异常，对unlock的调用就不会执行，于是互斥器就永远被把持住了
//bgimage指向一个已被删除的对象，imagechages也已被累加，而其实并没有新的图像安装
```

- 防止第一个问题·可以用资源管理类，详情见条款十四
- 异常安全函数提供一下三个保证之一,(异常安全码必须提供以下三种保证之一)

```
1，基本承诺，如果异常被抛出，程序内的任何事物仍然保持在有效状态下。
2，强烈保证，如果异常被抛出，程序状态不改变。即如果函数调用失败，程序会回复到调用函数之前的状态
3，不抛掷保证，承诺绝不抛出异常，因为它们总是能够完成它们原先承诺的功能。
```

- 可能的话请提供nothrow保证，但对于大部分函数而言，抉择往往落在基本保证和强烈保证之间
- 上述代码解决办法，在bgimage上使用智能指针（详见条款十三），之后并改变语句次序，即确保更换完图像再累加

```cpp
class prettymenu{
  ...
  std::tr1::shared_ptr<Image>bgiamge;
  ...
}
void prettymenu::changebackground(std::istream& imgsrc)
{
  Lock m1(&mutex);
  bgimage.reset(new Image(imgsrc));
  ++imagechanges;
}
```

- 如果函数只操作局部性状态，相对容易提供强烈保证。
- 应挑选现实可施作条件下最强烈的等级，只有函数调用了传统代码，才设为无保证。

总结：
- 异常安全函数即使发生异常也不会泄露资源或允许任何数据结构破坏，这样的函数区分为三种可能的保证：基本型，强烈型，不抛出异常型。
- 强烈保证往往能够以copy-and-swap实现出来，但强烈保证并非对所有函数都可实现或具备现实意义。
- 函数提供的异常安全保证通常最高只等于其所调用之各个函数的异常安全的最弱者

##### 条款三十，透彻了解inlining的里里外外
- 当inlining某个函数，或许编译器就因此有能力对它执行语境相关最优化。
- inline会导致额外的换页行为，降低指令高速缓存装置的击中率。
- inline只是对编译器的一个命令，不是强制指令，这项申请可以隐喻提出，也可以明确提出，隐喻方式是将函数定义于class定义式内：

```cpp
class person{
public:
  ...
  int age()const {return theage;}    //一个被隐喻的inlining申请
  ...
private:
  int theage;
};
//这样的函数通常是成员函数。
```

- 避免将template声明为inlining。会引发代码膨胀
- virtual函数的调用也都会使inlining函数落空。以为虚函数是在运行期确定，inlinine则是在执行前替换。
- 编译器通常不对通过函数指针而进行的调用实施inlining
- 构造函数析构函数不可以inlining。
- 如果f是程序库内的一个inline函数，客户将f函数本体编进其程勋内，一旦程序库决定改变f，所有用到f的客户端程序都必须重新编译。
- 大部分调试无法对inline起作用
- 一开始先不要声明inline，或至少将inline施行范围局限在那些一定成为inline或平淡无奇的函数身上。

总结：
- 将大多数inlining限制在小型，被频繁调用的函数身上，这可使日后的调试过程和二进制升级过程更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
- 不要只因为function templates出现在头文件，就将它们声明为inline。

##### 条款三十一，将文件间的编译依存关系降至最低
- 如果一个类里的成员变量有其他文件里的类的对象，那么这两个文件就会产生编译依存
- 将接口的实现与声明分开，一个只提供接口，一个负责实现接口

```cpp
#include<string>
#include<memory>
class personimpl;         //person实现类的前置声明。
class date;                   //person接口用到的classes的前置声明
class address;
class person{
public:
  person(const std::string& name, const date& birthday, const address& addr);
  std::string name() const;
  std::string birthdate() const;
  std::string address() const;
  ...
private:
  std::tr1::shared_ptr<personimpl>pimpl;
}
```

- 不该尝试手动声明标准库
- 如果使用object references或object pointers可以完成任务，就不要使用objects。
- 如果能够，尽量以class声明式替换class定义式。
- 为声明式和定义式提供不同的头文件。
- 上述class person被称为handle class，制作办法是将函数转交给相应的实现类或成为abstract base class。

总结：
- 支持编译依存性最小化的一般构想是相依于声明式，不要相依于定义式。基于此两个构想的手段是handle classes 和interface classes。
- 程序头文件应该以完全且仅有声明式的形式存在。这种做法不论是否涉及templates都适用。
