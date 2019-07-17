该算法找出数据中的中位数作为划分栅栏值，最坏运行时间为 O(n)

算法步骤：（若有偶数个数字时，取下中位数）

1. 将输入数组分为 ⌊n/5⌋ 组，至多只有一个组由剩下的 n mod 5 个元素组成
2. 寻找 ⌈n/5⌉ 个组中每一组的中位数，对每组中的元素进行插入排序，并选出每组中的中位数
3. 在找出来的中位数中递归调用 SELECT 直到找出中位数 x
4. 使用 x 对原输入数组进行已知栅栏值的 PARTITION 划分，得到栅栏 k
5. 若 i == k，返回 x；否则对 i 所在的区域继续递归调用 SELECT

```c++
#include <iostream>
#include <vector>
#include <math.h>
using namespace std;
void Print(vector<int> &A)
{
    for (auto i : A)
    {
        cout << i << " ";
    }
    cout << endl;
}
void InsertSort(vector<int> &A, int p, int r)
{
    int key;
    int j = 0;
    for (int i = p + 1; i <= r; ++i)
    {
        key = A[i];
        j = i - 1;                   // 每次循环，尾部位置向后移一格
        while (j >= p && A[j] > key) // 从后往前找到一个比 key 小的元素
        {
            A[j + 1] = A[j];
            --j;
        }
        A[j + 1] = key;              // 在找到的元素后面插入 key
    }
}
int PartitionWithPivot(vector<int> &A, int p, int r, int pivot)
{
    int pivotIndex = p;
    int i = p - 1;
    for (int j = p; j <= r; ++j)
    {
        if (A[j] <= pivot)
        {
            swap(A[++i], A[j]);
            if (A[i] == pivot)
                pivotIndex = i;
        }
    }
    swap(A[i], A[pivotIndex]);
    return i;
}
int SELECT(vector<int>& A, int p, int r, int i)
{
    int elementCount = r - p + 1;
    int fullGroupCount = elementCount / 5;
    int overElementCount = elementCount - fullGroupCount * 5;
    vector<int> medians;
    for (int j = 0, offset = 0, count = 0; j < fullGroupCount; ++j)
    {
        offset = p + j * 5;
        InsertSort(A, offset, offset + 4);
        medians.push_back(A[offset + 2]);
    }
    if (overElementCount > 0)
    {
        InsertSort(A, fullGroupCount * 5, r);
        medians.push_back(A[p + fullGroupCount * 5 + (overElementCount - 1) / 2]);
    }
    int medianValue = (medians.size() == 1) ? 
        medians[0] :
        SELECT(medians, 0, medians.size() - 1, (medians.size() + 1) / 2);
    int q = PartitionWithPivot(A, p, r, medianValue);
    int k = q - p + 1;
    if (i == k)
    {
        return A[q];
    }
    else if (i < k)
    {
        return SELECT(A, p, q - 1, i);
    }
    else
    {
        return SELECT(A, q + 1, r, i - k);
    }
}
int main()
{
    vector<int> arr = { 3,6,24,9,18,4,29,14,11,1,17,13,27,16,21,22,23,19,12,2,5,7,10,8,15,26,20,25,28 }; // 1~29
    cout << SELECT(arr, 0, arr.size() - 1, 18) << endl;
    Print(arr);
}
```

