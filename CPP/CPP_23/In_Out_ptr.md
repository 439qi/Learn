---
title: in_out_ptr
created: 2025-03-13
tags:
  - cpp/23
  - "#TBD"
---
> [!cite]- References  
> - [C++23特性总结 - 下 - 知乎](https://zhuanlan.zhihu.com/p/562383556#h_562383556_10)  
> - [c++ - Understanding std::inout_ptr and std::out_ptr in C++23 - Stack Overflow](https://stackoverflow.com/questions/68918312/understanding-stdinout-ptr-and-stdout-ptr-in-c23)  

```cpp
#include <memory>
```

`std::out_ptr` 与 `std::inout_ptr` 用于解决智能指针与 C 接口的调用繁琐的包装类  

对于一个接受二级指针的 C 函数，通过智能指针调用时仍需要用户手动管理内存  
```cpp
void func(int**);

std::unique_ptr<int> ptr;
auto raw_ptr = ptr.release();
func(raw_ptr);
ptr.reset(raw_ptr);
```

`std::inout_ptr` 在智能指针之上进行封装，自动调用 `release()` 和 `reset()`，则上面的代码可以简化为    

```cpp
func(std::inout_ptr(ptr));
```

相应地，`std::out_ptr` 仅会调用 `reset()`，即其假定调用函数会重新分配内存返回而不会手动删除原有内存  