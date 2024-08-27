## auto

`auto`关键字在C++11以后用于自动推导变量的类型。

> 在C语言和C++11以前，auto是一个存储类说明符。

### auto的使用

`auto`在编译器进行自动类型推断，可以用于定义变量、函数返回值、模板参数等。注意`auto`仅仅作为占位符，并不会真正“智能”地推断类型，也不能用于函数参数、类成员变量等。

```cpp
auto i = 1; // auto -> int
auto d = 1.0; // auto -> double

std::vector<int> v = {1, 2, 3};
auto it = v.begin(); // auto -> std::vector<int>::iterator

for (auto& e : v) { // auto& -> int&
    std::cout << e << std::endl;
}
```

### auto原理

`auto` 的推断原理主要基于**模板类型推导**（Template Argument Deduction）机制。编译器在处理 `auto` 声明时，实际上是将其视为一个模板类型参数，然后应用模板参数推导规则。

编译器将 `auto` 声明视为一个假想的模板函数参数，初始化表达式则被视为传递给这个参数的实参。

例如：

```cpp
auto x = expr;
```

被视为：

```cpp
template<typename T>
void deduceType(T param);

deduceType(expr);
```

`auto`有以下规则：

1. 引用和`const`处理：
   - 对于`auto&`和`const auto&`，保留引用和`const`属性。
   - 对于普通的`auto`，会忽略引用，但保留`const`（除非显式声明为 `const auto`）。

2. 数组和函数退化：
   - 当用数组初始化`auto`变量时，推导为指针类型。
   - 当用函数名初始化时，推导为函数指针。

3. 特殊情况处理：
   - 对于初始化列表，`auto`默认推导为 `std::initializer_list<T>`。
   - 在C++17中，`auto`可以用于结构化绑定，此时会根据被绑定对象的成员类型进行推导。

4. 多个声明的一致性：
   在同一个声明语句中，如果使用多个`auto`，它们必须推导出相同的类型。

   ```cpp
   auto a = 1, b = 2;   // 正确, 都是 int
   auto c = 1, d = 2.0; // 错误，推导类型不一致
   ```

以上内容比较复杂，在实际应用时一般通过IDE的提示来辅助使用。

### 防止auto滥用

`auto`的使用可以简化代码，但也可能导致代码出现一些问题，例如：

```cpp
#include <iostream>  
#include <vector>  
#include <string>  

int main() {  
    std::vector<std::string> names = {"saber", "archer", "lancer"};  
  
    for (auto i = names.size(); i >= 0; --i) {  
        auto name = names[i];  
        auto length = name.length();  
        auto c = name[0];  
        
        std::cout << "Name: " << name << ", Length: " << length << ", First char: " << c << std::endl;  
    }  

    return 0;  
}
```

这段代码中，`auto`的使用使得代码可读性下降，维护者第一眼很难看出代码的意图。而且，这段代码也因为`auto`带来了一些很隐蔽的错误：

1. 变量`i`的类型被推导为`size_t`，而不是`int`，而`size_t`是一个无符号值，因此`i >= 0`永远为真，导致越界访问。
2. `name`的类型被推导为`std::string`，而`names[i]`实际上是`std::string&`，因此`name`是一个拷贝，而不是引用，导致性能损失。

更好的写法是：

```cpp
for (const auto& name : names) {  
    size_t length = name.length();  
    char c = name[0];  
    
    std::cout << "Name: " << name << ", Length: " << length << ", First char: " << c << std::endl;  
}
```

这样代码更加简洁，也更容易理解。