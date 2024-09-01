## RAII

RAII（Resource Acquisition Is Initialization）是一种编程思想，它是一种资源管理的方法，通过在对象的构造函数中申请资源，在析构函数中释放资源，从而保证资源的正确管理。

RAII的核心思想是：资源的生命周期与对象的生命周期绑定。

### 关注资源的生命周期

我们先来看一个简化的例子：

比如我现在有一个网络协议的控制块：

```cpp
struct ControlBlock {
    uint32_t ipaddr;
    uint16_t port;
    uint32_t metric;
};
```

我现在要将其序列化到一个字符串中：

```cpp
struct ControlBlock {
    uint32_t ipaddr;
    uint16_t port;
    uint32_t metric;
    
    char *to_packet() {
        char *packet = new char[12];
        memcpy(packet, &ipaddr, 4);
        memcpy(packet + 4, &port, 2);
        memcpy(packet + 6, &metric, 4);
        return packet;
    }
};
```

然后我在使用中去获取这个控制块，并且进行序列化：

```cpp
void func() {
    ControlBlock cb;
    cb.ipaddr = 0x7f000001;
    cb.port = 80;
    cb.metric = 100;
    
    char *packet = cb.to_packet();
    
    // do something with packet
    
    delete[] packet;
}
```

在序列化的代码中我们申请了一个数组的资源，然后在外面进行释放。

假如我在中间进行了什么操作，这个`delete[] packet`没有执行（比如提前`return`，或者代码的控制流没有到达这里），那么这个资源就会泄漏。

又或者我有很多个这种控制块，但是对其进行了很多不同的操作，比如一部分需要序列化，另一部分不需要，那么这个`delete[] packet`就会变得非常麻烦。

除此以外，假如后来你的协作者来接手这一段代码，他可能会忘记释放这个资源。

一个很简单的解决办法是这样：

```cpp
struct ControlBlock {
    uint32_t ipaddr;
    uint16_t port;
    uint32_t metric;
    
    void to_packet(char *packet) {
        memcpy(packet, &ipaddr, 4);
        memcpy(packet + 4, &port, 2);
        memcpy(packet + 6, &metric, 4);
    }
};

void func() {
    ControlBlock cb;
    cb.ipaddr = 0x7f000001;
    cb.port = 80;
    cb.metric = 100;
    
    char packet[12];
    cb.to_packet(packet);
    
    // do something with packet
}
```

这样，我们将这个资源的申请和释放拉到了同一个作用域下，对于一个新的维护者，不需要再关心这个函数内执行了什么，特别是有没有申请资源；而且在这个作用域结束的时候，这个资源就会被自动释放（栈区），避免了隐含的内存泄露风险。

### RAII的实现

RAII的实现就是将资源的申请和释放封装到一个类中，通过构造函数申请资源，析构函数释放资源。

因为析构函数会被自动调用，我们可以利用这一点来做类似于栈区的资源管理。

比如这样一个文件管理的例子：

```cpp

#include <iostream>
#include <fstream>
#include <stdexcept>

class FileHandler {
private:
    std::fstream file;

public:
    FileHandler(const std::string& filename, std::ios_base::openmode mode)
        : file(filename, mode)
    {
        if (!file.is_open()) {
            throw std::runtime_error("Failed to open file: " + filename);
        }
        std::cout << "File opened successfully.\n";
    }

    ~FileHandler() {
        if (file.is_open()) {
            file.close();
            std::cout << "File closed successfully.\n";
        }
    }

    void writeLine(const std::string& line) {
        file << line << std::endl;
    }

    std::string readLine() {
        std::string line;
        std::getline(file, line);
        return line;
    }

    // 禁止拷贝构造和赋值操作
    FileHandler(const FileHandler&) = delete;
    FileHandler& operator=(const FileHandler&) = delete;
};

int main() {
    try {
        FileHandler file("example.txt", std::ios::out);
        file.writeLine("Hello, RAII!");
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }
    // 文件会在这里自动关闭，即使发生异常

    return 0;
}
```

这个例子展示了RAII的核心概念：
- 在构造函数中，我们打开文件。如果打开失败，抛出异常，直接结束这段调用。
- 在析构函数中，我们确保文件被关闭。无论程序如何退出（正常退出或异常退出），析构函数都会被调用，从而保证资源被正确释放。
- 即使在文件操作过程中发生错误，RAII也能确保文件被正确关闭。
- 为了避免资源的意外共享或重复释放，我们禁用了拷贝构造函数和赋值操作符。

使用这个`FileHandler`类，我们不需要显式地调用`close()`函数。当`FileHandler`对象离开作用域时，析构函数会自动被调用，确保文件被关闭。这就是RAII的魅力所在：它将资源的生命周期与对象的生命周期绑定在一起，大大减少了资源泄漏的风险。

### 标准库

标准库中也有很多使用了RAII的类，比如智能指针、`std::lock_guard`等。

智能指针是一种RAII的实现，它会在对象离开作用域时自动释放内存。`std::unique_ptr`和`std::shared_ptr`是两种常用的智能指针，它们分别实现了独占所有权和共享所有权的内存管理。

例如：

```cpp
#include <memory>
#include <iostream>

void func() {
    std::unique_ptr<int> p(new int(42));
    std::cout << *p << std::endl;
} // p离开作用域时，内存会被自动释放

int main() {
    func();
    return 0;
}
```

`std::unique_ptr`更多地使用在被类或函数封装，需要强调**所有权**的场景下。

而`std::lock_guard`是一个用于管理互斥锁的RAII类，它会在构造函数中锁定互斥锁，在析构函数中释放互斥锁。这样可以确保在离开作用域时，互斥锁一定会被释放，避免了忘记释放锁的风险。

例如：

```cpp
#include <mutex>
#include <thread>
#include <iostream>

std::mutex mtx;

void func_with_lock_guard() {
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << "Hello, RAII!" << std::endl;
} // lock离开作用域时，互斥锁会被自动释放

void func_without_lock_guard() {
    mtx.lock();
    std::cout << "Hello, RAII!" << std::endl;
    // 如果在这里发生异常，互斥锁可能不会被释放
    mtx.unlock();
} 
```

前者使用了`std::lock_guard`，能自动释放，在规模更大的代码中也能方便地管理；后者手动管理互斥锁，有锁不被释放导致死锁的风险。