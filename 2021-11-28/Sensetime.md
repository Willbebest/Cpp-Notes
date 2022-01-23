## 基础知识

#### 1、继承、封装、多态

- 封装：将抽象得到的数据和与数据相关的行为相结合到一起，形成类，其中数据和函数都是类的成员。优点是便面使用者直接使用对象的属性，只能根据开发者定义的函数来操作对象属性。保证了数据的安全、屏蔽细节，提高可维护性。

- 继承：可以扩展已存在的代码，目的也是为了代码重用。同时可以实现多态。

- 多态：调用的是同一个接口，但是效果各不相同。分为动态多态和静态多态

  - 动态多态：依赖虚函数和继承实现

    - 在父类中写一个虚函数，在子类中分别重写，用这个父类指针调用这个虚函数，它实际上会调用各自子类重写的虚函数。
    - 当某个类声明了虚函数时，编译器将为该类对象安插一个虚函数表指针，并为该类设置一张唯一的虚函数表，虚函数表中存放的是该类虚函数地址。
    - 在程序运行时才能确定函数和实现的链接，此时才能确定调用哪个函数，父类指针或者引用能够指向子类对象，调用子类的函数，所以在编译时是无法确定调用哪个函数
    - 缺点：运行期间进行虚函数绑定，提高了程序运行开销；编译器无法对虚函数进行优化。当虚函数被调用时，编译器会使用该对象中的虚指针来查找虚函数表，然后遍历虚函数表，以查找到虚函数的指针（地址），最终找到正确版本的函数。

  - 静态多态：重载、奇异的递归模板模式。在编译期就把函数链接起来，此时即可确定调用哪个函数或模板，静态多态是由模板和重载实现

    - 奇异递归模板：基类模板利用了其成员函数体（即成员函数的实现）在声明之后很久都不会被实例化（实际上只有被调用的模板类的成员函数才会被实例化），并利用了派生类的成员函数（通过[类型转化](https://zh.wikipedia.org/w/index.php?title=类型转化&action=edit&redlink=1)）。

      在下例中，`Base<Derived>::interface()`，虽然是在`struct Derived`之前就被声明了，但未被编译器实例化直至它被实际调用，这发生于`Derived`声明之后，此时`Derived::implementation()`的声明是已知的。

    ```cpp
      template <class T> 
      struct Base
      {
          void interface()
          {
              // ...
              static_cast<T*>(this)->implementation();
              // ...
          }
      
          static void static_func()
          {
              // ...
              T::static_sub_func();
              // ...
          }
      };
      
      struct Derived : Base<Derived>
      {
          void implementation();
          static void static_sub_func();
      };
    ```


#### 2、虚函数的调用机制

C++中使用虚指针和虚表来实现虚函数机制，虚指针指向虚表的起始地址，虚表中保存指向虚函数的指针（即地址）。使用时，需要:

- 需要有继承关系
- 基类有虚函数，子类对虚函数进行覆盖（override）
- 利用基类指针或对象指向派生类的对象，然后利用该指针调用虚函数

#### 3、重载、覆盖、重定义的区别

重载：在一个类中声明多个名称相同但参数列表不同的函数，这些参数可能个数、顺序或类型不同，但是不能靠返回类型来判断。

覆盖：子类重新定义基类的函数。函数名、参数列表（个数、类型和顺序）、返回值类型相同，**函数体存在区别**。

隐藏：不在同一个作用域（基类和子类），函数名相同。若仅仅是函数名或参数相同，返回类型不同，带有virtual，会发生错误。

#### 4、new和malloc的区别

- new在申请堆空间的时候指明空间保存的数据类型，并可以指明初始化的值，调用构造函数进行初始化。返回的是指定数据类型的指针，申请的空间需要使用delete释放。运算符
- malloc申请时需要指明申请空间大小作为参数，返回值是通用指针。申请的空间需要使用free释放。是函数
- new申请失败会报错，malloc申请失败返回空指针

#### 5、[c++](https://www.nowcoder.com/jump/super-jump/word?word=c%2B%2B)代码的性能优化

- 尽量使用C++11的右值语义，减少临时对象的构造。
- 简单的功能函数可以使用内联。少用继承，多用组合，尽量减少继承层级。
- 在循环遍历时，优化判断条件，减少循环次数。
- 优化堆内存的使用，如果有内存频繁的申请与释放，可以考虑内存池。

#### 6、内联的作用

在程序编译阶段，如果遇到内联函数，则将内联函数的实现在当前位置展开。 内联的目的是为了减少函数的调用开销，从而提高运行效率，但会增加代码体量。

#### 7、C++ 智能指针：是一个模板类，对指针进行封装，管理申请的堆空间，利用析构函数来释放申请的堆空间。

- unique_ptr: 不能被拷贝，只能通过`std::move`来转递它所指向的内存的所有权。

- shared_ptr: `std::shared_ptr`使用[引用计数](https://zh.wikipedia.org/wiki/引用计数)。每一个`shared_ptr`的拷贝都指向相同的内存。引用计数为零的时候，内存才会被释放。

  `std::shared_ptr`使用引用计数，所以有循环计数的问题。为了打破循环，可以使用`std::weak_ptr`.

- weak_ptr：是一个弱引用，只引用，不计数。不保证它指向的内存一定是有效的，在使用之前需要检查。

  ```cpp
  #include <iostream>
  #include <memory>
  using namespace std;
  
  class B;
  
  class A {
  public:
      ~A() {
          cout << "~A()" << endl;
      }
  
      shared_ptr<B> m_b;
  };
  
  class B {
  public:
      ~B() {
          cout << "~B()" << endl;
      }
  
      shared_ptr<A> m_a;
  };
  
  int main()
  {
      shared_ptr<A> a(new A);
      shared_ptr<B> b(new B);
      a->m_b = b;
      b->m_a = a;
  
  }
  ```

  

#### 8、C++ 类型转换

- static_cast：

  - 基本数据类型等之间的转换

  - 类型指针和通用指针之间的转换

  - 派生类对象指针转换为基类指针是安全的；基类指针转换为派生类，由于没有动态类型检查，所以是不安全的

- const_cast：去出const常量指针或引用的const修饰

- reinterpret_cast：

  - 改变指针或引用的类型
  - 将指针或引用转换为一个足够长度的整形
  - 将整型转换为指针或引用类型。
  
- dynamic_cast：
  - 其他三种都是编译时完成的，dynamic_cast是运行时处理的，运行时要进行类型检查。
  - 不能用于内置的基本数据类型的强制转换。
  - 使用dynamic_cast进行转换的，基类中一定要有virtual虚函数，否则编译不通过。
  - 子类指针指向父类指针（一般不会出问题），效果与static_cast一样；向下转换，即将父类指针转化子类指针，dynamic_cast具有类型检查的功能，比static_cast更安全。

#### 9、右值与左值的区别

右值是临时变量或字面值，生命后期仅限于当前表达式，并且不能取地址。

左值生命周期不限于一条表达式，可以取地址。

10、GDB 调试死锁的方法

- 使用参数-g编译空调是程序，然后运行终端
- 打开另一个终端，查询进程号：ps -aux | grep
- 进入gdb，切换到要调试的程序：attach 
- 查看线程信息：info threads
-  查看所有线程信息：thread apply all bt
- 找出有问题的线程后，切换到这个线程：thread
- 查看堆栈：bt
- 再查看函数栈：frame

11、GDB 调试程序崩溃

- 生成方法：

  ① 查看core文件大小：默认情况下core文件大小为 0。使用命令：ulimit -c

  ② 修改core文件大小：当大小为 0 时，需要修改：ulimit -c unlimited

    或ulimit -c 10000用于防止调试大量消耗内存的的

- 调试方法：gdb -c core ./main

12、





11、堆排序

13、CMAKE

14、设计模式



## 代码 

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

```cpp
#include <iostream>
#include <vector>
#include <stack>

using namespace std;

int Pattern(vector<int>& nums, int start, int back) {
    int val = nums[start];
    while (start < back) {
        while (start < back && nums[back] >= val) back--;
        if (start < back) nums[start] = nums[back];
        while (start < back && nums[start] <= val) start++;
        if (start < back) nums[back] = nums[start];
    }
    nums[start] = val;
    return start;
}
// 递归版本
void QuickSort(vector<int>& nums, int start, int back) {
    if (nums.empty() || start > back || start < 0 || back >= nums.size()) {
        return;
    }
    int mid = Pattern(nums, start, back);
    if (mid > start) QuickSort(nums, start, mid-1);
    if (mid < back)  QuickSort(nums, mid+1, back);
}
// 非递归版本
void QuickSort(vector<int>& nums) {
    if (nums.size() <= 1) return;
    int left = 0;
    int right = nums.size() - 1;
    stack<pair<int, int>> stk;
    stk.push({left, right});
    while (stk.size()) {
        int start = stk.top().first;
        int back = stk.top().second;
        stk.pop();
        int mid = Pattern(nums, start, back);
        if (mid > start) stk.push({start, mid-1});
        if (mid < back) stk.push({mid+1, back});
    }
}

int main()
{
    vector<int> nums{ 8,4,3,2,7,5,0,3,4,3,2 };
    // QuickSort(nums, 0, nums.size()-1);
    QuickSort(nums);
    for (auto elem : nums) {
        cout << elem << " ";
    }
    cout << endl;
    return 0;
}
```

时间复杂度：最快O(N logN)， 最慢O(logN*N)

空间复杂度：最快O(lgN)，最慢O(N)

***

#### 3、二维矩阵相乘

```cpp
using namespace std;

int MatrixMultiply(vector<vector<int>>& m1, vector<vector<int>>& m2,
    vector<vector<int>>& ans)
{
    ans.clear();
    if (m1.empty() && m2.empty()) {
        return 0;
    }
    if (m1.empty() || m2.empty() || m1[0].size() != m2.size()) {
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

int main()
{
    vector<vector<int>> m1{ {1, 2}, {2, 3} };
    vector<vector<int>> m2{ {1, 4}, {4, 5} };

    vector<vector<int>> rets;
    int ret = MatrixMultiply(m1, m2, rets);
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

****

#### 4、获取 Top K

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <unordered_map>

using namespace std;

using PII = pair<int, int>;

vector<int> FindTopK(vector<int>& nums, int k) {
    unordered_map<int, int> mp;
    for (auto elem : nums) {
        mp[elem]++;
    }
    priority_queue<PII, vector<PII>, greater<PII>> pq;
    for (auto elem : mp) {
        pq.push({elem.second, elem.first});
        if (pq.size() > k) {
            pq.pop();
        }
    }

    vector<int> ans;
    while (pq.size()) {
        ans.insert(ans.begin(), pq.top().second);
        pq.pop();
    }

    return ans;
}

int main()
{
    vector<int> nums{ 1, 2, 3, 2, 2, 2, 1, 1, 1, 1, 3, 4, 5, 8, 9 };
    int k = 2;
    vector<int> ans = FindTopK(nums, k);
    for (auto elem : ans) {
        cout << elem << " ";
    }
    cout << endl;

    return 0;
}
```

5、智能指针 shared_ptr 实现

```cpp
#include <iostream>

using namespace std;

template <typename T>
class SharedPtr {
public:
    SharedPtr(T* ptr) {
        if (ptr) {
            pData = ptr;
            pCount = new size_t(1);
        }
    }

    void IncreaseCount() {
        *pCount++;
    }

    void DescreaseCount() {
        *pCount--;
    }

    size_t Use_count() const {
        return *pCount;
    }

    T* get() {
        return pData;
    }

    T* operator->() {
        return pData;
    }

    T& operator*() {
        return *pData;
    }

    ~SharedPtr() {
        DescreaseCount();
        if (Use_count() == 0) {
            delete pData;
            delete pCount;
        }
    }

private:
    T* pData {nullptr};
    size_t* pCount {nullptr};
};

int main() {
    SharedPtr<int> a(new int(1));
    SharedPtr<int> a2{ a };

    cout << *a << " " << *a2 << endl;

    return 0;
}
```











