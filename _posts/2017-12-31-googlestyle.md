# google styleguide
## Header Files
### head file
- 普遍来说，一个.c文件都有一个.h头文件，除非是那种小.c文件例如只有main函数的
- 一个头文件需要有header guards并包括其他需要的头信息
- 将模板和内联函数的定义放到一个文件中作为它们的声明 ，并包含在每个使用他们的.c文件中
- 不要把定义放在头文件中。
- the #define guide保护头文件foo/src/bar/baz.h

```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
...
#endif //FOO_BAR_BAZ_H_
```

- 避免使用向前声明。
- 定义内联函数只可以在它们很小，10行或者更小
- Names and Order of Includes

```
    dir2/foo2.h.
    C system files.
    C++ system files.
    Other libraries' .h files.
    Your project's .h files.
```

## Scoping
### namespaces
- 将代码尽可能的放到命名空间里，不要使用using，不要使用内联namespace
- 可以防止全局内命名冲突

```cpp
// In the .h file
namespace mynamespace {

// All declarations are within the namespace scope.
// Notice the lack of indentation.
class MyClass {
 public:
  ...
  void Foo();
};

}  // namespace mynamespace
```

### Unnamed Namespaces and Static Variables
- 在不需要别处引用的定义，将它们放到未命名的命名空间或者声明为静态。
### Nonmember, Static Member, and Global Functions
- 尽量用命名空间分组函数而不是类
- 尽量在namespace里用非成员函数而不是全局函数
- 静态成员函数尽量写成非成员函数，如下

```cpp
namespace myproject {
namespace foo_bar {
void Function1();
void Function2();
}  // namespace foo_bar
}  // namespace myproject
```

intead of

```cpp
namespace myproject {
class FooBar {
 public:
  static void Function1();
  static void Function2();
};
}  // namespace myproject
```

### Local Variables
- 将函数变量放在尽可能窄的范围
- 声明变量尽可能接近第一次使用
- 声明时就应初始化，如下

```cpp
int i;
i = f();      // Bad -- initialization separate from declaration.

int j = g();  // Good -- declaration has initialization.

std::vector<int> v;
v.push_back(1);  // Prefer initializing using brace initialization.
v.push_back(2);

std::vector<int> v = {1, 2};  // Good -- v starts initialized.
```

### Static and Global Variables
- 禁止静态类变量(why.....)
- 考虑使用指针代替（且不能是智能指针）

## Classes
### Doing Work in Constructors
- 构造函数不能为虚函数
- 构造函数中的任意工作都不能传递给其他线程
### Implicit Conversions
- 不能隐式转换
- 尽量使用列表初始化（effective c++里有说）
- 复制和移动构造函数不应该是显示的。
###Copyable and Movable Types
- 如果复制和移动没有有明确的意义应当禁用隐式生成的复制和移动特殊函数
- 请一起提供copy和move函数

```cpp
class Foo {
 public:
  Foo(Foo&& other) : field_(other.field) {}
  // Bad, defines only move constructor, but not operator=.

 private:
  Field field_;
};

```
### Structs vs. Classes
- struct用于数据集和，不会提供数据行为（除了初始化和析构），无需通过函数访问，否则其它一律用类。
### Inheritance（继承）
- 所有的继承应为公有继承
- 如果类有虚拟的方法，则析构函数应为虚（effective c++）
### Multiple Inheritance（多继承）
- 只有在一个基类中有一个实现时，我们才允许多继承；所有其他基类都必须是接口后缀标记的纯接口类。
### Operator Overloading
- 不应创建自定义的文字用来重载运算符
- 重载运算符要注意意义相同
- 如果定义操作符，要将与其相关一同定义
### Access Control
- 使成员变量私有，除非它们是静态变。
### Declaration Order
- 将相似的声明放在一起，将公共部分提前放置，应先从共有开始，之后是保护，之后是私有。

## Functions
### Parameter Ordering
- 定义一个函数时，输入参数在前，输出参数在后。
### Write Short Functions
- 考虑函数在40行之内。。。（what？）
### Reference Arguments
- 引用标记的参数须声明为const
- 输入参数是值或const引用，输出参数是指针

```cpp
void Foo(const string &in, string *out);
```

### Default Arguments
- 默认参数在虚函数上禁用
### Trailing Return Type Syntax
- 普通语法函数不使用尾随返回类型

```cpp
int foo(int x);
```

- 返回模板参数或者lambada表达式参数时使用尾随返回类型

```cpp
template <class T, class U> auto add(T t, U u) -> decltype(t + u);
template <class T, class U> decltype(declval<T&>() + declval<U&>()) add(T t, U u);
```

## Google-Specific Magic
### Ownership and Smart Pointers
- 使用智能指针

## Other C++ Features
### Rvalue References
