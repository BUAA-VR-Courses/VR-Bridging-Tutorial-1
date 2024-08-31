## 面向对象

与C语言不同，C++有**类**和**面向对象**的概念。

### 类

在C++中，引入了`class`关键字，用于定义一个类。

```cpp
class typename {
// 或 struct typename {
public:
    // 公共变量
    // 公共函数

private:
    // 私有变量
    // 私有函数
};  
```

通过上面的形式可以定义一个类，注意结尾的分号，这与C语言的`struct`相同。

在类中除了可以定义变量外，还可以定义属于这个类的函数，这些函数在使用自己的变量时，不需要传入参数，因为它们属于这个类。

```cpp
class A {
    int attr;
public:
    void getAttr() {
        return attr;
    }
};
```

你应该已经注意到了`public`和`private`，这是访问权限的控制，`public`表示公共的，可以在类外访问，`private`表示私有的，只能在类内访问。

```cpp
class A {
public:
    int attr1;
private:
    int attr2;
public:
    void foo1() {
        std::cout << attr1 << std::endl;  
    }
private:
    void foo2() {
        std::cout << attr2 << std::endl;  
    }
};

int main() {
    A a;
    a.attr1 = 1;
    a.attr2 = 2;  // 错误
    a.foo1();
    a.foo2();   // 错误
}
```

还有一种情况，就是`protected`，它的访问权限介于`public`和`private`之间，`protected`成员可以被派生类访问，但不能被外部访问。

> C++的`class`和`struct`本质上是同一种东西，只是访问权限不同，`class`默认是`private`，而`struct`默认是`public`。因此你在C++中也可以使用和C语言一样的`struct`。

### 对象

我们学过的C语言是面向过程编程，对于一个变量（整形、字符串或是结构体），它的函数和变量都是分开的，其本质是上帝视角的我们用这个函数使用这个变量。

面向对象是符合人的思维的一种编程思想，我们称一个类为一个对象。比如人，它是一个对象，它有年龄，名字等“属性”，也有吃饭等“方法”（即属于对象的函数）。

```cpp
class People {
 	size_t age;
    std::string name;

public:
    void eat() {
        // ...
    }
};
```

这就是面向对象的思维方式。

### 继承

大多数情况下，对象有着**A is B**的继承关系。

比如`Chinese is People`，对象间有继承关系。比如Chinese可以继承People的年龄性别等属性，也有比如说省份等自己的属性；继承吃饭的方法，也可以有使用筷子等自己的方法。

```cpp
class Chinese : public People {
    std::string province;  
    // 其他属性直接继承了People类

public:
    void useChopsticks() {
        // ...
    }
};
```

在这个例子中，`Chinese`类称为**派生类**，`People`类称为**基类**。也可以叫子类和父类。

继承是一种抽象真实世界的思维方式，再举一个例子，比如我现在需要实现一个线性代数库，我可以这么组织：

1. 首先定义一个Base类，他规定了线性代数需要的一些基本函数，比如加减乘除等。
2. 然后定义一个Matrix类，它继承了Base类，实现了矩阵的加减乘除。
3. 然后可以定义一个Vector类，它继承了Base类，实现了向量的加减乘除。
4. 对Matrix，我可以进一步定义一个SparseMatrix类，它继承了Matrix类，实现了稀疏矩阵。

现实世界中比如林奈的分类法，也是一种继承思想的体现。

> 在面向对象的定义中，**封装、继承和多态**是面向对象的三大特性。但实际上继承不一定是必须的，比如Rust就没有传统意义上的继承，它的面向对象通过对功能的组合来实现。

### 多态

多态是面向对象编程的核心概念之一，它允许我们使用统一的接口来处理不同类型的对象。

在C++中，多态主要通过**虚函数**和**虚表**来实现。

#### 虚函数和动态绑定

在C++中，使用`virtual`关键字声明的函数称为虚函数。虚函数是实现多态的关键。当我们通过基类指针或引用调用虚函数时，C++会在运行时决定调用哪个版本的函数，这个过程称为**动态绑定**。

```cpp
#include <iostream>

class Animal {
public:
    virtual void makeSound() {
        std::cout << "The animal makes a sound" << std::endl;
    }
};

class Dog : public Animal {
public:
    void makeSound() override {
        std::cout << "The dog barks: Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() override {
        std::cout << "The cat meows: Meow!" << std::endl;
    }
};

int main() {
    Animal* animal1 = new Dog();
    Animal* animal2 = new Cat();

    animal1->makeSound();  // 输出: The dog barks: Woof!
    animal2->makeSound();  // 输出: The cat meows: Meow!

    delete animal1;
    delete animal2;

    return 0;
}
```

在这个例子中，我们通过基类`Animal`的指针调用`makeSound()`函数。由于`makeSound()`是虚函数，C++会根据指针实际指向的对象类型来调用相应的函数。

#### 纯虚函数和抽象类

纯虚函数是一种特殊的虚函数，它在基类中没有实现，而是要求派生类必须提供实现。包含纯虚函数的类称为抽象类，不能被实例化。

```cpp
class Shape {
public:
    virtual double area() = 0;  // 纯虚函数
};

class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() override {
        return 3.14159 * radius * radius;
    }
};

class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double area() override {
        return width * height;
    }
};

int main() {
    Shape* s = new Shape();  // 错误: 无法实例化抽象类

    Shape* c = new Circle(1.0); // 正确
    Shape* r = new Rectangle(2.0, 3.0); // 正确

    std::cout << c->area() << std::endl;  // 输出: 3.14159
    std::cout << r->area() << std::endl;  // 输出: 6.0
}
```

