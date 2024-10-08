## STL

C++标准模板库提供了一系列的容器、算法和迭代器，称为STL（Standard Template Library）。STL的设计目标是提供一组通用的数据结构和算法，以便开发者能够更加高效地编写独立于数据类型的各种代码。

正是因为STL的泛型设计，我们将STL的介绍放在了本章的最后一节。在学习了C++的基础知识之后，我们可以更好地理解STL的设计思想。

这里仅仅是“是什么”程度的介绍，具体到每个容器或是算法的使用，请参考[C++ reference](https://en.cppreference.com/w/)。

### 组件

STL的设计理念是将数据和算法分离，通过迭代器作为桥梁来连接它们。这种设计使得STL具有高度的可复用性和灵活性。

STL主要由以下几个组件组成：

1. 容器（containers）
    - 各种数据结构，如`vector`、`list`、`deque`、`set`、`map`。可以理解为包含对象、具有一定通用功能的模板类。
2. 算法（algorithms）
    - 各种常用算法如`sort`、`search`、`copy`、`erase`。适用于不同类型容器的函数。
3. 迭代器（iterators）
    - 容器中的“指针”，用于遍历容器中的元素。
4. 仿函式（functors）
    - STL包括重载函数调用操作符的类。类的实例称为函数对象或仿函数。函数允许在传递参数的帮助下自定义相关函数的工作。
5. 适配器（adapters）
    - 修饰容器（containers）、仿函式（functors）或迭代器（iterators）。
6. 分配器（allocators）
    - 负责空间分配与管理，即用于分配空间的对象。

有些组件之间的界限并没有那么清晰，在这里我们将介绍前3个组件。

### 容器

STL中的容器可以大致分为两类：有序容器和无序容器。有序容器中的元素按照特定的顺序排列，而无序容器则不保证元素的顺序。

有序容器的一个典型例子是`std::vector`。`std::vector`是一个动态数组，它在内存中连续存储元素，支持随机访问，并且可以在尾部快速插入和删除元素。例如：

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};
vec.push_back(6);  // 在尾部添加元素，或emplace_back
int third_element = vec[2];  // 随机访问
```

`std::vector`使用连续的内存布局，这使得它在遍历和访问元素时非常高效。然而，在中间插入或删除元素可能会导致后续元素的移动，从而影响性能。

相比之下，无序容器（如`std::unordered_set`和`std::unordered_map`）使用哈希表来组织数据。这些容器不保证元素的顺序，但它们提供了常数时间复杂度的插入、删除和查找操作。无序容器特别适合需要快速查找和不关心元素顺序的场景。

### 迭代器

迭代器是STL的另一个核心概念，它提供了一种统一的方式来访问不同类型的容器。迭代器抽象了元素的访问方式，使得算法可以独立于具体的容器类型工作。例如，我们可以使用相同的方式遍历`std::vector`和`std::list`：

```cpp
for (auto it = container.begin(); it != container.end(); ++it) {
    // 使用 *it 访问元素
}
```

迭代器分为几种不同的类型，包括输入迭代器、输出迭代器、前向迭代器等。这些迭代器提供了不同的功能和性能保证，我们可以根据需要选择合适的迭代器类型。

### 算法

STL提供了大量的算法，包括查找、排序、合并、删除等。这些算法通常接受迭代器作为参数，因此它们可以用于不同类型的容器。

例如，我们可以使用`std::sort`对`std::vector`进行排序：

```cpp
std::vector<int> vec = {3, 1, 4, 1, 5, 9};
std::sort(vec.begin(), vec.end());
```

这表示对`vec`中的元素进行排序。`std::sort`接受两个迭代器作为参数，分别指向排序的起始和结束位置。在这个例子中，`vec.begin()`返回指向第一个元素的迭代器，`vec.end()`返回指向最后一个元素的下一个位置的迭代器。

当然，我们也可以自定义比较函数来实现不同的排序方式：

```cpp
std::sort(vec.begin(), vec.end(), [](int& a, int& b) {
    return a > b;  
});
```

这里我们使用了一个lambda表达式作为比较函数，表示按照降序排序。

### STL的错误

STL的设计是非常优秀的，但它因为一些历史原因，遗留了一些非常难绷的问题。

比如`std::string`，其并没有考虑字符编码问题，并且经常导致内存的额外拷贝，它甚至没有实现字符串的分割和连接操作。在传递`std::string`时，我们通常使用`const std::string&`，以避免不必要的拷贝。这么做非常麻烦，而且容易出错。

> 基于字符串在大多情况下是不可变的前提，一个理想的字符串设计应该是轻量的，只需要存储一个指向字符数组的指针，然后维护一个长度，比如C++17中的`std::string_view`。

还有`std::vector<bool>`，它并不是一个真正的STL容器。为了节省空间，`std::vector<bool>`将多个bit打包存储在一个字节中。这导致了与其他`std::vector`类型不同的行为。在引用、迭代和多线程等场景下，都会出现非预期的问题。而且虽然节省了空间，但是访问时并不会更快。

> 一般来说用`std::vector<int>`就行。