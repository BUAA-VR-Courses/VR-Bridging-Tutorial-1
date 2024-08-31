## 模板

C++模板是一种强大的泛型编程工具，允许程序员编写与类型无关的通用代码。模板的核心思想是"一次编写，多次使用"，即通过一个模板可以生成多个具体的函数或类，从而提高代码的复用性和灵活性。

有函数模板和类模板两种类型。

### 函数模板

函数模板允许我们定义一个通用函数，可以处理不同类型的参数。

```cpp
template <typename T>
T functionName(T parameter1, T parameter2, ...) {
    // 函数体
}
```
比如：

```cpp
#include <iostream>

template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    std::cout << "Max of 3 and 7: " << max(3, 7) << std::endl;
    std::cout << "Max of 3.14 and 2.72: " << max(3.14, 2.72) << std::endl;
    std::cout << "Max of 'a' and 'z': " << max('a', 'z') << std::endl;
    return 0;
}
```

在这个例子中，`max`函数模板可以用于比较整数、浮点数和字符。

### 类模板

类模板允许我们定义一个通用类，可以用不同的数据类型来实例化。

```cpp
template <typename T>
class ClassName {
    // 类定义
};
```

比如：

```cpp
#include <iostream>

template <typename T>
class Box {
private:
    T content;

public:
    Box(T content) : content(content) {}
    
    T getContent() {
        return content;
    }
    
    void setContent(T newContent) {
        content = newContent;
    }
};

int main() {
    Box<int> intBox(42);
    std::cout << "Int box contains: " << intBox.getContent() << std::endl;
    
    Box<std::string> stringBox("Hello, Templates!");
    std::cout << "String box contains: " << stringBox.getContent() << std::endl;
    
    return 0;
}
```

在这个例子中，`Box`类模板可以用于存储不同类型的数据。

### 模板特化

模板特化允许我们为特定类型提供自定义实现。

```cpp
#include <iostream>
#include <string>

template <typename T>
T absolute(T x) {
    return x < 0 ? -x : x;
}

// 为 std::string 类型特化
template <>
std::string absolute(std::string x) {
    return x;
}

int main() {
    std::cout << "Absolute of -5: " << absolute(-5) << std::endl;
    std::cout << "Absolute of -3.14: " << absolute(-3.14) << std::endl;
    std::cout << "Absolute of 'Hello': " << absolute(std::string("Hello")) << std::endl;
    return 0;
}
```

在这个例子中，我们为`std::string`类型提供了一个特化版本的`absolute`函数。

> 模板可以大大简化代码实现，并且没有额外的性能开销。