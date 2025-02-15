# Algorithm

## 回溯

回溯算法的求解过程实质上是先序遍历“状态树”的过程

- 剪枝  
  回溯算法解决问题的过程中创建的状态树并不都是满二叉树，因为在试探的过程中，有时会发现此种情况下，再往下进行没有意义，所以会放弃这条死路，回溯到上一步

## 动态规划(DP)

### 状态转移方程

1. 设计状态
2. 列出状态转移方程
3. 初始状态
4. 非法状态
5. 状态初始化

### 记忆化搜索

记忆化搜索 = 动态规划的思想 + 深度优先搜索的实现

1. 合法性剪枝  
  递归计算时保证参数合法性  
2. 偏序关系剪枝  
  代表状态转移的方向
3. 记忆化剪枝  
  对于已计算的结果直接返回
4. 递归计算结果并返回  
  对应深搜的递归过程

## KMP算法

## Deflate压缩算法

## Floyd判圈算法(Floyd Cycle Detection Algorithm)

在有限状态机、迭代函数或者链表上判断是否存在环，以及判断环的起点与长度

设快指针为fast，慢指针为low，
如果有环

1. 有限时间内两个指针都能运动到环上
2. 快慢指针在环上时，在有限时间内必然能相遇

设链表总长为L+C，环长为C，当两者相遇时，low移动了S距离，fast移动了2S距离，则有
$$
2S - S  = S = kC, k∈N
$$

此时将两者速度都置1，将low指针移到链表头，由如下等式
$$
2S+L = 2kC+L, k∈N
$$
两者必定在循环开始处相遇

## SWAR算法(SIMD(Single Instruction, Multiple Data) within a Register)

```cpp
uint32_t swar(uint32_t i){
    i = (i & 0x55555555) + ((i>>1) & 0x55555555);  // 步骤1
    i = (i & 0x33333333) + ((i>>2) & 0x33333333);  // 步骤2
    i = (i & 0x0F0F0F0F) + ((i>>4) & 0x0F0F0F0F);  // 步骤3
    i = (i * 0x01010101) >> 24;                    // 步骤4
    return i;
}
```

1. 步骤一  
分别取奇数位和偶数位相加（0x55555555 = 0b01010101010101010101010101010101），值域[0,2]，结果用相邻的两bit存储
2. 步骤二  
同上，每两个bit为一组，分别取奇数组与偶数组相加，值域[0,4]，结果用相邻的4bit存储
3. 步骤三  
同上
4. 步骤四  
    4个8bit整数相加存在更简便算法，等价于移位相加

    $$
    0x01010101 = 2^{24}+2^{16}+2^{8}+2^{0}\\
    i * 0x01010101 =i * 2^{24}+i * 2^{16}+i * 2^{8}+i * 2^{0}  \\
    i * 0x01010101 = (i<<24)+(i<<16)+(i<<8)+(i<<0) \\
    (i * 0x01010101)>>24 = i+(i>>8)+(i>>16)+(i>>24) \\
    (i * 0x01010101)>>24 = i\&0xFF+(i>>8)\&0xFF\\+(i>>16)\&0xFF+(i>>24)\&0xFF
    $$
    其中0xFF防止移位时高位补1

## Sprague-Grundy定理

1. 策梅洛定理（Zermelo's theorem）  
    若一个游戏满足如下条件：

    - 双人、回合制；
    - 信息完全公开（perfect information）；
    - 无随机因素（deterministic）；
    - 必然在有限步内结束；
    - 没有平局；  

    则游戏中的任何一个状态，要么先手有必胜策略，要么后手有必胜策略

Sprague-Grundy定理前提条件：

- 游戏双方可以采取的行动是相同的  
  井字棋、五子棋、围棋、象棋、跳棋这些棋类游戏均不满足这个条件，因为游戏的双方只能下（或移动）己方的棋子。
- 游戏双方的胜利目标是相同的  
  常见的胜利目标包括把棋盘清空或填满，或者把棋子排成特定的形状。注意，如果双方的目标是把棋子排成不同的形状，则游戏不满足这个条件。

## 逐比特确认法

## 摩尔投票法

常用于求众数  

分为pairing和counting两个阶段

- pairing阶段  
  两个不同的元素互相抵消，直至只剩同一元素

  记录当前元素以及计数，当计数为0时，更新为新元素；否则遇到相同元素时，计数加一，遇到新元素时，计数减一
- counting阶段  
  判断最后剩余的元素个数是否超过$\frac{1}{2}$

  重新扫描数组进行计数
