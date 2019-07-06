https://leetcode-cn.com/problems/lru-cache/description

因为经常需要进行页面的挪动操作，为了提高页面移动效率，应使用链表记录页面

又因为需要快速地判断对应页面是否已经存在于链表中，因此需要使用哈希表记录不同页面的位置

```c++
#include <list>
#include <unordered_map>
using namespace std;
typedef pair<int, int> keyval;
typedef list<keyval>::iterator it;
class LRUCache
{
public:
    LRUCache(int capacity) : maxSize(capacity)
    {
    }

    int get(int key)
    {
        // 检查哈希表有无对应缓存键
        // 若无对应缓存键，返回 -1
        auto mapIt = theMap.find(key);
        bool keyIsFound = (mapIt != theMap.end());
        if (!keyIsFound)
        {
            return -1;
        }

        // 若有对应缓存键，将其移到最新位置，更新并返回缓存值
        auto listIt = mapIt->second;
        theList.splice(theList.begin(), theList, listIt);
        mapIt->second = theList.begin();
        return theList.front().second;
    }

    void put(int key, int value)
    {
        // 检查哈希表有无对应缓存键
        // 若有对应缓存键，更新缓存值对应的结点的值，并将其放到列表最新位置
        auto mapIt = theMap.find(key);
        bool keyIsFound = (mapIt != theMap.end());
        if (keyIsFound)
        {
            mapIt->second->second = value;
            theList.splice(theList.begin(), theList, mapIt->second);
        }
        // 若无对应缓存键，将对应结点值插入到列表最新位置上，并在哈希表中缓存对应的迭代器
        else
        {
            theList.push_front(make_pair(key, value));
            theMap.insert(make_pair(key, theList.begin()));
        }

        // 检查缓存大小是否超过最大限制大小，若是，则删除最久的缓存和对应的结点
        if (theList.size() > maxSize)
        {
            theMap.erase(theList.back().first);
            theList.pop_back();
        }
    }

private:
    unordered_map<int, it> theMap;
    list<keyval> theList;
    const int maxSize;
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache* obj = new LRUCache(capacity);
 * int param_1 = obj->get(key);
 * obj->put(key,value);
 */
```

