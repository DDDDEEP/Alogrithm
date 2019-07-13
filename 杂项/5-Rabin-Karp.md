https://leetcode-cn.com/problems/longest-duplicate-substring/

Rabin-Karp 也是一种模板串查找算法，它讲每个字符转为 R 进制的数字，然后计算模板串对应的 R 进制值，将其记入 hash 表中

然后，遍历被匹配字符串，计算相同长度的子串的哈希值，如果与模板串的值相同，说明匹配成功

而因为依次遍历被匹配字符串时，下一个字符串只不过是去掉了最前面的一个字符，和添加了最后面的一个字符，因此利用上一个子串的哈希值，就可以很快地通过“滚动”算出下一个子串的哈希值

在计算中，取 R 进制，并取一个足够大的质数 M（取质数是为了让散列值分布均衡）

如果 R 非常大，那么累加各个字符对应的数值是很容易溢出的，我们应该利用求余运算的性质：

```cpp
(a + b) % n = (a % n + b % n) % n;
(a - b) % n = (a % n - b % n + n) % n;
(a * b) % n = (a % n * b % n) % n;
```

当然是用哈希就会有碰撞问题，如果元素较少且比较随机，那么采用一个较大的质数 M 就一般不会碰撞，但元素较多就不太稳定了

因此如果只是子字符串查找的话，稳定的 KMP 一般会更好用，但 Rabin-Karp 的思想可用于解决很多问题

## 子字符串查找

https://leetcode-cn.com/problems/implement-strstr/

```cpp
class Solution
{
public:
    int strStr(string haystack, string needle)
    {
        return RabinKarp(haystack, needle);
    }

private:
    inline int pr(int num, int prime = PRIME)
    {
        return num % prime;
    }
    int RabinKarp(const string &str, const string &pattern)
    {
        if (pattern.size() == 0)
            return 0;
        if (pattern.size() > str.size())
            return -1;

        // maxRadixNum = radix ^ (pattern.size() - 1) % prime
        int maxRadixNum = 1;
        for (int i = 0; i < pattern.size() - 1; ++i)
        {
            maxRadixNum = pr(maxRadixNum * RADIX);
        }

        int strHash = 0, patternHash = 0;
        for (int i = 0; i < pattern.size(); ++i)
        {
            strHash = pr(strHash * RADIX + str[i]);
            patternHash = pr(patternHash * RADIX + pattern[i]);
        }

        for (int i = 0; i <= str.size() - pattern.size(); ++i)
        {
            if (i != 0)
            {
                // ((strHash - maxRadixNum * highChar) * radix + lowChar) % prime
                strHash = pr(
                    pr(
                        (pr(pr(strHash) * RADIX)
                        - pr(pr(pr(maxRadixNum) * str[i - 1]) * RADIX) + PRIME)
                    ) + str[i + pattern.size() - 1]
                );
            }
            if (strHash == patternHash)
            {
                int charIndex;
                for (charIndex = 0; charIndex < pattern.size(); ++charIndex)
                {
                    if (str[i + charIndex] != pattern[charIndex])
                        break;
                }

                if (charIndex == pattern.size())
                {
                    return i;
                }
            }
        }

        return -1;
    }

    // 2147483647 / 128 = 16777215.99
    const static int RADIX = 128;
    const static int PRIME = 16777213;
};
```

