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

##### 条款五
- 一个空类会默认生成copy构造函数，一个copy assignment操作符，一个析构函数，一个default构造函数
- 如果已经声明了一个构造函数，编译器将不再为它创建default构造函数，
- 要注意，一般而言，只有代码合法且证明它有意义编译器才会生出operator = （比如&，const就会出现问题）
