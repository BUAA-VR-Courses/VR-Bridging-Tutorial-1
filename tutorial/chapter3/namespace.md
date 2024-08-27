## 命名空间

### 什么是命名空间

在之前的例子中，有这么一段代码：

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

也有些人会这么写：

```cpp
#include <iostream>

using namespace std;

int main() {
    cout << "Hello, World!" << endl;
    return 0;
}
```

你可能已经注意到，有时我们会使用`using namespace std;`，或是在使用标准输出时写成`std::cout`，这里的std便是命名空间。

### 为什么需要命名空间

大型程序往往会使用多个独立开发的库，这些库又会定义大量的全局名字，如类名、函数名等。当应用程序用到标准库和第三方提供的库时，不可避免地会发生某些名字相互冲突的情况，这时编译器无法正确的找到我们需要使用的函数等符号。多个库将名字放置在全局命名空间中将引发**命名空间污染（namespace pollution）**。

例如你可能会自己实现一个`max`函数，但是标准库中也有一个`max`函数，它在`<algorithm>`头文件中，假如你这么写了一个程序：

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

int max(int a, int b) {
    return a > b ? a : b;
}

int main() {
    cout << max(1, 2) << endl;
    return 0;
}
```

那么编译器会报错，因为你将`std`命名空间进行了引入，但是你又定义了一个`max`函数，编译器不知道你到底是要调用你自己写的`max`函数还是标准库中的`max`函数。

传统的C语言中，程序员通过将其定义的全局实体名字设得很长来避免命名空间污染问题，这样的名字中通常包含表示名字所属单元的前缀部分。

比如：

```c
// Allocate a new process.
proc_t *proc_alloc();

// Create a user page table for a given process, with no user memory,
// but with trampoline and trapframe pages.
void proc_init_upt(proc_t *p);

// Initialize the user space
void proc_init_uvm(proc_t *proc, thread_t *init_td, const char *name, const void *bin, size_t size);
```

这三个函数都属于`proc`这个抽象单元中，后两个函数都用于初始化`proc`，这一功能。这样的命名方式虽然可以避免命名空间污染，但是对于程序员来说，书写和阅读这么长的名字过于繁琐，且不直观。

### 命名空间的使用

命名空间的定义使用`namespace`关键字，其后跟着命名空间的名字，然后是一对大括号，括号内是命名空间的内容。

在上面的例子中，我们可以整理相关命名空间如下：

```cpp
namespace proc {
    // Allocate a new process.
    proc_t *alloc();

    namespace init {
        // Create a user page table for a given process, with no user memory,
        // but with trampoline and trapframe pages.
        void upt(proc_t *p);

        // Initialize the user space
        void uvm(proc_t *proc, thread_t *init_td, const char *name, const void *bin, size_t size);
    }
}
```

这样我们就可以使用`proc::alloc()`、`proc::init::upt()`、`proc::init::uvm()`来调用这三个函数了。

如果需要精简代码，我们可以在保证不冲突的情况下适当使用`using`关键字：

```cpp
using namespace proc;

int main() {
    proc_t *p = alloc();
    init::upt(p);
    return 0;
}
```

一种更保险的做法是只引入需要的符号：

```cpp
using proc::alloc;
using proc::init::upt;

int main() {
    proc_t *p = alloc();
    upt(p);
    return 0;
}
```

命名空间(namespace)为防止名字冲突提供了更加可控的机制。命名空间分割了全局命名空间，其中每个命名空间是一个作用域。通过在某个命名空间中定义库的名字，库的作者以及用户可以避免全局名字固有的限制，就是说两个不同命名空间中的类、函数、变量可以有相同的名字。

> 注意不要在头文件中使用`using namespace`，因为头文件可能会被多个文件包含，这样极容易导致命名空间污染，尤其是在多人协作的项目中。
