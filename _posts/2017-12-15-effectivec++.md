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
