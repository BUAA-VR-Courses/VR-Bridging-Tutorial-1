## 引用

### 什么是引用

在C语言中不存在引用的概念。引用变量是一个别名，也就是说，它是某个已存在变量的另一个名字。一旦把引用初始化为某个变量，就可以使用该引用名称或变量名称来指向变量。

可以通过`&`符号来创建引用。例如：

```cpp
#include <iostream>
// T& ref = var;

int main() {
    int a = 10;
    int &b = a;

    b = 20;
    std::cout << "a = " << a << std::endl;
    std::cout << "b = " << b << std::endl;
 
    return 0;
}
```

运行可以观察到变量`a`的值被改变为20。

引用很容易与指针混淆，它们之间有三个主要的不同：
- 不存在空引用。引用必须连接到一块合法的内存。
- 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象。
- 引用必须在创建时被初始化。指针可以在任何时间被初始化。

### 为什么使用引用

在函数形参中使用引用，在函数调用时更符合编程习惯。

使用指针：

```cpp
void foo(int *a) {
    // ...
}

std::cin >> a;
foo(&a);
```

使用引用：

```cpp
void foo(int &a) {
    // ...
}

std::cin >> a;
foo(a);
```

引用通常用于函数参数传递，可以避免拷贝大对象的开销。例如：

```cpp
#include <iostream>
#include <vector>

void print(const std::vector<int>& v) {
    for (auto i : v) {
        std::cout << i << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    print(v);

    return 0;
}
```

在遍历容器时，使用引用可以避免拷贝的开销。

```cpp
#include <iostream>
#include <vector>

struct A {
    int a[1000];
};

int main() {
    std::vector<A> v(100, A());
    // ... 赋值v
    for (int& i : v) {
        // ... 操作v
    }
    return 0;
}
```

> 在Java中，所有对象都是通过引用来访问的。

### 悬垂引用

有如下场景：

```cpp
int& foo() {
    int a = 10;
    return a;
}

int main() {
    int& b = foo();
    std::cout << b << std::endl;
    return 0;
}
```

在上面场景中，这个引用的对象的生命周期已经结束（局部变量），但我们仍然试图访问它，这个引用就是悬垂引用。访问悬垂引用是一种很严重的未定义行为。

虽然指针也会产生悬垂的问题，但是引用的悬垂会更加隐蔽，在使用引用时要注意这个问题。