## CRTP

CRTP（Curiously Recurring Template Pattern）是一种模板元编程技服，它的核心思想是通过模板继承来实现多态。CRTP通过模板继承，将派生类作为基类的模板参数，从而实现多态。

### 运行时多态的问题

我们之前介绍的多态是运行时多态，它是通过虚函数和虚表实现的。

#### 虚函数和虚表

C++中的运行时多态主要通过虚函数（virtual function）来实现。当一个类声明了虚函数，编译器就会为这个类生成一个虚函数表（virtual function table）。这个机制允许在运行时根据对象的实际类型来调用正确的函数。

虚表本质上是一个函数指针数组，每个包含虚函数的类都有一个与之对应的虚表，虚表中的每个条目都指向一个虚函数的实现。

当一个类包含虚函数时，该类的每个对象都会包含一个指向虚表的指针(通常称为vptr)，这个vptr通常位于对象内存布局的开始位置。当调用一个虚函数时，程序首先通过对象的vptr找到对应的虚表；然后,根据函数在虚表中的索引，找到并调用正确的函数实现。

#### 问题

虚表和动态绑定机制会带来一定得性能开销：每个包含虚函数的对象都需要额外的内存来存储vptr；需要额外的间接寻址，而且会**破坏访存的局部性**。

过度使用虚函数还可能增加代码的复杂性，尤其是在大型项目中，虚函数的调用关系可能会变得非常复杂，导致代码难以维护。而且还很容易引入一些隐藏错误，比如某些资源没有被正确释放。

### 静态多态

在人类思维中，我们经常倾向于通过继承和类似性来理解和分类事物。CRTP以一种类似的方式工作，通过继承自己（在子类中使用父类模板），它在技术上实现了一种“自我认知”的模式。

比如：

```cpp

#include <iostream>

// Base class template using CRTP
template <typename Derived>
class Counter {
public:
    void increment() {
        count++;
        static_cast<Derived*>(this)->onCountChanged();
    }

    void decrement() {
        count--;
        static_cast<Derived*>(this)->onCountChanged();
    }

    int getCount() const {
        return count;
    }
};

// Derived class for incrementing counter
class IncrementCounter : public Counter<IncrementCounter> {
private:
    int count = 0;

public:
    void onCountChanged() {
        std::cout << "Increment counter changed. New value: " << getCount() << std::endl;
    }
};

// Derived class for decrementing counter
class DecrementCounter : public Counter<DecrementCounter> {
private:
    int count = 0;

public:
    void onCountChanged() {
        std::cout << "Decrement counter changed. New value: " << getCount() << std::endl;
    }
};

int main() {
    IncrementCounter inc;
    DecrementCounter dec;

    inc.increment(); // Output: Increment counter changed. New value: 1
    inc.increment(); // Output: Increment counter changed. New value: 2

    dec.decrement(); // Output: Decrement counter changed. New value: -1
    dec.decrement(); // Output: Decrement counter changed. New value: -2

    return 0;
}
```

在编译时知道派生类的具体类型，从而可以调用派生类的方法。在CRTP中，基类模板的方法可以调用派生类的方法，从而实现静态多态。

CRTP还可以提高代码的可重用性，例如多态拷贝构造：

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual Shape* clone() const = 0;
};

template <typename Derived>
class ShapeCRTP : public Shape {
public:
    Shape* clone() const override {
        return new Derived(static_cast<const Derived&>(*this));
    }
};

class Circle : public ShapeCRTP<Circle> {
public:
    Circle(const Circle& other) : radius(other.radius) {}
    Circle(double r) : radius(r) {}
private:
    double radius;
};

class Rectangle : public ShapeCRTP<Rectangle> {
public:
    Rectangle(const Rectangle& other) : width(other.width), height(other.height) {}
    Rectangle(double w, double h) : width(w), height(h) {}
private:
    double width, height;
};

int main() {
    Circle c1(1.0);
    Circle c2 = *c1.clone();

    Rectangle r1(2.0, 3.0);
    Rectangle r2 = *r1.clone();

    return 0;
}
```

传统的实现方式是，基类有个虚拟clone函数，每个继承类实现自己的`clone`函数功能。依靠CRTP技术，只定义一个就够了，减少了冗余代码，提高了代码的复用性。

CRTP还有很多的应用和操作，比如接下来同学们在课程中要使用的`Eigen`库就使用了大量的CRTP模式。在标准库中，`std::enable_shared_from_this`也是一个CRTP模式的例子。