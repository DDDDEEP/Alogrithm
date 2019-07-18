https://www.nowcoder.com/practice/4c776177d2c04c2494f2555c9fcc1e49?tpId=13&tqId=11173&rp=1&ru=/ta/coding-interviews&qru=/ta/coding-interviews/question-ranking

辅助栈记录可以单调递减的值列表

```cp
values: 4 6 2 5 9 1 1 3 8
min   : 4   2     1 1
```

记得判断上下、左右两对边界值是否相等，若是则不要重复输出

```cpp
#include <stack>
class Solution {
public:
    void push(int value) {
        values.push(value);
        if (mins.empty() || value <= mins.top())
            mins.push(value);
    }
    void pop() {
        if (values.top() == mins.top())
            mins.pop();
        values.pop();
    }
    int top() {
        return values.top();
    }
    int min() {
        return mins.top();
    }
private:
    stack<int> values;
    stack<int> mins;
};
```
