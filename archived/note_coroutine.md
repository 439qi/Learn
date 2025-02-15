# 协程
>
> <https://zplutor.github.io/2022/03/25/cpp-coroutine-beginner/>  

- **定义**: 协程是一个函数，它可以暂停以及恢复执行

协程执行的是并发任务，由协程自身管理上下文，并在外设 IO 等需要等待的情况时主动切换  

相对应地，线程能利用多核处理器进行并**行**任务，通过线程执行并**发**任务的问题在于线程切换的开销以及同步  

协程分为有栈和无栈两种
> 此处的 `栈` 指逻辑栈，而非内存栈
>
## 有栈协程

相当于用户态线程，可抢占内核调度  
切换成本与用户态线程切换的成本相同

## 无栈协程

可挂起/恢复的函数，只能被线程调用，本身不抢占内核调度  
切换成本与函数调用相同
> C++20 协程使用无栈协程  

## Cpp Coroutines

协程的关键在于允许手动调度，在明确需要进行长时间等待的操作时，主动让出 CPU  

协程指满足如下要求的函数

- 参数不能为可变参数
- 函数体中包含 `co_await` 或 `co_yield` 或 `co_return`
- 返回值不能使用普通返回语句，占位返回类型(`auto`、 `Concept`)
- 不能为 consteval 函数，constexpr 函数，构造函数，析构函数，main 函数

每一个协程与如下对象关联

- promise_type 对象（!!!**不是标准库中的 `std::promise`**）  
    协程通过该对象提交结果或异常
- 协程句柄  
    协程外部通过该对象恢复或销毁协程帧
- 协程状态  
    协程内部的动态存储空间，包含  
  - promise 对象  
  - 协程参数  
  - 当前暂停点的相关信息  
  - 局部变量

### Cpp 协程数据类型

- `std::coroutine_handle`

### Cpp 协程执行流程

1. 使用 `new` 运算符分配协程状态对象
2. 拷贝协程参数(存在悬垂引用的可能)
3. 调用 promise 对象的构造函数  
4. 调用 promise 对象的 `promise::get_return_object()` 方法
5. 调用 promise 对象的 `promise::initial_suspend` 方法并 `co_await` 结果
6. 当从 `promise::initial_suspend` 恢复时，继续执行协程的函数体剩余部分
