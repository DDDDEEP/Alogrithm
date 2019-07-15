整理一下几种二分的版本：

## 查找是否在数组

```cpp
#include <iostream>
#include <vector>
using namespace std;
int BinarySearch(const vector<int> &arr, int key)
{
    int l = 0, r = arr.size() - 1, mid;
    while (l <= r)
    {
        mid = l + (r - l) / 2;
        if (key < arr[mid])
            r = mid - 1;
        else if (arr[mid] < key)
            l = mid + 1;
        else
            return mid;
    }
    return -1;
}

int main()
{
    cout << BinarySearch(vector<int>{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10}, 3) << endl;
}
```

## 查找第一次出现的位置

```cpp
#include <iostream>
#include <vector>
using namespace std;
int BinarySearch(const vector<int> &arr, int key)
{
    int l = 0, r = arr.size() - 1, mid;
    while (l <= r)
    {
        mid = l + (r - l) / 2;
        if (key <= arr[mid])
            r = mid - 1;
        else
            l = mid + 1;
    }
    return arr[r + 1] == key ? r + 1 : -1;
}

int main()
{
    vector<int> temp{0, 1, 2, 3, 3, 3, 3, 4, 5, 6, 6, 6, 6, 6, 7, 8, 9, 10, 10, 10, 10};
    cout << BinarySearch(temp, 6) << endl;
}
```

## 查找最接近 key 且大于 key 的数的位置

```cpp
#include <iostream>
#include <vector>
using namespace std;
int BinarySearch(const vector<int> &arr, int key)
{
    int l = 0, r = arr.size() - 1, mid;
    while (l <= r)
    {
        mid = l + (r - l) / 2;
        // lower_bound:
        // if (key <= arr[mid])
        // upper_bound:
        if (key < arr[mid])
            r = mid - 1;
        else
            l = mid + 1;
    }
    // return key <= arr[r + 1] ? r + 1 : -1;
    return key < arr[r + 1] ? r + 1 : -1;
}

int main()
{
    vector<int> temp{0, 2, 2, 3, 5, 6, 9, 10};
    cout << BinarySearch(temp, 2) << endl;
}
```

