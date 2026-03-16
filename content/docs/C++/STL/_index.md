---
title: "3.3.STL"
weight: 1
bookCollapseSection: true
---

# STL 简介

**STL（Standard Template Library）** 是 C++ 标准库中最重要的一部分，它提供了一组高效、通用、可复用的数据结构和算法组件。通过模板（Template）机制，STL 将常见的数据结构与算法抽象为通用组件，使程序员能够在不重复造轮子的情况下快速构建高性能程序。

STL 的设计核心是 **泛型编程（Generic Programming）**：算法与数据结构相互独立，通过迭代器（Iterator）连接，从而实现高度的通用性和可扩展性。

通常来说，STL 主要由以下几个部分组成：

* **容器（Containers）**：用于存储数据，如 `vector`、`list`、`map`、`set` 等。
* **算法（Algorithms）**：对数据进行操作，如 `sort`、`find`、`copy` 等。
* **迭代器（Iterators）**：用于遍历容器元素，充当容器与算法之间的桥梁。
* **函数对象（Function Objects）**：可像函数一样调用的对象，常用于自定义算法行为。
* **适配器（Adapters）**：对已有组件进行封装或转换，如 `stack`、`queue` 等。

STL 的出现极大地提升了 C++ 的开发效率，并成为现代 C++ 编程的基础工具之一。熟练掌握 STL，不仅能够减少重复代码，还能写出更加简洁、高效和可靠的程序。



## 顺序容器类型

| 容器名称       | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| `std::vector`  | 可变大小数组。支持快速随机访问。在尾部之外的位置插入/删除元素可能很慢 |
| `std::deque`   | 双端队列。支持快速随机访问。在头部/尾部位置插入/删除速度很快 |
| `std::list`    | 双向链表。只支持双向顺序访问。在list中任何位置进行插入/删除操作速度都很快 |
| `forward_list` | 单向链表。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快 |
| `std::array`   | 功能上和原生数组几乎一样(固定大小，内存连续)。区别：原生数组传给函数后退化为指针，丢失长度信息 |
| `std::string`  | 与vector相似的容器，但专门用于保存字符。随机访问快。在尾部插入/删除速度快 |



## 关联容器类型



## 迭代器

### 一、迭代器的本质

迭代器本质上是**对指针的抽象**。原生指针天然就是一种迭代器——它指向一块内存，可以解引用、可以递增移动到下一个元素。STL 把这个概念泛化了，让不同的数据结构都能通过统一的接口来访问元素，而不需要暴露内部实现。

```cpp
// 原生指针就是最原始的迭代器
int arr[] = {1, 2, 3, 4, 5};
int* p = arr;
++p;        // 移动到下一个元素
*p;         // 解引用，得到 2

// STL 迭代器做的事情一模一样
std::vector<int> v = {1, 2, 3, 4, 5};
auto it = v.begin();
++it;       // 移动到下一个元素
*it;        // 解引用，得到 2
```



**使用迭代器遍历一个vector：**

```cpp
std::vector<int> v = {1,2,3,4};

for(auto it = v.begin(); it != v.end(); ++it)
{
    std::cout << *it << std::endl;
}
```



这个抽象的意义在于：算法不需要知道数据存在数组里还是链表里还是树里，只要通过迭代器就能统一操作。`std::sort` 能排序 `vector`，也能排序原生数组，就是因为它们的迭代器都支持随机访问。



**常用方法**

| 方法                    | 方向 | 可修改 | 说明           |
| ----------------------- | ---- | ------ | -------------- |
| `begin()` / `end()`     | 正向 | 是     | 最常用         |
| `cbegin()` / `cend()`   | 正向 | 否     | 只读遍历       |
| `rbegin()` / `rend()`   | 反向 | 是     | 从尾到头       |
| `crbegin()` / `crend()` | 反向 | 否     | 从尾到头，只读 |



### 二、迭代器的五种类别

STL 把迭代器按能力强弱分成五类，从弱到强依次是：

**1.输入迭代器(Input Iterator)**

只能单次前向遍历，只读。读过的位置不能再读。

典型代表：`std::istream_iterator`

```cpp
#include <iterator>
#include <iostream>

// 从标准输入逐个读取 int
std::istream_iterator<int> in(std::cin);
std::istream_iterator<int> eof;  // 默认构造就是结束哨兵

while (in != eof) {
    std::cout << *in << std::endl;
    ++in;
}
```

实际项目中很少直接用，但理解它有助于理解算法对迭代器的最低要求。



**2.输出迭代器(Output Iterator)**

只能单次前向写入。

典型代表：`std::ostream_iterator`、`std::back_inserter`

```cpp
#include <iterator>
#include <algorithm>

std::vector<int> src = {1, 2, 3};
std::vector<int> dst;

// back_inserter 返回一个输出迭代器，每次赋值就 push_back
std::copy(src.begin(), src.end(), std::back_inserter(dst));
// dst: {1, 2, 3}

// 输出到标准输出，用逗号分隔
std::copy(src.begin(), src.end(), std::ostream_iterator<int>(std::cout, ", "));
// 输出: 1, 2, 3,
```

