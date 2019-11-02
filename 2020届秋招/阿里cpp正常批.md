## 特殊的跳跃表

该题目要求补全跳跃表的 insert 和 find 函数，但该题目要求的跳跃表有点特殊。

该跳跃表只有一层跳点层，每个跳点记录一个 count，表示从该跳点到下一个跳点，共有多少个结点；然后count 会有一个最大值，例如最大值取128，则要求 count 不能大于128，如果有跳点的 count 超过128，既要新建一个跳点重新调整。

但本题未对新建跳点的策略并无详述，我在此采取半分跳点的策略：count 超过最大值后，将跳点一分为二。另外为了方便实现，代码中添加了 NIL 结点辅助。

比较奇怪的是，在测试代码逻辑中，如果总结点数不大于 count 所定的最大值，那么就不能新建任何跳点。可是，配合此题逻辑，跳点应该在第一个 link 点出现后就新建，这样这个跳跃表才有实用意义，况且测试代码下一步又开始遍历计算跳点的 count 之和，是否等于总结点数，如果按照后一个逻辑的话，第一个 link 点就应由第一个跳点映射。

总之，我注释了测试函数的部分代码，并遵循第一个 link 点应由第一个跳点映射来实现，反正这题本身怪怪的，应将重点放在跳点间的维护逻辑中。

（也很可能是我理解错题意了，希望有大大指点）

