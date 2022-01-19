## Sensetime 

#### 1. [删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

**【题目描述】**

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

**示例一：**

> 处理前：		①=>②=>③=>④=>⑤  
>
> 处理后变成： ①=>②=>③====>⑤ 
>
> 输入：head = [1,2,3,4,5], n = 2
>
> 输出：[1,2,3,5]

示例二：

> 输入：head = [1], n = 1
>
> 输出：[]

示例三：

> 输入：head = [1,2], n = 1
>
> 输出：[1]

**提示：**

> - 链表中结点的数目为 `sz`
> - `1 <= sz <= 30`
> - `0 <= Node.val <= 100`
> - `1 <= n <= sz`

**【解法】**

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* pSlow = new ListNode(-1, head);
        ListNode* pFast = head;
        ListNode* pRet = pSlow;
		// 快指针先跑n个节点
        for(int i=1; i<n; i++) {
            if(pFast==nullptr) return pRet->next;
            pFast = pFast->next;
        }

        while(pFast->next) {
            pFast = pFast->next;
            pSlow = pSlow->next;
        }
        pSlow ->next = pSlow->next->next;

        return pRet->next;
    }
};
```

*****

#### 2. 快排算法



3、C++ 类型转换

4、C++ const关键字

5、C++ 虚函数的作用

6、C++ 智能指针

7、GDB

8、[c++](https://www.nowcoder.com/jump/super-jump/word?word=c%2B%2B)代码的性能优化

- 尽量使用C++11的右值语义，减少临时对象的构造。
- 简单的功能函数可以使用内联。少用继承，多用组合，尽量减少继承层级。
- 在循环遍历时，优化判断条件，减少循环次数。
- 优化堆内存的使用，如果有内存频繁的申请与释放，可以考虑内存池。

9、内联的作用

在程序编译阶段，如果遇到内联函数，则将内联函数的实现在当前位置展开。 内联的目的是为了减少函数的调用开销，从而提高运行效率，但会增加代码体量。

10、矩阵转置

11、快排和堆排序

12、矩阵乘法、TOP K

13、CMAKE

14、设计模式





智能指针实现：

```cpp
#include <iostream>

using namespace std;

template<typename T>
class SharedPtr
{
public:
    SharedPtr(T data)
        : refCount(new int(1)), pData(new T(data))
    {}

    SharedPtr(SharedPtr& other)
    {
        other.IncreaseRefCount();
        refCount = other.refCount;
        pData = other.pData;
    }

    void IncreaseRefCount()
    {
        ++*refCount;
    }

    void DecreaseRefCount()
    {
        --*refCount;
    }

    int GetCount() const {
        return *refCount;
    }

    T* GetData() const {
        return pData;
    }

    ~SharedPtr()
    {
        DecreaseRefCount();
        if (GetCount() == 0) {
            delete pData;
            delete refCount;
        }
    }

private:
    int *refCount;
    T* pData;
};

int main() {
    SharedPtr<int> a(1);
    SharedPtr<int> a2{ a };

    cout << *a.GetData() << " " << *a2.GetData();

    return 0;
}
```



find the K most occurred elements of an array

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
#include <queue>

using namespace std;

typedef pair<int, int> PII;
// find the K most occurred elements of an array
vector<int> kElementsWithHighestFrequency(vector<int>& nums, int k) {
    unordered_map<int, int> freq;

    for (auto x : nums) {
        ++freq[x];
    }

    priority_queue<PII, vector<PII>, greater<PII>> pq;

    for (auto elem : freq) {
        pq.emplace(elem.second, elem.first);
        if (pq.size() > k) {
            pq.pop();
        }
    }

    vector<int> rets;

    while (!pq.empty()) {
        rets.push_back(pq.top().second);
        pq.pop();
    }

    return rets;
}

int main() {

    vector<int> input{ 1, 2, 3, 2, 2, 2, 1, 1, 1, 1, 3, 4, 5, 8, 9 };

    auto rets = kElementsWithHighestFrequency(input, 2);

    for (auto x : rets) {
        cout << x << " ";
    }

    return 0;
}
```



矩阵相乘

```cpp
#include <iostream>
#include <vector>

using namespace std;

int matrixMultiply(vector<vector<int>>& m1, vector<vector<int>> m2, vector<vector<int>>& ans) {
    ans = {};
    if (m1.empty()&&m2.empty()) {
        return 0;
    }
    if (m1.empty() || m2.empty()) {
        return -1;
    }
    if (m1[0].size() != m2.size()) {
        return -1;
    }
    int m = m1.size();
    int n = m2[0].size();
    int t = m2.size();
    ans.resize(m, vector<int>(n, 0));
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            for (int k = 0; k < t; k++) {
                ans[i][j] += m1[i][k] * m2[k][j];
            }
        }
    }

    return 0;
}

int main() {
    vector<vector<int>> m1{ {1, 2}, {2, 3} };
    vector<vector<int>> m2{ {1, 4}, {4, 5} };

    vector<vector<int>> rets;
    int ret = matrixMultiply(m1, m2, rets);
    if (ret) cout << "failed";
    else {
        for (auto& row : rets) {
            for (auto x : row) {
                cout << x << " ";
            }
            cout << endl;
        }
    }

    return 0;
}
```