`back_inserter` 在实际项目里用得很多，配合算法做容器间的数据搬运非常方便。



**3.前向迭代器(Forward Iterator)**

可以多次前向遍历，读写都行，可以保存位置反复访问。

典型代表：`std::forward_list::iterator`、`std::unordered_map::iterator`

```cpp
std::forward_list<int> fl = {1, 2, 3};
auto it = fl.begin();
auto save = it;   // 可以保存位置
++it;
*save;             // 还能回去访问，得到 1
// 但是不能 --it，只能往前走
```



**4.双向迭代器(Bidirectional Iterator)**

在前向的基础上可以往回走。

典型代表：`std::list::iterator`、`std::map::iterator`、`std::set::iterator`

```cpp
std::list<int> lst = {1, 2, 3, 4, 5};
auto it = lst.end();
--it;       // 可以往回走
*it;        // 5
--it;
*it;        // 4

// 反向遍历
for (auto it = lst.rbegin(); it != lst.rend(); ++it) {
    std::cout << *it << " ";
}
// 输出: 5 4 3 2 1
```



**5.随机访问迭代器(Random Access Iterator)**

最强的迭代器，支持所有指针运算：加减整数、下标访问、迭代器之间求距离和比较大小。

典型代表：`std::vector::iterator`、`std::deque::iterator`、原生指针

```cpp
std::vector<int> v = {10, 20, 30, 40, 50};
auto it = v.begin();

it += 3;          // 直接跳到第4个元素
*it;              // 40

it - v.begin();   // 3，迭代器间距离

it[1];            // 50，下标访问

it > v.begin();   // true，可以比较大小
```

**为什么这个分类重要？** 因为不同的算法对迭代器有不同的最低要求：

| 算法           | 要求的迭代器   | 原因                   |
| -------------- | -------------- | ---------------------- |
| `std::find`    | 输入迭代器     | 只需要逐个往前看       |
| `std::copy`    | 输入+输出      | 一边读一边写           |
| `std::reverse` | 双向迭代器     | 需要从两头往中间走     |
| `std::sort`    | 随机访问迭代器 | 需要跳跃访问和比较位置 |

这就是为什么 `std::list` 不能用 `std::sort`（`list` 是双向迭代器，达不到随机访问的要求），但 `list` 自己提供了成员函数 `lst.sort()`。



### 三、迭代器失效

这是实际开发中**最容易踩坑**的地方，也是面试高频题。

**vector 的迭代器失效**

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// 情况1：插入导致扩容，所有迭代器失效
auto it = v.begin() + 2;
v.push_back(6);
// 如果发生了扩容，it 已经是野指针了，用它就是未定义行为
// *it;  // 危险！

// 情况2：erase 之后，被删位置及其后面的迭代器全部失效
auto it2 = v.begin() + 2;
v.erase(it2);
// it2 失效了，但 erase 返回了下一个有效迭代器
```

第一种情况**不能“自动解决”**，只能**通过设计规避**。

第二种情况的解决方法：

```cpp
std::vector<int> v = {1, 2, 3, 2, 4, 2, 5};

// ❌ 错误：erase 后 it 失效，++it 是未定义行为
//规则：erase 会使 被删除位置及其之后的所有迭代器失效
for (auto it = v.begin(); it != v.end(); ++it) {
    if (*it == 2) {
        v.erase(it);  // it 失效了，下一次 ++it 就炸了
    }
}

// ✅ 正确：用 erase 的返回值
for (auto it = v.begin(); it != v.end(); ) {
    if (*it == 2) {
        it = v.erase(it);  // vector::erase 返回的是“被删除元素后面的那个元素的新迭代器”。
    } else {
        ++it;
    }
}

// ✅ 更好：直接用 erase-remove 惯用法
v.erase(std::remove(v.begin(), v.end(), 2), v.end());

// ✅ C++20 最简洁
std::erase(v, 2);
```



**map / set 的迭代器失效**

关联容器友好得多：`erase` 只会让被删除元素的迭代器失效，其他元素不受影响。

```cpp
std::map<int, std::string> m = {{1, "a"}, {2, "b"}, {3, "c"}, {4, "d"}};

// ✅ C++11 起可以这样写
for (auto it = m.begin(); it != m.end(); ) {
    if (it->first % 2 == 0) {
        it = m.erase(it);
    } else {
        ++it;
    }
}

// ✅ C++20
std::erase_if(m, [](const auto& pair) {
    return pair.first % 2 == 0;
});
```



**unordered_map / unordered_set 的迭代器失效**

和 `map`/`set` 类似，`erase` 只影响被删除的那个迭代器。但**插入**操作可能触发 rehash（哈希表扩容），此时所有迭代器都会失效。
