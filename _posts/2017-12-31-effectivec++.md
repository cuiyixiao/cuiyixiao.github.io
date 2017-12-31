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
