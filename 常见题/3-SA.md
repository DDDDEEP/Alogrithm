https://leetcode-cn.com/problems/longest-duplicate-substring/

假设对字符串 S，里面含有的最长重复子串为 R，重复次数为两次，此时其形式可以形如：

S = ^(.\*)R(.\*)R(.\*)$，（注：此处只是简单作例子，两个 R 其实是可以重合的）

对应的以 R 为开头的后缀字符串分别为：R(.\*)R(.\*)$、R(.*)\$，分别记录为 R<sub>1</sub> 和 R<sub>2</sub>

如果将 S 的所有后缀字符串记录在一个数组中，并对数组排序，那么可知 R<sub>1</sub> 和 R<sub>2</sub> 的位置相邻

换句话说，在对后缀字符串数组排序后，带重复前缀的后缀字符串的位置都相邻

例如，banana 对应的后缀数组为：

```cpp
[0] =>      a
[1] =>    ana
[2] =>  anana
[3] => banana
[4] =>     na
[5] =>   nana
```

因此，只要求出字符串对应的后缀数组，并遍历计算相邻字符串的重复前缀的长度，记录最大的重复前缀长度，即为题目的解

因此可以初步得到解法：

```c++
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
// 返回公共最长前缀长度
int CommonSize(const string& lhs, const string& rhs)
{
    int res;
    int minSize = min(lhs.size(), rhs.size());
    for (res = 0; res < minSize; ++res)
    {
        if (lhs[res] != rhs[res])
            break;
    }
    return res;
}
class Solution
{
public:
    string longestDupSubstring(string S)
    {
        
        // 得到后缀数组
        for (size_t i = 0; i < S.size(); ++i)
        {
            suffixArr.push_back(S.substr(i, S.size() - i));
        }
        sort(suffixArr.begin(), suffixArr.end());

        // 比较相邻的子串，记录最大的最长前缀长度
        int maxCommonSize = 0;
        int suffixIndex = 0;
        for (size_t i = 0; i < suffixArr.size() - 1; ++i)
        {
            int commonSize = CommonSize(suffixArr[i], suffixArr[i + 1]);
            if (commonSize > maxCommonSize)
            {
                maxCommonSize = commonSize;
                suffixIndex = i;
            }
        }

        return suffixArr[suffixIndex].substr(0, maxCommonSize);
    }

private:
    vector<string> suffixArr;
};
```

但是这样的后缀数组的空间复杂度为 O(n<sup>2</sup>)，对于字符串长度为 10w 的用例，会超出题目的内存限制

此外这个做法还有很多问题，下面一一列出并给出解决方案：

- 后缀数组占用内存过大，应只是记录后缀串对应的下标
- 新问题：如果只记录下标，如何进行字符串排序？应使用倍增算法
- 快速排序为 O(nlogn)，但因为字符串比较为 O(n)，导致算法的时间复杂度为 O(n<sup>2</sup>logn)，需要优化字符串比较的方法，应在倍增算法中使用基数排序

========== 分割线 ==========

KMP 算法在处理多种字符串模板的匹配时效率较低，因为它在每次查找时都需要对新的模板进行预处理

在这种场景应用中，更适合使用结合倍增算法的后缀数组，此外，还可以利用它生成 height 数组处理最大重复子串问题

> 后缀数组其实为后缀 Trie 的替代物，后缀 Trie(banana) 的结构形如：
>
> ```cpp
>               根
>         /   |   |   \
>        $    a   b    n
>            / \   \    \
>           $   n   a    a
>               |   |   / \
>               a   n  $   n
>             / |   |      |
>            $  n   a      a
>               |   |      |
>               a   n      $
>               |   |
>               $   a
>                   |
>                   $
> ```
>
> 在里面查找一个长度为 m 的模板的时间复杂度为 O(m)，但 Trie 的构造比较复杂

下面描述后缀数组的构造，首先称后缀数组为 SA；然后定义 rank 数组，rank[i] 表示 i 后缀串在 SA 中的排名

在讨论倍增思想原理前，先直接看使用倍增思想构造的过程：

```cp
  len=2^m |  a  |  a  |  b  |  a  |  a  |  a  |  a  |  b  |

1st rank: (  1     1     2     1     1     1     1     2  )
    m=0      |  /        |  /        |  /        |  /
    len=1 | 1 1 | 1 2 | 2 1 | 1 1 | 1 1 | 1 1 | 1 2 | 2 0 |

2nd rank: (  1     2     4     1     1     1     2     3  )
    m=1      |          /
    len=2    |   /-----/      ...（省略）
             |  /
          | 1 4 | 2 1 | 4 1 | 1 1 | 1 2 | 1 3 | 2 0 | 3 0 |

3rd rank: (  4     6     8     1     2     3     5     7  )
    m=2      |                      /
    len=4    |   /-----------------/      ...（省略）
             |  /
          | 4 2 | 6 3 | 8 5 | 1 7 | 2 0 | 3 0 | 5 0 | 7 0 |

4th rank: (  4     6     8     1     2     3     5     7  )
    m=3
    len=8 >= S.size(), END
```

在比较两个后缀的大小时，其实一般比较两个后缀的前面部分字符即可得出结果

对于后缀 i 和 后缀 j，有下面两个性质：

- i 和 j 的前 2k 个字符相等，等价于 i 和 j 的前 k 个字符相等且 i+k 和 j+k 的前 k 个字符相等
- i 的前 2k 个字符小于 j 的前 2k 个字符，等价于（i 的前 k 个字符小于 j 的前 k 个字符）或（i 的前 k 个字符等于 j 的前 k 个字符且 i+k 的前 k 个字符小于后缀 j+k 的前 k 个字符）

可大致用代码表示为：

- `i.substr(0, 2k) == j.substr(0, 2k)` <=> `(i+k).substr(0, k) == (j+k).substr(0, k)`
- `i.substr(0, 2k) < j.substr(0, 2k)` <=> 
    `i.substr(0, k) < j.substr(0, k)` 或
    `(i.substr(0, k) == j.substr(0, k) && (i+k).substr(0, k) < (j+k).substr(0, k)`

通过这两个性质，我们通过所有后缀的前 k 个字符的大小关系，就可以推出所有后缀前 2k 个字符的大小关系

比如在上面的第二次排序中， 其通过 2 个字符串的大小关系得到了 4 个字符串的大小关系：

```cpp
  len=2^m |  aa  |  ab  |  b  |  a  |  a  |  a  |  a  |  b  |
```





