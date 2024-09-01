## 运算符重载

在C++中，运算符指的是用于操作数据的符号，例如`+`、`*`、`+=`、`->`、`[]`等。

运算符本质上是一种函数，例如`a + b`实际上是`a.Add(b)`的简写（或者严格上说是`a.operator+(b)`）。

在上一节中我们介绍了类和面向对象的概念，对于`int`、`double`等内置类型，我们可以直接使用运算符进行操作，但是对于自定义类型，我们需要自己定义运算符的行为。

> 不是所有语言都提供运算符重载的功能，比如Java。

### 运算符重载原因

我们通过一个例子来感受运算符重载的重要性。

我们现在实现一个高精度整数类`BigInt`，它可以处理任意长度的整数。

```cpp
class BigInt {
private:
    std::vector<int> digits;

public:
    BigInt(std::string str) {
        for (char c : str) {
            digits.push_back(c - '0');
        }
    }

    BigInt(std::vector<int> digits) : digits(digits) {}
};
```

假如我们现在要实现加法和乘法的功能，我们可以这么写：

```cpp
class BigInt {
// ...
public:
    BigInt Add(BigInt& other) {
        std::vector<int> result;
        int carry = 0;
        int i = digits.size() - 1, j = other.digits.size() - 1;
        while (i >= 0 || j >= 0 || carry) {
            int sum = carry;
            if (i >= 0) sum += digits[i--];
            if (j >= 0) sum += other.digits[j--];
            result.push_back(sum % 10);
            carry = sum / 10;
        }
        std::reverse(result.begin(), result.end());
        return BigInt(result);
    }

    BigInt Multiply(BigInt& other) {
        BigInt result("0");
        for (int i = 0; i < other.digits.size(); i++) {
            BigInt temp = *this;
            for (int j = 0; j < other.digits[i]; j++) {
                result = result.Add(temp);
            }
            temp.digits.insert(temp.digits.begin(), 0);
        }
        return result;
    }
};
```

那么，我在使用这样一个式子`(12345 * 54321 + 67890) * 98765`时，就需要这么写：

```cpp
int main() {
    BigInt a("12345");
    BigInt b("54321");
    BigInt c("67890");
    BigInt d("98765");

    BigInt result1 = ((a.Multiply(b)).Add(c)).Multiply(d);
    return 0;
}
```

看上去非常难受，而假如我们进行运算符重载，我们可以这么写：

```cpp
class BigInt {
// ...
public:
    BigInt operator+(BigInt& other) {
        return Add(other);
    }

    BigInt operator*(BigInt& other) {
        return Multiply(other);
    }
};

int main() {
    BigInt a("12345");
    BigInt b("54321");
    BigInt c("67890");
    BigInt d("98765");

    BigInt result1 = ((a * b) + c) * d;
    return 0;
}
```

这看上去就清爽多了。

> 引用某位同学的观点，代码本质上是给人看的，而相比于函数调用的英文字符串，人对于符号的观测更加直观。

### 运算符重载规则

C++中的运算符重载允许我们为自定义类型定义运算符的行为。有两种主要的方式来实现运算符重载：成员函数方式和友元函数方式。

#### 成员函数方式

成员函数方式将运算符重载函数定义为类的成员函数。

- 左操作数必须是该类的对象
- 具有隐含的 `this` 指针,可以直接访问类的私有成员
- 运算符函数的参数比实际操作数少一个(左操作数是对象本身)

例如：

```cpp
class Complex {
public:
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
private:
    double real, imag;
};
```

#### 友元函数方式

友元函数方式将运算符重载函数定义为类的友元函数。

- 友元不是类的成员，但可以访问类的私有成员
- 所有操作数都作为参数传递
- 可以在左操作数不是该类对象的情况下使用

```cpp
class Complex {
    friend Complex operator+(const Complex& lhs, const Complex& rhs);
private:
    double real, imag;
};

Complex operator+(const Complex& lhs, const Complex& rhs) {
    return Complex(lhs.real + rhs.real, lhs.imag + rhs.imag);
}
```

#### 选择规则

一般来说，成员函数和友元函数有如下选择规则：

成员函数适合:
- 单目运算符(如++, --, *, &)
- 需要修改对象状态的运算符(如+=, -=, *=)
- 与类紧密相关的双目运算符(如[], ->)

友元函数适合:
- 交换律的双目运算符(如+, -, *, /)
- 需要进行类型转换的运算符
- 输入输出运算符(<<, >>)

#### 是否可重载

下面是可重载的运算符列表：

| 运算符         | 描述                                                        |
| -------------- | ------------------------------------------------------------ |
| 双目算术运算符 | + (加)，-(减)，*(乘)，/(除)，% (取模)                        |
| 关系运算符     | ==(等于)，!= (不等于)，< (小于)，> (大于)，<=(小于等于)，>=(大于等于) |
| 逻辑运算符     | &#124;&#124;(逻辑或)，&&(逻辑与)，!(逻辑非)                          |
| 单目运算符     | + (正)，-(负)，*(指针)，&(取地址)                            |
| 自增自减运算符 | ++(自增)，--(自减)                                           |
| 位运算符       | &#124; (按位或)，& (按位与)，~(按位取反)，^(按位异或),，<< (左移)，>>(右移) |
| 赋值运算符     | =, +=, -=, *=, /= , % = , &=, &#124;=, ^=, <<=, >>=              |
| 空间申请与释放 | new, delete, new[ ] , delete[]                               |
| 其他运算符     | **()**(函数调用)，**->**(成员访问)，**,**(逗号)，**[]**(下标) |

下面是不可重载的运算符列表：

- **.**：成员访问运算符
- **.\***, **->\***：成员指针访问运算符
- **::**：域运算符
- **sizeof**：长度运算符
- **?:**：条件运算符
- **#**： 预处理符
