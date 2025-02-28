---
title: Value Categories
aliases:
  - 值种类
created: 2025-02-27
tags:
  - cpp/11
  - TBD
---
> [!cite]- References  
> [Value categories - cppreference.com](https://en.cppreference.com/w/cpp/language/value_category)  
> [Value Categories: Lvalues and Rvalues (C++) | Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/lvalues-and-rvalues-visual-cpp?view=msvc-170)  
> [Universal References in C++11 -- Scott Meyers : Standard C++](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)  
> [现代C++之万能引用、完美转发、引用折叠 - 知乎](https://zhuanlan.zhihu.com/p/99524127)  

## 移动语义  

移动的思想是将对象的所有权进行传递  

如果对象中有数据位于堆上，则将指针拷贝给新对象，并将旧对象的指针置空  
## 性质  

任意**表达式**通过两个独立的性质可以划分为三种基本值种类  

1. 拥有身份(has identity)  

    是否指代非临时对象  
    身份指地址，名字或别名  

2. 可被移动(can be moved from)  
    是否可被右值引用类型匹配  

## 基本值种类  

### 左值 lvalue  

拥有身份且不可被移动  

> [!example] 部分左值表达式  
> - 变量名、函数名、模板形参对象名、数据成员名  
>   > [!warning] 右值引用类型的变量组成的表达式，其值种类属于左值表达式  
>   > ```cpp
>   > void func(int &&a){
>   >   a;//lvalue expression
>   > }
>   > ```
> - 字符串字面值  
> - ==返回左值引用==的函数调用，包括运算符重载函数  
> - 内置类型的赋值表达式  
> - 内置类型的前置自增、自减运算符  

### 将亡值 xvalue  

拥有身份且可被移动  

> [!example]  
> - 右值引用类型返回值的函数调用，包括运算符重载函数  
> - prvalue 实体化产生的临时对象  
> - 被 return/co_return/throw 的局部变量  

### 纯右值 prvalue  

不拥有身份且可被移动  

> [!example] 部分纯右值表达式  
> - 除字符串字面值之外的字面值  
> - 非引用类型返回值的函数调用，包括运算符重载函数  
> - 内置类型的后置自增、自减运算符  
> - 取地址运算符表达式  
> - lambda 表达式  

特定情况下 prvalue 会实体化并产生一个临时对象(temporary object)，种类为 xvalue  
C++17 规定了 guaranteed copy elision/RVO，prvalue 返回值用作初始化时直接在要初始化的变量上构造或移动构造  
```cpp
T func(){
	return T();
}
//...
T t = func();
```

如下返回值为 xvalue，不会触发 RVO，但 `func2()` 函数体的局部变量 `ret` 会被 NRVO 优化掉  

```cpp
T func2(){
	T ret;
	...
	return ret;
}

T t2 = func2();
```


## 混合值种类  

依据拥有身份或是否可被移动，可划分两种混合值种类  

- 泛左值 glvalue = 将亡值 + 左值  
- 右值 rvalue = 将亡值 + 纯右值  

## 值种类对应的构造方法  

> [!warning] 对于栈上的对象，直接构造、移动构造、拷贝构造没有区别  

| 值种类     | 被初始化的值种类 | 构造方法   |
| ------- | -------- | ------ |
| prvalue | gvalue   | 直接构建   |
| xvalue  | lvalue   | 移动构造函数 |
| lvalue  | lvalue   | 拷贝构造函数 |

## std::move()  

`move()` 作用于编译期，将一个左值转换为右值  
其原理是将移除传入参数的所有引用性质并强制转换为对应类型的右值引用并返回  

```cpp
_EXPORT_STD template <class _Ty>
_NODISCARD _MSVC_INTRINSIC constexpr remove_reference_t<_Ty>&& move(_Ty&& _Arg) noexcept {
    return static_cast<remove_reference_t<_Ty>&&>(_Arg);
}
```
## 引用折叠 reference collapsing

C++ 语言不允许引用的引用，但在模板中类型运算的结果有可能出现引用的引用  
因而 C++11 引入了引用折叠规则，当进行类型推导==后==出现引用的引用时，将其折叠为单个引用  

> [!note] 引用折叠规则  
> - 绑定右值引用的右值引用折叠为右值引用  
> - 其余组合折叠为左值引用，即只要有左值引用参与最终就会折叠为左值引用  

- 万能引用  
  ```cpp
  template<typename T>
  void func(T&& arg);
  ```
- 模板类中的 typedef 或 using 类型  
  ```cpp
  template<typename T>
  class A{
    using lvalue_ref = T&;
  }
  void func(A<int>::lvalue_ref&& param);
  ```
- auto  
  ```cpp
  int&& var=1;
  auto&& var2 =var;//deduced to int& &&var2, then collapsed to int& var2;
  ```
- decltype  
	#TBD decltype 推导规则略为晦涩  

### 引用折叠的类型推导  

> [!note] 引用折叠的类型推导  
> - 绑定到左值时 `T` 被推导为具体类型的左值引用  
> - 绑定到右值时 `T` 被推导为具体类型  

C++ 中模板推导的实参为引用类型时，以剔除引用的具体类型参与推导  
同时引用对象必定为左值，因而万能引用绑定的实例为也为引用时，无论左值引用还是右值引用，`T` 始终被推导为具体类型的左值引用  

如下代码中，`f(x)` 中 `T` 被推导为 `int&`，`f(10)` 中 `T` 被推导为 `int`  
`f(y)` 和 `f(y2)` 中 `T` 均被推导为 `int&`  

```cpp
template<typename T>
void func(T&& arg);

int x = 1;
f(x);

f(10);

int&& y=x;
int& y2=x;
f(y);
f(y2);
```

### 万能引用  

万能引用的本质是引用折叠  
> [!warning] 无论引用绑定到左值或右值，引用对象本身为左值对象  

- 左值引用: 可以指向左值，不能指向右值  
  > 例外: const 左值引用可以指向右值  

- 右值引用: 可以指向右值，不能指向左值  
  常用于延长临时对象的生命周期  
  > 只能延长 prvalue，若 prvalue 被实体化为 xvalue 则不会延长其生命周期  

  #TBD  
  右值引用能指向纯右值的本质是通过一个临时变量将右值提升为左值，然后通过 `std::move` 转换为右值  

  ```cpp
  int &&ref=5;
  //等价于
  int temp = 5;
  int &&ref = std::move(temp);
  ```

- 万能引用: 形如 `T&&` 或 `auto&&` 绑定到被推导类型的左值或右值的引用  

  引用符号 `&&` 必须直接跟在被推导类型后，且不能被 const 修饰  
  如下代码均为右值引用  

  ```cpp
  template<typename T>
  void func(std::vector<T> &&x);

  typename<typename T>
  void func(const T&& x);
  ```

  同时只有类型参数 `T` 必须发生类型推导时才为万能引用  
  如下代码中，`func()` 的模板类型参数与模板类 `A` 相同，当模板类 `A` 实例化时，模板类型参数 `T` 便已被实例化为具体类型， 当调用 `func()` 时无需进行类型推导，形参 `arg` 为右值引用  
  ```cpp
  template<typename T>
  class A{
  public:
    void func(T&& arg);
  } 
  ```  
  如下代码为万能引用  
  ```cpp
  template<typename T>
  class A{
  public:
    template<typename Y>
    void func(Y&& arg);
  } 
  ```

### 完美转发  

将万能引用作为实参调用其他函数时，由于万能引用对象本身是左值，会丢失被绑定对象的值种类  
如下代码中 `forward_func(x)` 和 `forward_func(1)` 均转发到 `void func(int& arg)`  

```cpp
void func(int&& arg);
void func(int& arg);

template<typename T>
void forward_func(T&& arg){
	func(arg);
}

int x = 1;
forward_func(x);
forward_func(1);
```

完美转发 `std::forward` 可以将万能引用转换为其原本的值种类  
其具体实现利用到了[引用折叠](CPP/CPP_11/Value_Categories.md#引用折叠%20reference%20collapsing)  
```cpp
//forward rvalue
_EXPORT_STD template <class _Ty>
_NODISCARD _MSVC_INTRINSIC constexpr _Ty&& forward(remove_reference_t<_Ty>&& _Arg) noexcept {
    static_assert(!is_lvalue_reference_v<_Ty>, "bad forward call");
    return static_cast<_Ty&&>(_Arg);
}

//forward lvalue
_EXPORT_STD template <class _Ty>
_NODISCARD _MSVC_INTRINSIC constexpr _Ty&& forward(remove_reference_t<_Ty>& _Arg) noexcept {
    return static_cast<_Ty&&>(_Arg);
}
```

