https://leetcode-cn.com/problems/range-sum-query-mutable/

[1, 3, 5, 7, 9, 11] 对应的线段树如下：

```cpp
              36[0-5]
            /        \
        9[0-2]       27[3-5]
        /    \       /    \
    4[0-1]   *5*  16[3-4]  *11*
    /    \        /     \
  *1*    *3*    *7*     *9*
```

因为线段树在创建后，结构一般不会变动，因此建议使用数组实现

数组大小应为 4n，详细原因见 [~~推销博文~~](https://ddddeep.cn/clrs-2/)，不想看的话，记住边界情况可能要用索引位置为 4n - 5 的结点就好

查询函数的实现重点：查询的是两个区间交集的值

更新函数的实现重点：更新子结点后，递归向上更新父结点

```cpp
#include <vector>
#include <algorithm>
using namespace std;
struct Node
{
    int val;
    pair<int, int> interval;
    inline void setVal(int value, int low, int high)
    {
        val = value;
        interval.first = low;
        interval.second = high;
    }
};
inline int lchild(int i)
{
    return 2 * i + 1;
}
inline int rchild(int i)
{
    return 2 * i + 2;
}
inline int parent(int i)
{
    return (i - 1) / 2;
}
class NumArray
{
public:
    NumArray(vector<int> &nums) : nodes(4 * nums.size())
    {
        if (nums.size() != 0)
        {
            build(ROOT_INDEX, 0, nums.size() - 1, nums);
        }
    }

    void update(int i, int val)
    {
        int index = ROOT_INDEX;
        while (nodes[index].interval.first != nodes[index].interval.second)
        {
            int left = lchild(index), right = rchild(index);
            index = (i <= nodes[left].interval.second) ? left : right;
        }

        int differ = val - nodes[index].val;
        while (index > 0)
        {
            nodes[index].val += differ;
            index = parent(index);
        }
        nodes[ROOT_INDEX].val += differ;
    }

    int sumRange(int i, int j)
    {
        return sumRangeHelper(ROOT_INDEX, i, j);
    }

private:
    vector<Node> nodes;
    const int ROOT_INDEX = 0;

    void build(int index, int l, int r, const vector<int> &nums)
    {
        Node* curr = &nodes[index];
        if (l == r)
        {
            curr->setVal(nums[l], l, r);
        }
        else
        {
            int mid = l + (r - l) / 2, left = lchild(index), right = rchild(index);
            build(left, l, mid, nums);      // 递归构造左子树
            build(right, mid + 1, r, nums); // 递归构造右子树
            curr->setVal(nodes[left].val + nodes[right].val, l, r);
        }
    }

    int sumRangeHelper(int index, int l, int r)
    {
        Node *curr = &nodes[index];
        if (l <= curr->interval.first && curr->interval.second <= r)
        {
            return curr->val;
        }
        else if ((curr->interval.first <= l && l <= curr->interval.second)
            || (curr->interval.first <= r && r <= curr->interval.second))
        {
            unsigned lowPos = max(l, curr->interval.first),
                     highPos = min(r, curr->interval.second);
            return sumRangeHelper(lchild(index), lowPos, highPos) +
                   sumRangeHelper(rchild(index), lowPos, highPos);
        }
        else
        {
            return 0;
        }
    }
};
```

