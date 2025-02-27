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
