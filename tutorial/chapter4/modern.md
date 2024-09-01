## Modern C++

C++的原始要求之一是与C语言向后兼容，因此，C++始终允许C样式编程，其中包含原始指针、数组、以0结尾的字符串和其他功能。 它们可以实现良好的性能，但也可能会引发bug并增加复杂性。 

现代C++的发展趋势就是逐渐取代C语言风格的编程，使用更安全、更易读的代码。这种风格的代码通常使用标准库中的容器、智能指针、字符串和算法，以及新的语言特性，如范围for循环、auto关键字、lambda表达式等。

### 智能指针

C样式编程的一个主要bug类型是内存泄漏。RAII的观念在之前已经介绍过了。

为了支持对RAII原则的简单采用，C++标准库提供了三种智能指针类型：`std::unique_ptr`、`std::shared_ptr`和`std::weak_ptr`。

```cpp
#include <memory>
class widget
{
private:
    std::unique_ptr<int[]> data;
public:
    widget(const int size) { data = std::make_unique<int[]>(size); }
    void do_something() {}
};

void functionUsingWidget() {
    widget w(1000000);  // lifetime automatically tied to enclosing scope
                        // constructs w, including the w.data gadget member
    // ...
    w.do_something();
    // ...
} // automatic destruction and deallocation for w and w.data
```

`widget`类中包含一个数组成员，该成员是在调用`make_unique()`时在堆上分配的。对`new`和 `delete`的调用将由`std::unique_ptr`类封装。当一个`widget`对象超出生命范围时，将调用自动调用`unique_ptr`析构函数，此函数将释放为数组在堆上分配的内存。

### for循环

C++11引入了范围for循环，它允许我们遍历容器中的元素，而不必担心索引或迭代器。

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v {1,2,3};

    // C-style
    for(int i = 0; i < v.size(); ++i)
    {
        std::cout << v[i] << " ";
    }

    // Modern C++:
    for(auto& num : v)
    {
        std::cout << num << " ";
    }
}
```

### constexpr

`constexpr`是C++11引入的关键字，用于在编译时计算表达式的值。`constexpr`函数是在编译时执行的函数，它们可以编译期计算。

在传统C语言中，const有两种含义，“只读”和“常量”。在现代C++中，希望`const`关键字的含义更接近“只读”，而`constexpr`关键字则更接近“常量”。

用`constexpr`定义的变量替代宏定义，可以提高代码的可读性，避免IDE的错误提示，也可以让代码更容易维护。

```cpp
// C-style
#define N 10
int arr[N];

// Modern C++
constexpr int N = 10;
int arr[N];
```

`constexpr`函数可以在编译时计算表达式的值，这样可以提高程序的性能。

```cpp
constexpr int factorial(int n) {
    return n <= 1 ? 1 : (n * factorial(n - 1));
}
```

### 移动语义

C++11引入了移动语义，它允许将资源从一个对象转移到另一个对象，而不是进行深拷贝。这样可以提高程序的性能。

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v1 {1, 2, 3};
    std::vector<int> v2 = std::move(v1);

    std::cout << "v1 size: " << v1.size() << std::endl; // 0
    std::cout << "v2 size: " << v2.size() << std::endl; // 3
}
```

C++的`std::move`与Rust不同，它本质上是一个向右值引用的类型转换函数，用于将左值转换为右值引用。它其实不会移动任何东西，也不会让原本对象失效。

### Lambda表达式

在C风格编程中，可以通过使用函数指针将函数传递到另一个函数。函数指针不便于维护和理解。它们引用的函数可能是在源代码的其他位置中定义的，而不是从调用它的位置定义的。此外，它们不是类型安全的。

在C++中，可以使用lambda表达式来解决这个问题。lambda表达式是一个匿名函数，可以在需要时定义并传递给其他函数。

```cpp
std::vector<int> v {1,2,3,4,5};
int x = 2;
int y = 4;
auto result = find_if(begin(v), end(v), [=](int i) { return i > x && i < y; });
```

Lambda表达式的语法如下：

```cpp
[capture clause] (parameters) -> return-type { function body }
```

- `capture clause`：捕获列表，用于捕获外部变量，可以是值传递（=）或引用传递（&）。
- `parameters`：参数列表。
- `return-type`：返回类型。
- `function body`：函数体。

### 异常

异常处理是一种错误处理机制，它允许程序在运行时检测到错误并采取适当的措施。异常处理是一种更安全、更优雅的错误处理机制，它可以使代码更加健壮。

> 慎用C++的异常，它会影响程序的性能。当然，在绝大多数情况下你不需要那么高的性能。

```cpp
#include <iostream>
#include <stdexcept>

int divide(int a, int b) {
    if (b == 0) {
        throw std::invalid_argument("division by zero");
    }
    return a / b;
}

int main() {
    try {
        int result = divide(10, 0);
        std::cout << "Result: " << result << std::endl;
    } catch (const std::invalid_argument& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }
}
```

异常处理的基本原则是：在可能引发异常的代码块中使用`try`块，然后在`catch`块中处理异常。如果没有异常被抛出，`catch`块将被跳过。

C++11中引入了`noexcept`关键字，用于指示函数是否可能引发异常。如果函数不会引发异常，可以使用`noexcept`关键字来提高程序的性能，并且可以阻止异常的扩散传播，让编译器最大限度地优化代码。

```cpp
int divide(int a, int b) noexcept {
    if (b == 0) {
        return 0;
    }
    return a / b;
}
```

### nullptr

C语言中使用`NULL`表示空指针，但它实际上是一个整数。`NULL`往往被宏定义为`0`，但这可能会导致很多问题。

C++11引入了`nullptr`关键字，用于表示空指针。`nullptr`是一个特殊的空指针常量，它可以隐式转换为任何指针类型。其类型为`std::nullptr_t`。

```cpp
int* p = nullptr;
```

### 展望

一般来说，现代C++是指C++11及以后的版本。从现在的角度来看也已经不那么“现代”了。后续C++还引入了包括`constexpr if`、`std::optional`、`std::variant`等功能，C++20还引入了概念、协程等新特性。总体上C++是一门还在不断发展的语言。

我们也不需要过度追求“现代”C++，更重要的是完成功能，选择合适的特性使用即可。