在这个例子中，`Shape`是一个抽象类，它定义了一个纯虚函数`area()`。`Circle`和`Rectangle`类继承自`Shape`并提供了各自的`area()`实现。

尽管`c`和`r`被向上转型为`Shape`指针，但由于多态动态绑定的特性，它们调用的是各自的`area()`函数。

#### 接口思维

接口是一种抽象的概念，它定义了对象的行为而不关心具体实现。在面向对象编程中，接口是一种非常重要的设计思想，其本质上是对功能的抽象。

虽然C++没有像Java那样的`interface`关键字，但我们可以使用纯虚函数来模拟接口。接口思维鼓励我们关注对象的行为而不是其具体实现，这有助于创建更加灵活和可扩展的代码。

```cpp
class Drawable {
public:
    virtual void draw() = 0;
};

class Movable {
public:
    virtual void move(int x, int y) = 0;
};

class Sprite : public Drawable, public Movable {
public:
    void draw() override {
        std::cout << "Drawing the sprite" << std::endl;
    }
    void move(int x, int y) override {
        std::cout << "Moving the sprite to (" << x << ", " << y << ")" << std::endl;
    }
};
```

在这个例子中，`Drawable`和`Movable`可以被视为接口。`Sprite`类实现了这两个接口，因此它必须提供`draw()`和`move()`方法的具体实现。

### 构造

C++的类有4中构造函数，分别用于实现对象的初始化、对象的销毁、对象的拷贝和对象的移动。

#### 构造函数 (Constructor)

构造函数是在创建对象时自动调用的特殊成员函数。它的主要任务是初始化对象的成员变量。

```cpp
#include <iostream>
#include <string>

class Person {
private:
    std::string name;
    int age;

public:
    // 默认构造函数
    Person() : name("Unknown"), age(0) {
        std::cout << "Default constructor called" << std::endl;
    }

    // 带参数的构造函数
    Person(const std::string& n, int a) : name(n), age(a) {
        std::cout << "Parameterized constructor called" << std::endl;
    }

    void display() const {
        std::cout << "Name: " << name << ", Age: " << age << std::endl;
    }
};

int main() {
    Person p1;                  // 调用默认构造函数
    Person p2("Alice", 30);     // 调用带参数的构造函数

    p1.display();
    p2.display();

    return 0;
}
```

#### 析构函数 (Destructor)

析构函数是在对象被销毁时自动调用的特殊成员函数。它的主要任务是释放对象可能占用的资源。

```cpp
#include <iostream>

class Resource {
private:
    int* data;

public:
    Resource() {
        data = new int[100];
        std::cout << "Resource acquired" << std::endl;
    }

    ~Resource() {
        delete[] data;
        std::cout << "Resource released" << std::endl;
    }
};

int main() {
    {
        Resource r;
        // r的作用域结束时,析构函数会被自动调用
    }
    std::cout << "End of main" << std::endl;
    return 0;
}
```

#### 拷贝构造函数 (Copy Constructor)

拷贝构造函数用于创建一个新对象作为已存在对象的副本。它在以下情况下被调用:
- 通过另一个对象初始化新对象
- 将对象作为参数按值传递
- 函数返回对象

```cpp
#include <iostream>
#include <cstring>

class String {
private:
    char* str;

public:
    // 构造函数
    String(const char* s = nullptr) {
        if (s) {
            str = new char[strlen(s) + 1];
            strcpy(str, s);
        } else {
            str = new char[1];
            str[0] = '\0';
        }
        std::cout << "Constructor called" << std::endl;
    }

    // 拷贝构造函数
    String(const String& other) {
        str = new char[strlen(other.str) + 1];
        strcpy(str, other.str);
        std::cout << "Copy constructor called" << std::endl;
    }

    // 析构函数
    ~String() {
        delete[] str;
        std::cout << "Destructor called" << std::endl;
    }

    void display() const {
        std::cout << str << std::endl;
    }
};

void printString(String s) {
    s.display();
}

int main() {
    String s1("Hello");
    String s2 = s1;  // 调用拷贝构造函数

    s1.display();
    s2.display();

    printString(s1);  // 调用拷贝构造函数(按值传递)

    return 0;
}
```

#### 移动构造函数 (Move Constructor)

移动构造函数是C++11引入的新特性,用于优化对象的转移过程。它允许将资源从一个对象转移到另一个对象,而不是进行深拷贝。


```cpp
#include <iostream>
#include <vector>

class BigData {
private:
    std::vector<int> data;

public:
    // 构造函数
    BigData(size_t size) : data(size, 0) {
        std::cout << "Constructor called" << std::endl;
    }

    // 拷贝构造函数
    BigData(const BigData& other) : data(other.data) {
        std::cout << "Copy constructor called" << std::endl;
    }

    // 移动构造函数
    BigData(BigData&& other) noexcept : data(std::move(other.data)) {
        std::cout << "Move constructor called" << std::endl;
    }

    size_t size() const {
        return data.size();
    }
};

BigData createBigData() {
    return BigData(1000000);  // 返回临时对象
}

int main() {
    BigData d1 = createBigData();  // 可能触发移动构造
    std::cout << "d1 size: " << d1.size() << std::endl;

    BigData d2 = d1;  // 触发拷贝构造
    std::cout << "d2 size: " << d2.size() << std::endl;

    BigData d3 = std::move(d1);  // 触发移动构造
    std::cout << "d3 size: " << d3.size() << std::endl;
    std::cout << "d1 size after move: " << d1.size() << std::endl;

    return 0;
}
```