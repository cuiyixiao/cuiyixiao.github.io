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
