https://leetcode-cn.com/problems/implement-strstr

[阮一峰大大的博文](http://www.ruanyifeng.com/blog/2013/05/Knuth–Morris–Pratt_algorithm.html)

移动位数 = 已匹配的字符数 - 对应的部分匹配值

关于 next 数组的构造，如果每次都对一个前缀字符串产生一个前缀集合和一个后缀集合，并找出最长的公共元素，这也太糟糕了

下面介绍一种 O(n) 的构造方法，具体实现细节会有很多种版本，下面这种是我觉得比较容易理解的

首先直接看它的结构：

```cpp
pattern:  A B A C A B A B D
next:    -1 0 0 1 0 1 2 3 2
```

在这种实现下，next[0] = -1，此时 next[i] 代表 pattern.substr(0, i) 中最长公共前后缀的长度

然后才是最大的问题，它到底是怎么计算的呢？下面直接以最后一个 next 值 "2" 的计算过程作为例子讲解：

```cpp
pattern:  A B A C A B A B D
next:    -1 0 0 1 0 1 2 3

1. k = next[i-1] = 3

2. 
                |-------|
                V       |
pattern:  A B A C A B A B D

index:          3         i

    next[i-1] 的值指示了 (A B A C A B A) 的最长公共前后缀的长度为 3，对应 ABA
    同理，next[i] 指示了 (A B A C A B A B) 的最长公共前后缀，且它的长度最大为 4
    它的公共前后缀长度何时才能为 4 呢？因为它只多了一个字符 (B)，而之前的前后缀有公共部分
    所以只要之前的前缀的后一个字符(C)，等于后缀多的那个字符 (B)，那么公共后缀可以被继承，此时长度为4 
    并且刚好的，next[i-1] 指示的，其实就是最长公共前缀的后一个字符，对应的位置

3. pattern[k] == pattern[i-1] ?
    true  ----> next[i] = k + 1
    false ----> k = next[k]

    true 的话就是可以继承上一个最长公共前后缀，容易理解，不详述
    那 false 是什么意思呢？是这样的，我们不能因为无法继承之前的最长公共前后缀就选择放弃
    如果之前的最长公共前后缀本身也存在自己的最长公共后缀，那么它也可以被利用
    比如此时，k=1，即有一个嵌套的、长度为 1 的公共部分：
    (ABA)C(ABA)B
     A A   A A
      ?        B
    如果 ? 处的字符和 i-1 处的字符相同，那么我们就可以继承 A 这个小公共前后缀啦
    这个过程其实就是步骤 2，也就是说步骤 2 和 3 构成一个循环

4. k == -1, break
    边界条件，不详述了
```

对于 next 数组的使用过程，记住搜索串的下标是单调递增的

而当模板串对应下标的字符等于搜索串下标的字符，或模板串下标在开始位置，两个下标才会递增；

否则，模板串下标取 next 数组的对应值

将上面相当意识流的解释写成代码，就是这个样子：

```cpp
#include <vector>
using namespace std;
class Solution {
public:
    int strStr(string haystack, string needle) {
        if (needle.size() == 0)
            return 0;
        // 创建next数组
        vector<int> next(needle.size(), 0);
        next[0] = -1;
        int k;
        for (int i = 1, k = 0; i < needle.size(); ++i)
        {
            k = next[i - 1];
            while (k != -1 && needle[k] != needle[i - 1])
            {
                k = next[k];
            }
            next[i] = k + 1;
        }

        int hayIndex = 0, needIndex = 0, res = -1;
        while (hayIndex < haystack.size())
        {
            if (needIndex == -1 || haystack[hayIndex] == needle[needIndex])
            {
                ++hayIndex;
                ++needIndex;
            }
            else
            {
                needIndex = next[needIndex];
            }

            if (needIndex == needle.size())
            {
                res = hayIndex - needle.size();
                break;
            }
        }
        return res;

    }
};

```