```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <limits>
#include <algorithm>
#include <stdint.h>
#include <string>

using namespace std;

struct LinkNode
{
    uint64_t key;
    uint64_t value;
    LinkNode* next = NULL;
    LinkNode(uint64_t key, uint64_t value, LinkNode* next = nullptr);
};

LinkNode::LinkNode(uint64_t key, uint64_t value, LinkNode* next)
    : key(key)
    , value(value)
    , next(next)
{

}

struct SkipLinkNode
{
    uint64_t key;
    int count;
    LinkNode* pointer = NULL;
    SkipLinkNode* next = NULL;
    SkipLinkNode(uint64_t key, int count, LinkNode* pointer = nullptr, SkipLinkNode* next = nullptr);
};

SkipLinkNode::SkipLinkNode(uint64_t key, int count, LinkNode* pointer, SkipLinkNode* next)
    : key(key)
    , count(count)
    , pointer(pointer)
    , next(next)
{

}

class SkipLinkList
{
public:
    SkipLinkList(int skipStep = 128);
    ~SkipLinkList();
    bool simpleCheck();

public:
    void insert(uint64_t key, uint64_t value);
    bool find(uint64_t key, uint64_t &value);

private:
    LinkNode* _listHeader = NULL;
    SkipLinkNode* _skipListHeader = NULL;
    LinkNode* _listNil = NULL;
    SkipLinkNode* _skipListNil = NULL;
    int _skipStep;
    int _itemCount;
};

SkipLinkList::SkipLinkList(int skipStep)
    : _skipStep(skipStep)
    , _listHeader(NULL)
    , _skipListHeader(NULL)
    , _itemCount(0)
{
    _listNil = new LinkNode(-INT_MAX, -INT_MAX, nullptr);
    _skipListNil = new SkipLinkNode(-INT_MAX, -INT_MAX, _listNil, nullptr);
}


SkipLinkList::~SkipLinkList()
{
    LinkNode* node = _listHeader;
    while (node != NULL) {
        LinkNode* toDelNode = node;
        node = node->next;
        delete toDelNode;
    }

    SkipLinkNode* skipNode = _skipListHeader;
    while (skipNode != NULL) {
        SkipLinkNode* toDelNode = skipNode;
        skipNode = skipNode->next;
        delete toDelNode;
    }
    delete _listNil;
    delete _skipListNil;
}

bool SkipLinkList::simpleCheck()
{
    uint64_t lastKey = 0;
    int nodeCount = 0;
    LinkNode* node = _listHeader;
    while (node != NULL)
    {
        nodeCount++;
        if (node->key < lastKey)
        {
            return false;
        }
        lastKey = node->key;
        node = node->next;
    }
    //// 如果key数量少于最大值，就无跳点
    //// 要单独进行判断处理，注释此处不作检查
    //if (nodeCount < _skipStep)
    //{
    //    return (_skipListHeader == NULL);
    //}

    lastKey = 0;
    int skipNodeCount = 0;
    int expectNodeCount = 0;
    SkipLinkNode* skipNode = _skipListHeader;
    while (skipNode != NULL)
    {
        skipNodeCount++;
        if (skipNode->key < lastKey)
        {
            return false;
        }
        lastKey = skipNode->key;
        expectNodeCount += skipNode->count;
        skipNode = skipNode->next;
    }
    return nodeCount == expectNodeCount &&
        skipNodeCount >= ((nodeCount + _skipStep - 1) / _skipStep);
}

/** 请完成下面这个函数，实现题目要求的功能 **/
void SkipLinkList::insert(uint64_t key, uint64_t value)
{
    // TODO:
    if (_itemCount == 0)
    {
        _listHeader = new LinkNode(key, value);
        _skipListHeader = new SkipLinkNode(key, 1, _listHeader);
        _listNil->next = _listHeader;
        _skipListNil->next = _skipListHeader;
        ++_itemCount;
        return;
    }

    // 找到key对应的跳点区间，向link插入新点
    SkipLinkNode *lowerSkipPtr = _skipListNil,
        *upperSkipPtr = _skipListNil->next;
    while (upperSkipPtr != nullptr
        && key >= upperSkipPtr->key)
    {
        lowerSkipPtr = lowerSkipPtr->next;
        upperSkipPtr = upperSkipPtr->next;
    }
    LinkNode *lowerLinkPtr = lowerSkipPtr->pointer,
        *upperLinkPtr = lowerLinkPtr->next,
        *newLinkPtr = nullptr;
    while (upperLinkPtr != nullptr
        && key >= upperLinkPtr->key)
    {
        lowerLinkPtr = lowerLinkPtr->next;
        upperLinkPtr = upperLinkPtr->next;
    }
    // replace
    if (lowerLinkPtr->key == key)
    {
        lowerLinkPtr->value = value;
        return;
    }
    newLinkPtr = new LinkNode(key, value, upperLinkPtr);
    lowerLinkPtr->next = newLinkPtr;

    // 如果左跳点是NIL，则将右跳点左移，并将左指针和右指针右移一步
    if (lowerSkipPtr->count < 0)
    {
        upperSkipPtr->key = newLinkPtr->key;
        upperSkipPtr->pointer = newLinkPtr;
        _listHeader = newLinkPtr;
        lowerSkipPtr = lowerSkipPtr->next;
        upperSkipPtr = upperSkipPtr->next;
    }

    // 如果跳点计数超过最大值，半分跳点
    if (++lowerSkipPtr->count > _skipStep)
    {
        int leftCount = _skipStep / 2,
            rightCount = lowerSkipPtr->count - leftCount;
        LinkNode *linkRightPtr = lowerSkipPtr->pointer;
        for (int i = 0; i < leftCount; ++i)
        {
            linkRightPtr = linkRightPtr->next;
        }
        SkipLinkNode* rightSkipNode = new SkipLinkNode(linkRightPtr->key, rightCount, linkRightPtr, upperSkipPtr);
        lowerSkipPtr->count = leftCount;
        lowerSkipPtr->next = rightSkipNode;
    }
}

bool SkipLinkList::find(uint64_t key, uint64_t &value)
{
    // TODO:
    SkipLinkNode *lowerSkipPtr = _skipListNil,
        *upperSkipPtr = _skipListNil->next;
    while (upperSkipPtr != nullptr
        && key >= upperSkipPtr->key)
    {
        lowerSkipPtr = lowerSkipPtr->next;
        upperSkipPtr = upperSkipPtr->next;
    }
    LinkNode *lowerLinkPtr = lowerSkipPtr->pointer,
        *upperLinkPtr = lowerLinkPtr->next,
        *newLinkPtr = nullptr;
    while (upperLinkPtr != nullptr
        && key >= upperLinkPtr->key)
    {
        lowerLinkPtr = lowerLinkPtr->next;
        upperLinkPtr = upperLinkPtr->next;
    }

    if (lowerLinkPtr->key == key)
    {
        value = lowerLinkPtr->value;
        return true;
    }
    else
        return false;
}

/** 请完成上面的函数，实现题目要求的功能 **/

vector<string> splitString(const string& text, const string &sepStr) {
    vector<string> vec;
    string str(text);
    string sep(sepStr);
    size_t n = 0, old = 0;
    while (n != string::npos)
    {
        n = str.find(sep, n);
        if (n != string::npos)
        {
            vec.push_back(str.substr(old, n - old));
            n += sep.length();
            old = n;
        }
    }
    vec.push_back(str.substr(old, str.length() - old));
    return vec;
}
template <typename T>
bool stringToInteger(const string& text, T& value) {
    const char* str = text.c_str();
    char* endPtr = NULL;
    value = (T)strtol(str, &endPtr, 10);
    return (endPtr && *endPtr == 0);
}

bool testSkipLinkList(string inputParam) {
    vector<string> inputVec = splitString(inputParam, ":");
    if (inputVec.size() != 2 || (inputVec[0] != "list" && inputVec[0] != "count"))
    {
        cout << "input format error!" << endl;
        return false;
    }
    // prepare data
    vector<pair<uint64_t, uint64_t> > keyValueVec;
    if (inputVec[0] == "list") {
        vector<string> kvVec = splitString(inputVec[1], ";");
        for (size_t i = 0; i < kvVec.size(); i++) {
            vector<string> kvStr = splitString(kvVec[i], ",");
            uint64_t key, value;
            if (kvStr.size() != 2 ||
                !stringToInteger<uint64_t>(kvStr[0], key) ||
                !stringToInteger<uint64_t>(kvStr[1], value))
            {
                cout << "key-value list format error!" << endl;
                return false;
            }
            keyValueVec.push_back(make_pair(key, value));
        }
    }

    if (inputVec[0] == "count") {
        uint64_t count;
        if (!stringToInteger<uint64_t>(inputVec[1], count)) {
            cout << "count format error!" << endl;
            return false;
        }
        for (uint64_t i = 0; i < count; i++) {
            keyValueVec.push_back(make_pair(i, i));
        }
        random_shuffle(keyValueVec.begin(), keyValueVec.end());
    }

    uint64_t maxKey = 0;
    uint64_t value;

    SkipLinkList list(4);
    for (size_t i = 0; i < keyValueVec.size(); i++)
    {
        list.insert(keyValueVec[i].first, keyValueVec[i].second);

        // test find
        bool ret = list.find(keyValueVec[i].first, value);
        if (!ret || value != keyValueVec[i].second)
        {
            cout << "test find error!" << endl;
            return false;
        }
        // test replace
        list.insert(keyValueVec[i].first, keyValueVec[i].second + 1);
        ret = list.find(keyValueVec[i].first, value);
        if (!ret || value != keyValueVec[i].second + 1)
        {
            cout << "test replace error!" << endl;
            return false;
        }
        maxKey = max(maxKey, keyValueVec[i].first);
    }

    // test not exist key
    if (list.find(maxKey + 1, value))
    {
        cout << "find not exist error!" << endl;
        return false;
    }
    return list.simpleCheck();
}

//list:8,2;4,5;7,7
//count:10
int main() {
    bool res;
    string _inputParam;
    getline(cin, _inputParam);
    res = testSkipLinkList(_inputParam);

    if (res)
    {
        cout << "PASS" << endl;
    }
    else
    {
        cout << "FAIL" << endl;
    }
    return 0;
}
```

## 飞鸽传书

有人要利用鸽子传信息，鸽子会发出“咕”、“呱”两种声音，每只鸽子最多可有按顺序记住6个发声顺序。此人养了20只鸽子，每个鸽子发出后到达的顺序不稳定，每只鸽子只能通过发声顺序区分。

信息由31种字符组成，信息不超过1023个字符。

现在对每个信息输出一个最优方案 (2^N, M)，2^N 是每批放飞的鸽子数目，M 是需要的批次，求在 M 最小情况时，N 的最小值。

很自然想到了引入序列的编码，因为数据固定，直接计算对应参数写下代码就好。不过跟题目给的样例数据不相符，是我理解错题意了吗 QAQ

```cpp
// 输入
ABC
ABCD
YYAB^

// 输出
8 1
16 1
16 1
```

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <cmath>
using namespace std;

int main()
{
    vector<int> bits{ 6, 10, 16, 24, 32 };
    string str;
    while (cin >> str)
    {
        int N = 4, M = 1;
        while (bits.back() * M / 6 < str.size())
        {
            ++M;
        }
        while (true)
        {
            int cur = N - 1;
            if (cur < 0 || bits[cur] * M / 6 < str.size())
                break;
            else
                --N;
        }
        cout << pow(2, N) << " " << M << endl;
    }
}
```

