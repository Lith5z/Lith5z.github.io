---
title: "语法 从C到C++"
date: 2026-04-14
tags: [编程]
---

# 从C到C++ 语法整理

## 说在前面

学再多语法也**不如自己动手写**，千万**不要只听课不写当耐听王**！
cpp的语法很复杂，不自己写一下几天后就会全部忘掉，这里推荐一个简单cpp语法练习网站：
[HackerRank-cpp practice](https://www.hackerrank.com/domains/cpp)
尤其练习Classes和Inheritance板块，可以很好的练习简单的OOP语法

如果你学习cpp是为了打算法比赛，那么直接当C with STL就行；
如果你要学习OOP，真的去做一个项目才是最关键、最能锻炼能力的

课程来自Cherno-C++
别人的[笔记](https://chocomintopia.github.io/)
49-51 跳过库管理
62-63 跳过线程、计时
72 74 76-84 跳

## 一些简单的差别
### include
cpp兼容c的标准库头文件
比如c的`#include<string.h>`就有一个cpp版本是`#include<cstring>`
两者都可以使用，只不过前者给函数和宏放入全局命名空间，后者放入std空间，推荐后者

### using
有两个主要用途，引入命名空间、类型别名（类似typedef）

**命名空间 namespace**
一种封装机制，用来避免命名冲突，不同的人或库可能定义相同名字的函数、类
```cpp
namespace MyCompany {
    class String { ... };
}

namespace OtherCompany {
    class String { ... };
}

MyCompany::String s1;
OtherCompany::String s2; //语法和使用类中的静态方法/嵌套的类是一样的，都是作用域的体现
```

如果写`using namespace MyCompany`，这个语句作用域中的所有String等定义的东西都会默认是MyCompany下定义的
不推荐到处写`using namespace std`，要尽量缩减这样写的作用域，千万不要在头文件里这样写（会污染所有带了这个头文件的代码）

`using apple::print;`
是针对性的，把 apple 命名空间里所有的print引入当前作用域，就在当前作用域中可以直接使用print而无需指明`apple::`


**类型别名**
```cpp
// 简化复杂的模板类型
using StringList = std::vector<std::string>;
using Callback = std::function<void(int, const std::string&)>;
```

### 类型转换 cast
cpp里有四个类型转换：static_cast reinterpret_cast dynamic_cast const_cast

```cpp
double d = 3.14;
int i = static_cast<int>(d); // static_cast在编译时转换，用于基本类型转换
Derived* derived = new Derived();
Base* base = static_cast<Base*>(derived); // 以及上行转换（子类变基类）

class Entity {
};
class Player : public Entity {
};
class Enemy : public Entity {
};
Entity *e = new Player;
Player* p = dynamic_cast<Player*>(e); // dynamic_cast用于下行转换，需要虚函数表
Entity *ee = new Enemy;
Player *p2 = dynamic_cast<Player*>(ee); // 这里指针转换失败 p2 = nullptr
// 原理是利用RTTI进行类型验证

const int* const_ptr = &a;
int* non_const_ptr = const_cast<int*>(const_ptr); // 移除常量指针的const属性，不能对常量这样用，UB

char* char_ptr = reinterpret_cast<char*>(&num); // 直接在二进制层面重新解释数据
```

可以代替c的强制类型转换

## 现代c++的新特性
如果没有特别说明，则是c++11

### auto 自动类型推定
编译器会根据初始化时的值来自动推断变量类型，比如`auto i = 1`自动推断为int，但是必须要做初始化
这是编译期推导，无运行时开销，属于典型的语法糖

基于范围的for循环
```cpp
for (auto i : arr) {
    cout << i << " ";   //注意不是a[i]，这里的i不是索引，是遍历arr的一个变量
}
for (auto &i : arr) {
    i++;
}
```

常用的使用情况就是迭代器/for遍历，或者模板这种类型名非常长的情况下
对于一般的int这种基本类型，没必要用auto

### 通用的`{}`列表初始化
暂无

### lambda表达式
只需要使用一次的函数，可以在使用处定义，除了传入参数，还能捕获外部变量
`auto lambda = []() { std::cout << "Hello"; };`
常用在sort、find_if里

[]为捕获类型，决定对外部变量要如何访问
如果留空，就是不访问外部变量
```cpp
class MyClass {
    int member = 100;
public:
    void test() {
        int local = 10;
        static int staticVar = 20;
        
        // 值捕获：local 被复制
        auto lambda1 = [local]() { 
            std::cout << local;  // 可以，local 是复制的副本
            // local = 20;       // ❌ 错误，值捕获默认是 const
        };
        
        // 引用捕获：可以修改原变量
        auto lambda2 = [&local]() { 
            local = 20;  // ✅ 可以，修改的是原变量
        };
        
        // 值捕获但允许修改（需要 mutable）
        auto lambda3 = [local]() mutable { 
            local = 20;  // ✅ 可以，但只修改副本
        };
        
        // 捕获 this：可以访问成员变量
        auto lambda4 = [this]() { 
            std::cout << member;  // 可以访问成员
        };
        
        // 静态变量不需要捕获，直接可用
        auto lambda5 = []() { 
            std::cout << staticVar;  // ✅ 静态变量可以直接访问
        };
    }
};
```
除此之外，[=]就是对所有的外部变量都值传递，[&]都为引用传递

()内的是参数
{}内的是函数体，和一般函数一样

返回类型会由编译器自行决定，如果要指定类型，写
```cpp
auto lambda = []() -> double { 
    if (condition) return 42;   // 转换为 double
    return 3.14; 
};
```

每个 lambda 都有一个唯一的、不可命名的类型
如果要把lambda表达式传给其他函数，那个函数的参数有两种写法
1. 函数指针 只有不捕获变量的才行
比如`[](int &a){return a + 1;}` 类型可以被隐式转换为`int(*func)(int &a)`
2. function
引入`functional`头文件后，调用函数写`(const function<int(int&)> &func, ...)`，也会被转换
但是有虚函数等额外开销
3. 模板（最优）
`template <typename T>`之后，函数参数写`(T &&func, ...)`，完美转发，零开销

### 标准库智能指针
用于自动管理动态内存的**类**模板，它们确保在适当的时候自动删除所指向的对象，防止内存泄漏
需要`#include <memory>`

实际上，智能指针不是个指针，而是一个类，只不过能实现指针的功能，而且会自动根据作用域进行delete
也就是说，原本`entity *p`的p是个指针变量，如果打印出来，就是个地址；而`unique_ptr<entity> p`的p是个类的实例，不能直接打印
可以通过`.get()`得到真正的地址

推荐使用auto代替类型名
`auto p = make_unique<Distance>(10);`

**独占所有权的unique_ptr**
当超出作用域的时候，自动delete
性能与裸指针相当

创建方式：
```cpp
auto up = std::make_unique<int>(42);
auto arr = std::make_unique<int[]>(10);  // 数组形式
```

不能拷贝，只能移动，也就是说`unique_ptr<Entity> ptr3 = ptr2`是报错的
（防止两个指针指向一个内存，造成多次释放）

**共享所有权的shared_ptr**
引用计数就是目标对象被多少个指针指向，当个数为0，自动delete释放对象
额外维护引用计数的开销

创建方式：
`auto sp = std::make_shared<std::string>(10, 'a');`

```cpp
void func() {
    std::shared_ptr<Entity> sp1 = std::make_shared<Entity>();
    {
        std::shared_ptr<Entity> sp2 = sp1;  // 该Entity被sp1和sp2指向，引用计数变为2
        // 使用 sp2...
    }  // sp2 销毁，引用计数变为1
    
    // 继续使用 sp1...
}  // sp1 销毁，引用计数变为0，Entity对象被删除
```

可以使用`.unique()`查看这个指针是否独占对象，`.use_count()`返回引用计数

**weak_ptr**
复制shared_ptr时，用这个不会增加引用次数
解决循环引用：在父子关系或双向引用中，一方使用 weak_ptr

解引用weak_ptr之前，先用`if (auto p = wptr.lock())`检查wptr指向的对象是否还存活
如果没存活，if判断不成立；如果存活，p会赋值一个临时的shared_ptr

### 多返回值 tuple pair
在c++17之前，tuple是这样用的
```cpp
std::tuple<std::string, int> CreatePerson() {
    return {"Miku", 17};
}
// 返回姓名和年龄的tuple 用pair也行 但是tuple可以用更多参数

int main() {
    std::tuple<std::string, int> person = CreatePerson();
    // 可以直接用auto来取代std::tuple<std::string, int>

    std::string& name = std::get<0>(person);  // 过于magic
    int age = std::get<1>(person);
}
```

后来有了结构化绑定，可以这样写
```cpp
int main() {
    auto[name, age] = CreatePerson();
    std::cout << "Name: " << name << ", Age: " << age << std::endl;
}
```

### print和println
c++23 print头文件 需要额外的-lstdc++exp参数
安全又高效

```cpp
int x = 10, y = 20;
std::println("x: {}, y: {}", x, y); // 输出: x: 10, y: 20

vector<int> a = {1,2,3,4,5};
std::println("{}", a); // 支持ranges打印

std::println("{1} {0} {1}", "A", "B"); // 输出: B A B

std::println("[{:>10}]", 42);   // 右对齐，输出 [        42]
std::println("[{:<10}]", 42);   // 左对齐，输出 [42        ]
std::println("[{:^10}]", 42);   // 居中对齐，输出 [    42    ]
std::println("[{:*<10}]", 42);  // 用*填充，输出 [42********]

double pi = 3.1415926;
std::println("π = {:.3f}", pi); // 保留3位小数，输出 π = 3.142

int num = 255;
std::println("dec: {}, hex: {:x}, oct: {:o}, bin: {:b}", num, num, num, num);
// 输出: dec: 255, hex: ff, oct: 377, bin: 11111111

// 也可以写到流里
```

## 指针和引用
### 内存分配 new/delete
`int *a = new int(6)`，括号内留空则是初始化为0，或者用c++11起通用的`{}`初始化
如果`int *a = new int`，则不初始化，垃圾值

对于类 `Entity* e = new Entity()` 推荐这样写，除了调用类中的构造函数外，对于**int等类型也会初始化为0**

`int* nums = new int[5]{1, 2, 3, 4, 5}` 对于数组，不用自己写乘法计算大小
`delete a`以及对应数组的`delete[] arr;`

和malloc不同的是，如果分配失败，不用手动`if (p == NULL)`检查，会自动抛出异常
以及new/delete会自动调用构造函数

更推荐使用智能指针

### 引用
简单给函数传递要修改的变量的时候，c语言需要用指针
```cpp
void swap(int *a, int *b) {
    int t = *a;
    *a = *b;
    *b = t;
}
```
对于这种情况，c++提供了更好的方法，引用，也就是变量的别名
```cpp
void swap(int &a, int &b) {    //传入实参的时候直接写变量名swap(num1,num2)，不用加其他东西
    int t = a;    //这里的a b并不会再从栈里申请空间，而只是作为实参的别名
    a = b;
    b = t;
}
```
对于各类函数，都**推荐传入const &**

引用在申明时就必须初始化，而且不能像指针一样更改引用的目标，比如
```cpp
int a = 5, b = 10;
int &ref = a;
ref = b;    //a = 10, b = 10
```

引用的底层实现就是个指针，只不过使用起来会自动解引用，就能像值一样操作。此外不能为空，不能重新绑定

## 操作符重载
操作符其实就是个自带函数名的函数，如同函数的重载，c++里可以给运算符定义自己需要的功能
下面以这个数组为例
```cpp
class MyArray {
private:
    int* data;
    size_t size;
}
```

### 一般的重载
有两种方式，作为类的成员函数重载，或者写成全局函数，再作为友元声明
区别是前者传入this指针，少一个参数，一般选择这种写法

```cpp
    // 加法运算符重载 +
    // 返回类型是类的类型
    MyArray operator+(const MyArray& other) const {
        MyArray result(size + other.size);
        std::copy(data, data + size, result.data);
        std::copy(other.data, other.data + other.size, result.data + size);
        return result;
    }
    // 加减乘除一般选择全局函数的方式重载，因为可能需要自动类型转换
    // 例如定义一个float类，写Myfloat a = 3.1 + b;就可以自动把3.1转换成自定义的类
```

### 比较运算符
```cpp
class MyString {
    char* _data;
public:
    bool operator==(const MyString& other) const {
        return strcmp(_data, other._data) == 0; // 等于0代表相等
    }
    bool operator!=(const MyString& other) const {
        return !(*this == other); // 调用 == 取反
    }
};
```

### 下标运算符
```cpp
    // 下标运算符重载 []
    // 提供两个版本，非const版本返回引用
    int& operator[](size_t index) {
        if (index >= size) {
            throw std::out_of_range("Index out of range");
        }
        return data[index];
    }
    // const版本返回常量引用，标记为const方法
    const int& operator[](size_t index) const {
        if (index >= size) {
            throw std::out_of_range("Index out of range");
        }
        return data[index];
    }
```

### 自增运算符
```cpp
    // 前置递增运算符 ++
    // 返回修改后的对象引用
    MyArray& operator++() {
        for (size_t i = 0; i < size; ++i) {
            ++data[i];
        }
        return *this;
    }
    
    // 后置递增运算符 ++(int)
    // 返回新建的旧值
    MyArray operator++(int) {
        MyArray temp(*this);  // 保存当前状态，之后返回
        ++(*this);           // 调用前置递增，修改这个原本的状态，而非返回的实例
        return temp;
    }
```

这种区分是自增运算符的使用习惯决定的，对于cpp的非基本类型，使用的惯例就是，前置返回引用，后置返回值（所以前置更高效，优先使用）

### 解引用和箭头操作符（智能指针）
下面以智能指针为例：
```cpp
#include <iostream>

class Data {
private:
    int value;
public:
    Data(int v = 0) : value(v) {}
    int get() const { return value; }
    void set(int v) { value = v; }

    static void* operator new(size_t size) {
        std::cout << "[Data] Allocated";
        void *p = malloc(size);
        return p;
    }
    static void operator delete(void* p) {
        std::cout << "[Data] Deleted";
        free(p);
    }

    /*
    //但是实际上，用new/delete更符合cpp规范
    static void* operator new(size_t size) {
        std::cout << "[Data] Allocated";
        return ::operator new(size);  // 使用全局 operator new
    }
    static void operator delete(void* p) {
        std::cout << "[Data] Deleted";
        ::operator delete(p);  // 使用全局 operator delete
    }
    */
};

class SharedPtr {
private:
    Data* ptr;
    int* refCount;
public:
    SharedPtr(Data* p = nullptr) : ptr(p), refCount(new int(1)) {}
    
    // 拷贝构造
    SharedPtr(const SharedPtr& other) : ptr(other.ptr), refCount(other.refCount) {
        (*refCount)++;
        // 多了一个指针指向对象
    }
    
    // 赋值操作符
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != &other) {  // 首先赋值操作符要防止自赋值
            
            // 释放旧资源，减少旧资源的引用计数
            (*refCount)--;
            // 检查减少后
            if (*refCount == 0) {
                delete ptr;
                delete refCount;
            }
            
            // 指向新资源
            ptr = other.ptr;
            refCount = other.refCount;
            (*refCount)++;
        }
        return *this;
    }
    
    ~SharedPtr() {
        // 析构后少了一个指针
        (*refCount)--;
        if (*refCount == 0) {
            delete ptr;
            delete refCount;
        }
    }

    Data* operator->() { return ptr; }
    Data& operator*() { return *ptr; }
};

class UniquePtr {
private:
    Data* ptr;
public:
    UniquePtr(Data* p = nullptr) : ptr(p) {}

    // 独占，禁止拷贝构造和赋值
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;
    ~UniquePtr() { delete ptr; }

    Data* operator->() { return ptr; }
    Data& operator*() { return *ptr; }
};
```

对于`shared_ptr`，在任何减少引用计数的地方，**都必须检查引用计数是否归零**

### new/delete的重载
```cpp
class MyString {
private:
    char *data;
    int size;
public:
    MyString(const char *str = "") {
        size = strlen(str);
        data = new char[size+1];
        strcpy(data,str);
    }
    ~MyString() {
        delete[] data;
    }

    static void* operator new(size_t size) {
        void *p = malloc(size);
        return p;
    }
    static void operator delete(void *p) {
        free(p);
    }
};
```

实际上就是包装了一次`free/malloc`，但是可以实现更多功能（创建了几个对象，实现自定义的内存分配防止碎片）

会自动进行对构造函数和析构函数的调用，`MyString* str = new MyString("Hello");`
这个语句，先调用`new`分配空间，然后把参数`"Hello"`传入构造函数；`delete`反过来

也可以重载带有其他参数的版本，但是第一个参数和一般的版本一样，而且`new/delete`要配对

### 赋值运算符
返回的类型都是类的引用，为了实现连续赋值

拷贝赋值，传入的参数是`&`，实际上是左值引用，引用的是有地址的对象
```cpp
    MyArray& operator=(const MyArray& other) {
        if (this != &other) {  // 比较指针，防止自赋值
            delete[] data;      // 释放原有资源
            
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;  // 返回引用，支持链式赋值
    }
```

c++11之后，引入了移动语义，新增了移动赋值
传入的参数是右值引用，可以引用临时变量
```cpp
    MyArray& operator=(MyArray&& other) noexcept {
        if (this != &other) {  // 防止自移动（虽然很少见）
            // 1. 释放当前对象持有的资源
            delete[] data;
            
            // 2. "偷取"other的资源
            data = other.data;
            size = other.size;
            
            // 3. 将other置于安全状态（可以被安全析构）
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }
```
从这个例子来看，移动赋值很像一次浅拷贝，只不过多了置空的步骤
因为传入的参数是右值引用，而可能拷贝的对象是左值，所以用`std::move`进行类型转换
```cpp
    MyArray a1 = {1,2,3}, a2 = {1,2,3,4,5};
    a1 = std::move(a2);
```
`std::move`本身不会修改数据，只是一个转换，具体的操作要在移动赋值操作符重载里实现


### 流运算符的重载
`<<` `>>`

两种方法：
1. 在类中声明，类外定义，使用友元访问数据
```cpp
class Point {
private:
    int x, y;
public:
    Point(int x = 0, int y = 0) : x(x), y(y) {}

    //参数必须写Box &b，友元不是类的方法，不会传入this指针
    //友元，使得<<能以b.的形式访问
    friend std::ostream& operator<<(std::ostream& os, const Point& p) {
        //os << 而不是cout 为了支持多种流
        os << "(" << p.x << ", " << p.y << ")";    //注意这里不应该输出换行，这个由调用者自己写std::endl
        return os;
    }

    //关于类的参数别写const，要修改对象
    friend std::istream& operator>>(std::istream& is, Point& p) {
        is >> p.x >> p.y;
        return is;
    }
};
```
推荐这种做法

2. 类外声明，相当于一个全局函数
```cpp
//类内什么都不用写
ostream& operator<<(ostream &os, const Point& p)  {
    os << p.getX() << " " << p.getY();
    //用getter接口函数
    return os;
}
```

### 类型转换操作符的重载
```cpp
class Temperature {
private:
    double celsius;
    
public:
    Temperature(double c) : celsius(c) {}
    
    // 转换为double类型（返回摄氏度值）
    operator double() const {
        return celsius;
    }
    
    // 转换为int类型（取整后的温度）
    operator int() const {
        return static_cast<int>(celsius);
    }
};

int main() {
    Temperature temp(36.8);
    
    // 隐式转换：operator double()
    double c = temp;           // 等价于 temp.operator double()
    cout << c << "°C" << endl;  // 输出：36.8°C
    
    // 隐式转换：operator int()
    int t = temp;
    cout << t << endl;          // 输出：36

    return 0;
}
```

## 模板
```cpp
template<typename T> //typename也可以用class，等价
void Print(T value) {
    std::cout << value << std::endl;
}
Print(5);
Print("Hello"); //编译器会隐式推断类型
Print<int>(5); //也可以显示给出 STL就是这样实现的
```
因为cout支持多个类型，这样就可以直接调用了

```cpp
#include <iostream>
#include <cstring>

// 类模板
template<typename T, int Capacity>
class Buffer {
private:
    T data[Capacity];
    int size;

public:
    Buffer() : size(0) {}

    void push(const T& value) {
        if (size < Capacity) {
            data[size++] = value;
        }
    }

    T& operator[](int index) {
        return data[index];
    }
    const T& operator[](int index) const {
        return data[index];
    }
};

// 模板类的成员函数可以在类外定义
template<typename T, int Capacity>
void Buffer<T, Capacity>::clear() {
    size = 0;
}
```
还可以有非类型的其他模板参数

模板函数若没调用是不会编译的，对于不同的参数组合，会根据模板编译成不同的代码

## 类 面向对象编程
### 例子
```cpp
using namespace std;

class SensorBuffer {
protected:
    double* data;
    size_t size;
    bool has_released;

public:
    SensorBuffer(size_t n = 0) {
        has_released = false;
        size = n;
        data = new double[n];
        for (int i = 0; i < n; i++) {
            data[i] = 0.0;
        }
        cout << "[SensorBuffer] Constructed, size=" << n << endl;
    }
    SensorBuffer(size_t n, double* _data) {
        has_released = false;
        size = n;
        data = new double[n];
        for (int i = 0; i < n; i++) {
            data[i] = _data[i];
        }
        cout << "[SensorBuffer] Constructed, size=" << n << endl;
    }

    SensorBuffer(const SensorBuffer& other) {
        has_released = false;
        size = other.size;
        data = new double[size];
        for (int i = 0; i < size; i++) {
            data[i] = other.data[i];
        }
        cout << "[SensorBuffer] Copied (deep copy)" << endl;
    }

    virtual ~SensorBuffer() {
        if (has_released == true) {
            //已经析构过了
            cout << "[SensorBuffer] Already released" << endl;
            return;
        }
        delete[] data;
        data = nullptr;
        size = 0;
        cout << "[SensorBuffer] Data released" << endl;
        has_released = true;
    }

    double get(int id) const {
        return data[id];
    }
    void set(int id, double value) {
        data[id] = value;
    }
    virtual double average() const {
        double sum = 0.0;
        for (int i = 0; i < size; i++) {
            sum += data[i];
        }
        return sum / size;
    }
    virtual void printInfo() const {
        cout << "[SensorBuffer] Info: size=" << size << endl;
    }
};

class TemperatureBuffer : public SensorBuffer {
private:
    string unit;

public:
    TemperatureBuffer(size_t n, const string& _unit = "C") : SensorBuffer(n), unit(_unit) {
        has_released = false;
        cout << "[TemperatureBuffer] Constructed, unit=" << _unit << endl;
    }
    TemperatureBuffer(size_t n, double* init_data, const string& _unit = "C") : SensorBuffer(n,init_data), unit(_unit) {
        has_released = false;
        cout << "[TemperatureBuffer] Constructed, unit=" << _unit << endl;
    }

    TemperatureBuffer(const TemperatureBuffer& other) : SensorBuffer(other) {
        has_released = false;
        unit = other.unit;
        cout << "[TemperatureBuffer] Copied from another TemperatureBuffer" << endl;
    }

    virtual ~TemperatureBuffer() {
        cout << "[TemperatureBuffer] Destructor called" << endl;
    }
    void printInfo() const override {
        cout << "[TemperatureBuffer] Info: unit=" << unit <<  ", size=" << size << endl;
    }
};

int main() {
    double test[5] = {1.1, 2.2, 3.3, 4.4, 5.5};
    TemperatureBuffer t1(5, test, "C");
    TemperatureBuffer t2 = t1;
    // 输出信息和平均值
    t1.printInfo();
    t2.printInfo();
    std::cout << std::fixed << std::setprecision(2);
    std::cout << "Average of t1 = " << t1.average() << std::endl;
    std::cout << "Average of t2 = " << t2.average() << std::endl;
    t2.~TemperatureBuffer();
    return 0;
}
```


### 类
类是一个包含了数据（成员变量）和函数（方法）的封装，类类型的变量称为对象，新的对象变量称为实例
区分：局部变量、成员变量、全局变量

一般而言，成员变量用 `m_` 前缀命名，区分函数中出现的临时的局部变量（虽然常用于private的成员变量）
也常用给局部变量前面加下划线的区分方法

可见性
类中的数据默认是私有/private的，加上public才能被外界访问，否则只能被类中的方法访问

类方法
如果是非静态的，在类外需要通过`obj.func()`来调用，类内可以直接调用
这时候类中的方法会默认传入`this`指针，来自调用该方法的那个对象，允许这个方法访问类的数据而不需要传参
静态方法static，在类外必须写`className::func()`来调用，类内可以直接写
（非静态的方法如果也不需要依据实例的话，推荐按静态方法的方式处理，避免命名空间问题）

实例化
栈上的对象
`Entity a("name")` 调用带参数的构造函数 推荐
`Entity a` 调用默认构造函数
`Entity e2 = Entity("123")` 先显式构造一个临时对象，再进行拷贝构造 不推荐 虽然会做编译器优化去掉拷贝
`Distance e{5}` c++11起的统一初始化

堆上的对象
`Entity *a = new Entity()` 推荐 除了调用类中的构造函数外，对于**int等类型也会初始化为0**
`Entity *a = new Entity` 不推荐

### this 指针
在类的（非静态）方法中，编译器会自动传入一个参数 this 指针，指向调用该方法的实例（不存在调用静态方法的实例，所以静态方法不传入this指针）
类型是`className* const`，如果是const方法就是`const className* const`

有以下用途：
**区分局部变量和成员变量**
```cpp
class person {
    string name;
    person(string &name) {
        //name = name; 这样初始化是错误的，因为作用域的遮蔽效应，实际上在给形参重复赋值
        this->name = name;    //这样是对的，虽然更推荐初始化列表
    }
    person& func1() {
        name += "a";
        return *this;    //因为返回类型是引用，所以对this解引用
    }
}
```
**链式调用**
比如`person.func1().func2()`这样的操作，这里的func1()就返回了`person&`类型，可以继续调用下一个函数（如果写成void，效果是一样的，但是就不能链式调用了）

### 结构体
c++中的**结构体也可以包含函数**，所以struct和class只有可见性的区别，struct默认是公开的，而class是私有的
一般而言，struct用于处理简单的数据组合

此外，使用时可以省略struct，相当于c中使用了typedef
```cpp
struct student {
    int id;
    string name;
};
student data[50];
```

### 可见性
private 只有类的作用域中（就是{}之间的代码）可以访问，（也就是只能被该类自己的成员函数访问），类的实例和类外都不能访问，友元除外
protected 在基类和子类中可以访问，但是实例和类外不能
public 不限制访问

### 友元 friend
允许特定的**非成员函数**或**其他类**访问当前类的 `private` 和 `protected` 成员。
打破封装限制，用于紧密协作的类或运算符重载。

**友元函数**
定义在类外部的普通函数，但在类内部声明为 `friend`
```cpp
class Box {
private:
    int width;
public:
    Box(int w) : width(w) {}
    
    // 【声明】在类内声明友元函数
    friend void printWidth(Box box); 
};

// 【定义】在类外直接定义，无需 Box::
void printWidth(Box box) {
    cout << box.width << endl; // 可直接访问私有成员
}
```
这里相当于只是在类内做了一个身份的声明，允许printWidth访问Box类的数据，但是实际上这个函数就是一个普通的全局函数，不是类的方法
因此也不会传入this指针，因而参数要写`Box box`

**友元类**
将整个类 B 声明为类 A 的友元，则 B 的**所有**成员函数均可访问 A 的私有成员。
```cpp
class A {
private:
    int secret;
public:
    // 【声明】声明整个类 B 为友元
    friend class B; 
};

class B {
public:
    void access(A& a) {
        a.secret = 10; // B 的所有函数都可访问 A 的私有成员
    }
};
```

**友元成员函数**
仅将另一个类的**特定成员函数**声明为友元。

要让 A 类的成员函数成为 B 类的友元：
1. 先声明 B
2. 再定义 A
3. 再定义 B
4. 最后实现 A 的成员函数

```cpp
class Book; //声明Book类，不然下面print声明不知道什么是Book类

class Library { //完整声明Library类，友元要求
public:
    void print(Book &b); //here
};

class Book {
    static int BookCnt;
    char* name;
public:
    friend void Library::print(Book &b); //here
};

void Library::print(Book &b) { //最后实现友元函数，因为Book类要完整声明才能访问其中的数据
    cout << b.name << endl;
}
```

### 构造函数 析构函数
```cpp
class Entity {
public:
    float X, Y;
    
    void Print() {
        std::cout << X << ", " << Y << std::endl;
    }
};
```
这样的类创建的实例中x y是不会初始化的，我们需要写一个初始化函数，也就是构造函数
```cpp
//...
    Entity() {
        X = 0.0f;
        Y = 0.0f;
    }
```
构造函数`Entity()`在**创建实例**的时候自动运行

构造函数也可以带参数，相当于重载实现多个构造函数
```cpp
//...
    Entity(float _X, float _Y) {
        X = _X;
        Y = _Y;
    }
```

如果不写构造函数，编译器也会自动创建一个默认构造函数，但是什么都不会做
但是如果**已经写了一个构造函数**（无论有无参数），编译器认为你明确想要控制对象的构造方式，所以**不会再生成默认构造函数**

如果只希望别人用类中的静态的方法，阻止实例化
```cpp
class Log{
private:
    Log() = delete; // 构造函数被删除了
public:
    static void Write() {
    
    }
}
```
我只想让别人这样用我的Log类 Log::Write(); 不希望别人实例化

析构函数里的内容一般针对的是类在堆上分配的数据
在栈上的对象会在离开作用域的时候调用析构函数（main中的会在程序结束运行时），所以可以不手动析构；堆（用new分配的）上的类对象会在delete的时候调用
写法是`~Entity()`

析构函数的调用顺序和析构函数正好相反，先构造的先析构（具体见 类的继承 章节）

### 构造函数的初始化列表
我们可以这样写**构造函数**（只有构造函数才能用初始化列表）
```cpp
class Entity {
private:
    std::string m_Name;
public:
    Entity() {
        m_Name = "Unknown";
    }
    Entity(const std::string& name) {
        m_Name = name;
    }
};
```
如果我们用`Entity a("Lith")`这样调用非默认构造函数来创建实例，是低效的，触发了**一次默认构造 + 一次拷贝赋值**：
1. 先给a分配内存
2. 对类中声明的m_Name变量初始化，m_Name是string类型，会调用string的默认构造函数，初始化为空字符串（构造）
3. 再进行构造函数里的赋值操作，这里是string类的拷贝（赋值）

```cpp
class Entity {
private:
    std::string m_Name;
    int m_Score;
public:
    Entity()
        : m_Name("Unknown"), m_Score(0) {}
    Entity(const std::string& name)
        : m_Name(name), m_Score(0) {}
};
```
写成这样，会直接使用参数构造函数，避免调用string的默认构造函数

**无论如何都优先使用初始化列表**

而且关于继承，如果基类没有默认构造函数，子类的构造函数也要使用初始化列表，这个可以见开头的例子

### 拷贝构造函数
`MyString(const MyString& other) {}`

```cpp
class MyString {
private:
    char* m_Buffer; // 指向字符缓冲区
    unsigned int m_Size;
public:
    MyString(const char* string) {
        m_Size = strlen(string);
        m_Buffer = new char[m_Size+1];
        memcpy(m_Buffer, string, m_Size+1);
    }
    ~MyString() {
        delete[] m_Buffer;
    }
    char& operator[](unsigned int index) {
        return m_Buffer[index];
    }
}
Mystring a = "hello";
Mystring b = a;
```
这里的拷贝是浅拷贝，只复制了指针，会造成两次释放！

因为调用了默认的拷贝构造函数：
```cpp
MyString(const MyString& other)
    : m_Buffer(other.m_Buffer), m_Size(other.m_Size) {}
```
所以深拷贝要实现自己的拷贝构造函数

另外，如果不允许拷贝，直接写`MyString(const MyString& other) = delete`

### 构造的隐式转换

单参数的构造函数除了`MyClass a(para)`这样的构造方法，`MyClass a = para`也会进行隐式转换来构造
比如

```cpp
class Distance {
private:
    double meters;
    
public:
    // 单参数构造函数：int → Distance
    explicit Distance(int km) : meters(km * 1000.0) {
        cout << "int构造: " << km << "km = " << meters << "m" << endl;
    }
    
    // 单参数构造函数：double → Distance
    Distance(double m) : meters(m) {
        cout << "double构造: " << m << "m" << endl;
    }
    
    void show() const {
        cout << meters << " meters" << endl;
    }
};
void printDistance(Distance d) {
    d.show();
}

int main() {
    Distance d1 = 3.5;     // 隐式转换：3.5 (double) → Distance    
    printDistance(2.8);    // 隐式转换：2.8 (double) → Distance
    
    
    // int版本的构造函数有explicit，禁止隐式转换
    // Distance d3 = 5;        // 错误：拷贝初始化不允许隐式转换
    // printDistance(3);       // 错误：参数传递不允许隐式转换
    
    Distance d2(5);        // OK
    printDistance(Distance(3)); // 显式构造 OK

    return 0;
}
```

## 类的继承
有层级结构的类可能有重复需要的代码，比如Player和Entity，可能都有坐标和移动的需要，那么就会造成代码重复
```cpp
class Entity {
public:
    float X, Y;
    
    void Move(float xa, float ya) {
        X += xa;
        Y += ya;
    }
};

class Player : public Entity {
public:
    const char* Name;
    
    void PrintName() {
        std::cout << Name << std::endl;
    }
};
```
这样Player就继承了Entity类的属性

`class Player : public Entity`里写的`public`意思是Entity类里各成员、方法继承可见性给Player（public还是public protected还是protected）
如果用`private`继承，Entity 类中的 public 和 protected 成员，在 Player 类中全部变成 private；如果是`protected`继承，则全变成 protected

子类的构造函数会自动调用基类的构造函数，就算不在初始化列表中写出来编译器也会自动调用
而且会自动调用默认的构造函数，如果缺失就会报错：
```cpp
class BaseLogger {
protected:
    string tag;
public:
    // 构造函数：接收标签名，输出 "[tag] Logger created"
    BaseLogger(const string& tag_name)
        : tag(tag_name) {
        cout << '[' << tag << ']' << " Logger created\n";
    }
    
    // 析构函数：输出 "[tag] Logger destroyed"
    virtual ~BaseLogger() {
        cout << '[' << tag << ']' << " Logger destroyed\n";
    }
    
    // 记录普通日志：输出 "[tag] " + msg
    void log(const string& msg) {
        cout << '[' << tag << ']' << " " << msg << "\n";
    }
};

class TimestampLogger : public BaseLogger {
private:
    int* timestamps;      // 动态分配的数组，存储时间戳
    int timestamp_count;  // 当前已存储的时间戳数量
public:
    // 构造函数：接收标签名和数组容量 capacity
    // 需要输出 "[tag] TimestampLogger created with capacity=X"
    TimestampLogger(const string& tag_name, int capacity)
            :BaseLogger(tag_name) { //只看这个地方！！！！！！！！！！！！！！
        timestamp_count = 0;
        timestamps = new int[capacity];
        cout << '[' << tag << ']' << " TimestampLogger created with capacity=" << capacity << "\n";
    }
    
    // 析构函数：输出 "[tag] TimestampLogger destroyed, total timestamps=X"
    ~TimestampLogger() {
        delete[] timestamps;
        cout << '[' << tag << ']' << " TimestampLogger destroyed, total timestamps=" << timestamp_count << "\n";
    }
    
    // 添加时间戳（模拟当前时间）
    void addTimestamp(int time_val) {
        timestamps[timestamp_count] = time_val;
        timestamp_count++;
    }
};
```
这里必须在初始化列表里写出来调用的是带参数的构造函数，否则编译器自动调用默认的，而基类里只有一个含参的，报错！
（已经写了一个构造函数（即使带参数），编译器认为你明确想要控制对象的构造方式，所以不会再生成默认构造函数）

### 构造和析构的顺序
构造时
1. 先构造基类（调用基类的构造函数）
2. 再构造子类的成员变量（按照它们在类中声明的顺序，而非初始化列表的顺序）
3. 最后执行子类自己的构造函数体

析构时正好相反
1. 先执行子类自己的析构函数体（此时子类成员还存活）
2. 再销毁子类的成员变量（按声明顺序的逆序）
3. 最后调用基类的析构函数

因此，不要在构造函数/析构函数中调用虚函数
在基类构造期间，子类尚未构造完成，此时虚函数表指向的是基类版本，不会多态到子类。同理，析构期间子类已被销毁，调用虚函数也只会执行基类版本。
```cpp
class Base {
public:
    Base() { foo(); }  // 永远调用 Base::foo，即使子类重写了它
    virtual void foo() {}
};
```

### 虚函数
虚函数允许子类继承父类的时候，在子类中重写父类的方法
```cpp
class Entity {
public:
    std::string GetName() { return "Entity"; }
};

class Player : public Entity {
private:
    std::string m_Name;
public:
    Player(const std::string& name)    //传入引用类型，防止name作为string的拷贝传值
        : m_Name(name) {}   //列表初始化，由冒号开始，直接调用m_Name作为string的拷贝构造函数，避免了先调用string的默认构造函数（只创建了空串），再赋值的两部操作
    
    std::string GetName() { return m_Name; }
};

int main() {
    Entity* e = new Entity();
    std::cout << e->GetName() << std::endl;
    
    Player* p = new Player("123");
    std::cout << p->GetName() << std::endl;
    
    Entity* entity = p;    //基类指针直接指向派生类对象，这是安全的，称为向上转型
    std::cout << entity->GetName() << std::endl;    //输出Entity而不是123
}
```
尽管entity指针的确指向了个Player类，但是entity指针的类型是Entity*，编译器只允许它访问Entity类的方法，所以会输出Entity
但我们希望C++能知道这个Entity实际上是Player，让它调用Player的GetName

因此需要虚函数，通过v表/虚函数表来实现编译，v表就是一个表，包含基类中所有虚函数的映射，这样就可以在运行时，将它们映射到正确的覆写/override函数
```cpp
class Entity {
public:
    virtual std::string GetName() { return "Entity"; }    //在基类里virtual关键字
    //注意，就算基类不打算实现该虚函数，也要写{}，否则只有函数声明没有实现
};

class Player : public Entity {
private:
    std::string m_Name;
public:
    Player(const std::string& name)
        : m_Name(name) {}
    
    std::string GetName() override { return m_Name; }    //结尾添加override关键字
    //实际起作用的是virtual，不写override也能动态绑定。override只是让编译器检查函数签名一致性
}
```
注意虚函数是有额外成本的，基类中要有一个成员指针指向v表，以及每次调用虚函数时要遍历这个表，来确定要映射到哪个函数

这种在运行时根据对象的实际类型来确定调用哪个函数的机制叫**动态绑定**，与之相对的，在编译时确定的是静态绑定

### 虚析构函数
**只要需要继承 就声明基类的析构函数为虚函数**

```cpp
class Base {
   public:
    Base() { std::cout << "Base Constructor\n"; }
    ~Base() { std::cout << "Base Destructor\n"; }
};
class Derived : public Base {
   public:
    Derived() { std::cout << "Derived Constructor\n"; }
    ~Derived() { std::cout << "Derived Destructor\n"; }
};
int main() {
    Base* base = new Base();
    delete base;
    std::cout << "-------\n";
    Derived* derived = new Derived();
    delete derived;
}
```
输出
```
Base Constructor
Base Destructor
-------
Base Constructor
Derived Constructor
Derived Destructor
Base Destructor
```
对于derived这个子类的实例，实际内存中也有父类的内容，所以两个构造函数都会触发，析构函数也同理

但是如果我们**用一个父类的指针**（多态，通用接口）
`Base* derived = new Derived();` //作为Derived子类被构造，调用两个构造函数
但是delete的时候作为Base类被析构，只会触发Base类的析构，只要子类里有堆分配就会造成内存泄漏

所以需要一个类似虚函数的东西
```cpp
class Base {
public:
    Base() { std::cout << "Base Constructor\n"; }
    virtual ~Base() { std::cout << "Base Destructor\n"; } //子类不需要override覆写
};
```
不是覆写析构函数 而是加上一个析构函数
会调用两个析构函数 会先调用派生类析构函数 然后调用基类析构函数（先构造基类再构造子类，析构顺序反过来）

**只要需要继承 就声明基类的析构函数为虚函数**

### 多继承和虚继承

可以让一个类继承多个类，比如``class A : public B, public C {``

但是可能遇到一个问题，如果继承的结构是这样的：
```
    Base (有 _baseVal)
   /    \
Mid1    Mid2   ← 各自包含一份 Base
   \    /
  Derived     ← 包含两份 Base，就有两个 _baseVal
```
- 内存浪费：Derived 对象里有两份 _baseVal
- 二义性：访问 _baseVal 时编译器不知道用哪一份，必须写 Mid1::_baseVal 或 Mid2::_baseVal

解决办法是，在Mid1和Mid2类继承Base类的时候，添加一个`virtual`关键字，同时在Derived类的构造函数里主动调用Base类的构造函数
```cpp
class Base {
public:
    int _baseVal;
    Base(int val) : _baseVal(val) {}
};

// 虚继承
class Mid1 : virtual public Base {
public:
    Mid1(int val) : Base(val) {}
    // 这行初始化会被 Derived 忽略
};
class Mid2 : virtual public Base {
public:
    Mid2(int val) : Base(val) {}
};

class Derived : public Mid1, public Mid2 {
public:
    // 最终派生类必须负责初始化虚基类
    Derived(int val) : Base(val), Mid1(val), Mid2(val) {}
};
```
虽然 Mid1 和 Mid2 的构造函数里也写了 Base(val)，但当它们作为 Derived 的一部分时，这些调用会被忽略
如果 Derived 不显式调用 Base(val)，编译器会用 Base 的默认构造函数（这里没有，所以会编译错误）

### 虚函数实现通用的接口
```cpp
class Animal {
public:
    virtual void speak() = 0;
    virtual ~Animal() {}
};

class Dog : public Animal {
    void speak() override { cout << “Woof !”; }
};

class Cat : public Animal {
    void speak() override { cout << “Meow !”; }
};

int main() {
    Animal* animals[] = {new Dog(), new Cat()};
    for (auto* a : animals) {
        a->speak();  // 通过统一接口调用不同实现
    }
}
```
通过基类统一接口操作不同派生类对象

### 纯虚函数 接口
纯虚函数允许我们在基类中定义一个没有实现的函数，然后强制子类去实现该函数
```cpp
class Entity {
public:
    virtual std::string GetName() = 0;
};
```
这样子Entity类就不能实例化，Player类继承Entity类，如果没有实现GetName函数，Player也不能实例化

## 异常处理
try/catch/throw
```cpp
#include <iostream>
using namespace std;

void testFunction(int value) {
    if (value < 0) {
        throw value;  // 抛出int
    }
    else if (value == 0) {
        throw "值为零";  // 抛出const char*
    }
    else if (value > 100) {
        throw 3.14;  // 抛出double
    }
    cout << "正常执行，value = " << value << endl;
}

int main() {
    int num;
    cout << "输入一个数字: ";
    cin >> num;
    
    try {
        testFunction(num);
    }
    catch (int e) {
        cout << "捕获到整数异常: " << e << " (负数错误)" << endl;
    }
    catch (const char* e) {
        cout << "捕获到字符串异常: " << e << endl;
    }
    catch (double e) {
        cout << "捕获到浮点数异常: " << e << " (数值过大)" << endl;
    }
    catch (...) {  // 捕获所有未指定的异常
        cout << "捕获到未知类型的异常" << endl;
    }
    
    return 0;
}
```

除了抛出这些简单的异常对象，在`<stdexcept>`下提供了几个定义好的异常类：`std::out_of_range`等
```cpp
int largest_proper_divisor(int n) {
    if (n == 0) {
        throw invalid_argument("largest proper divisor is not defined for n=0");
    }
    if (n == 1) {
        throw invalid_argument("largest proper divisor is not defined for n=1");
    }
    for (int i = n/2; i >= 1; --i) {
        if (n % i == 0) {
            return i;
        }
    }
    return -1;
}

void process_input(int n) {
    try {
        int d = largest_proper_divisor(n); // 如果抛出意外，try内部后面的代码不会执行
        cout << "result=" << d << endl;
    }
    catch (const invalid_argument& e) { // 传入常量引用
        cout << e.what() << endl;
    }
    cout << "returning control flow to caller" << endl;
}

int main() {
    int n;
    cin >> n;
    process_input(n);
    return 0;
}
```

另外，`<exception>`头文件只提供了`exception`基类，自定义的异常类可以继承这个基类

## 其他关键字

大部分修饰函数的关键字都写在函数返回类型的前面，比如`virtual void process()`

只有override noexpect const等少数关键字是写在函数名后面的

### 结构体和类外的static
多文件编译的时候，类似于class内的private变量，static声明的静态变量仅在自身的翻译单元可见，例如：
```cpp
//file A
int a = 10;
//file B
#include <stdio.h>
extern int a;
int main() {
    printf("%d",a);
    return 0;
}
```
在A B文件链接的时候，extern关键字会使得变量a在外部文件中寻找，最后可以正常输出10
如果A文件中`static int a = 10;`，就会链接失败

### 结构体和类内的static
在类内声明一个static变量，意思是这个变量不属于某个实例，而是**属于整个类**
往往用 `s_` 前缀命名，区分静态成员变量和非静态的
所有实例都共享这个变量，可以直接用`className::variables`来引用它
但是类内只是完成声明，静态成员变量需要在类外部进行定义和初始化，一般是全局部分（main等各个函数和类之外，因为static变量储存在全局/静态区）完成定义，比如`int entity::x = 0;`

静态成员函数/类方法与之类似，因为属于整个类，可以不创建实例就用`className::func`去调用
而且这个函数不会传入this指针，只能访问静态成员变量（因为不传入this，无法确定修改的是什么哪个成员变量）

### 局部静态的static
静态局部变量，生存期基本上相当于整个程序的生存期，但作用域只在这个函数内
存储在全局区，所以所在函数调用后不会被摧毁

### const
变量的const
const int* a或者int const* a
const在*左边 表示指针指向的内容是常量 而指针本身可变

int* const a
const在*右边 表示指针本身是常量 但指向的内容可以修改

类方法后面的const
```cpp
class Entity {
private:
    int m_X, m_Y;
public:
    int GetX() const {    //在类的方法名之后const 意思是这个方法不会修改任何实际的类
        //m_X= 2; 不合法
        return m_X;
    }
};
```
const对象只能调用const方法

**参数包含类的常量引用**的函数如果调用方法，这个方法必须标注const
```cpp
void PrintEntity(const Entity& e) {
    //这个函数传入的是常量引用，不能修改e，然而调用的GetX()方法如果没标注const，就不能调用，因为不能保证它不会修改e
    std::cout << e.GetX() << std::endl;
}
```

### mutable 可修改
```cpp
class Entity {
private:
    int m_X, m_Y;
    mutable int var;
public:
    int GetX() const {
        var = 2;    //标记为const的方法也可以修改类了
        return m_X;
    }
};
```

### explicit 禁止构造函数使用隐式转换

- 防止构造函数进行的隐式转换
- 防止类型转换运算符进行的隐式转换（C++11起）

```cpp
class MyVector {
private:
    int size;
public:
    explicit MyVector(int s) : size(s) {}
};

// 不使用explicit的问题：
// MyVector v = 10;  // 看起来像是把10赋值给向量，语义不清
// void func(MyVector v);
// func(10);         // 莫名其妙传个数字进去

// 使用explicit后
MyVector v1(10);          // 明确表示构造
MyVector v2 = MyVector(10); // 明确表示构造
```

### noexcept 函数不会抛出异常

## STL
array vector set map deque ...
STL 是 Standard Template Library（标准模板库）的缩写

常见的公共成员函数包括：
size(): 返回容器中元素的数量。对于 std::array 这种固定大小的容器，size() 是一个编译时常量。
empty(): 检查容器是否为空（即 size() == 0）。这是一个更清晰、更推荐的判断容器是否为空的方式，而不是比较 size() == 0。
clear(): 删除容器中的所有元素，使其变为空。注意：std::array 因为大小固定，没有 clear() 函数。

此外，STL是遵循RAII**自动管理内存**的，当容器对象销毁时（离开作用域、被 delete 等），其析构函数会自动释放所有分配的内存，不需要手动调用 free 或 delete。

### 静态数组 array
首先c++可以用c风格的数组，也就是`int arr[5]`这种

`std::array<int,3> arr1 = {1,2,3};` 适合小数据、固定大小、追求极致性能
储存在栈上，不初始化为垃圾值
`arr1`就是一个类对象，而不会转换成指针（c的数组名），因而支持直接拷贝`arr1 = arr2`（类型相同时）
传参的时候不是传递指针，而是值传递对象本身。所以传递时要用引用
随时可以通过`arr1.size()`获取长度而不用单独传递长度参数（然而size是作为模板参数提供的，所以不会有额外内存消耗）

一些成员函数：

访问
at(index)：安全地访问元素。它会进行边界检查，如果索引超出范围，会抛出`std::out_of_range`异常
operator[] (index)：通过索引访问元素。不进行边界检查，行为类似 C 数组。速度快，但不安全（也就是通过arr[0]这样的c风格访问）
front()：访问第一个元素
back（）：访问最后一个元素。
data()：返回指向内部存储数据的原始指针（T*）
和string同样，只能通过data来获得指针，array对象的名不会作为指向首元素的指针

### 动态数组 vector
`std::vector<int> vec = {1,2,3}` 适合大数据、大小未知的动态数组，可以很方便的扩容，频繁增删
数据储存在堆上，但是实例在栈上（指针，size等成员变量）
不初始默认数据为0

初始化
vector<int> vec;    //分配了0空间
vector<int> vec(10);    //分配了10空间，并且所有数据初始化为0
注意这样做，除了分配空间，**还会调用默认构造函数创建默认的对象，相当于vector已经有了10个元素**
如果只想预留空间，例如已知将来要插入 10 个元素，但想先构造后再push_back，用reserve
vector<int> vec(10,2);  //分配了10空间，所有数据初始化为2

常用成员函数：
front(): 访问第一个元素。
back(): 访问最后一个元素。

push_back(value): 在 vector 的末尾添加一个新元素。这是增加元素最常用的方法。
pop_back(): 删除 vector 的最后一个元素。

clear(): 删除 vector 中的所有元素，使其变为空。
insert(iterator, value): 在指定位置插入一个或多个元素。
erase(iterator) / erase(start_it, end_it): 删除指定位置或范围内的元素。
注意这里传入的是迭代器，比如删掉[2]元素，应当传入`.begin()+2`；删除[1]-[3]的元素，传入`.begin()+1, .begin()+4`，因为迭代器左闭右开，这里不会包括.begin()+4也就是[4]
另外，erase之后，元素位置会自行前移

reserve(new_capacity): 预先分配至少能容纳 new_capacity 个元素的内存空间，以避免多次重新分配，提高性能
resize(new_size): 改变 vector 的大小。如果新大小更大，会用默认值填充新空间；如果更小，则会销毁多余的元素。
capacity(): 返回 vector 当前分配的存储空间能容纳多少元素。
size()：返回 vector 中已经有多少元素。
capacity()通常大于或等于 size()，以便为 push_back 等操作预留空间，减少内存重新分配的次数。

迭代器 (begin, end, rbegin, rend): 同样提供标准迭代器接口。

**二维数组**
vector<vector<int>> a;
可以进行如下的初始化，一个3*2矩阵，全填充0，避免写循环
vector<vector<int>> a(3,vector<int>(2,0));

**使用优化**
emplace_back(parameters...) 传递参数而不是实际对象
```cpp
struct point {
    int m_x, m_y, m_z;

    point(int x, int y, int z)
        : m_x(x), m_y(y), m_z(z) {}

    point(const point &other) 
        : m_x(other.m_x), m_y(other.m_y), m_z(other.m_z) {
        cout << "Copied!" << endl;
    }
};

int main() {
    vector<point> a;
    a.reserve(2);
    a.push_back({1,2,3}); //先{1,2,3}构造一个栈上的临时的point实例，然后再拷贝到vector的堆内存里，最后摧毁栈上的临时对象
    a.push_back({4,5,6}); //开销：涉及一次额外的构造 + 一次拷贝/移动构造 + 一次析构

    return 0;
}
```
输出2个"Copied!"
如果使用`a.emplace_back(1,2,3);`，直接传递参数给构造函数，在堆上创建对象，没有临时对象，仅涉及一次构造，不会发生拷贝

如果要新建一个临时元素放进去，用 emplace_back；如果要放入已有的元素，用 push_back

### string 字符串类
注意，c++的string是类，没有指针算数操作，`string s`的s也**不能被视作指针**
如果想用printf输出string，用`.cstr()`

1. 用+运算符来拼接，用==比较（运算符重载）
2. 用`s.length()`获取s的长度，虽然结尾依旧有'\0'，但是获取长度不依赖'\0'，所以串内可以储存的'\0'也会被完整打印（不是strlen那样低效率的遍历）
3. 自动内存管理

string头文件下还有其他函数
1. `to_string(data)`函数将各类数字转换为string，注意浮点数默认有六位小数
2. `stoi()` `stod()`等各类函数可以把string转换为int/double/long/...
3. 用`s.substr(n,m)`来获取“从index为n开始，长度为m”的子串，如果只有n一个参数，就一直到结尾

### 集合 set/哈希数组 unordered_set
一个元素递增且互异的数据结构，基于红黑树
初始化
set<int> s = {1,2,3,3};    //可以这样初始化，会自动排序+删除多余元素（这其实是现代c++引入的特性）

insert(data)
erase(data)
find(data)：查找，返回指针，会逐一遍历直到最后一个元素之后，也就是`s.find(4) != s.end()`成立说明4在s中

unordered_set基于哈希实现，插入搜索比set快，然而无法有序遍历，适合只需要插入搜索的情景

### 键值对 map/哈希表 unordered_map
存储 <Key, Value> 键值对，默认按 Key 升序排列，Key 必须唯一，重复插入会覆盖或失败
基于红黑树查找、插入、删除均为 O(log n)

初始化
map<string,int> a = {{"Alice", 90}, {"Bob", 85}};
map<string,int> a(10); //创建10个桶

m["One"] = 1;	Key 不存在则创建，存在则覆盖
m.insert({"Two", 2});	若 Key 已存在，插入失败
m["One"] 这样访问，如果不存在会创建一个空键值对！
m.at("One")	安全访问，Key 不存在抛异常

m.find("One")	返回迭代器，未找到返回 end()
m.count("One")	返回 1 (存在) 或 0 (不存在)

m.erase("One")	删除指定 Key 的元素
m.clear()	清空所有元素

如果不需要顺序，用unordered_map可实现O(1)的效率，基于哈希表

### 栈 stack/队列 queue
stack<int> s;
栈只对栈顶的元素进行操作，后进先出 LIFO

s.push(val); 压栈
s.pop(); 出栈
s.top(); 访问栈顶，**返回栈顶元素的引用**
s.empty(); 是否为空，返回bool

queue<int> q;
队列中，新进入的元素在队尾，弹出的元素是队首，先进先出 FIFO

push(val) 加入队尾
pop() 移除队头元素
front()	返回队头元素的引用
back() 返回队尾元素的引用
empty()	是否为空，返回bool

### 双端队列 deque/优先级队列 priority_queue
可以在头部和尾部高效地进行插入和删除
支持通过下标 [] 或迭代器进行随机访问（类似 vector），但速度略慢于 vector
由一系列连续的内存块组成（分段连续），因此扩容时不需要像 vector 那样移动所有元素

push_back(val)	尾部插入
push_front(val)	头部插入 (vector 不支持)
pop_back()	尾部删除
pop_front()	头部删除 (vector 不支持)
operator[] / at()	随机访问元素
begin(), end()	支持迭代器遍历

本质是一个容器适配器，默认基于 vector 实现大顶堆，保证队首元素始终是最大值
插入和删除的时间复杂度为 O(log n)，访问队首元素为 O(1)
priority_queue<int> pq;                           // 默认大顶堆，最大优先
priority_queue<int, vector<int>, greater<int>> pq; // 小顶堆，最小优先
priority_queue<int> pq(nums.begin(), nums.end()); // 用区间元素初始化

### 双向链表 list
底层为双向链表，支持任意位置 O(1) 插入/删除（已知迭代器），但不支持随机访问，查找需 O(n)

初始化
list<int> a = {1, 2, 3, 4, 5};
list<int> a(10);          // 创建10个默认值0的元素
list<int> a(10, 5);       // 创建10个值为5的元素

头部/尾部操作
a.push_front(0);          // 头部插入元素
a.push_back(6);           // 尾部插入元素
a.pop_front();            // 删除头部元素
a.pop_back();             // 删除尾部元素
a.front();                // 返回头部元素的引用
a.back();                 // 返回尾部元素的引用

插入与删除
auto it = a.begin();
advance(it, 2);
a.insert(it, 10);         // 在指定位置前插入元素
a.erase(it);              // 删除指定位置的元素
a.remove(3);              // 删除所有值为3的元素
a.clear();                // 清空所有元素

特有操作
a.sort();                 // 链表排序（成员函数，非std::sort）
a.reverse();              // 反转链表
a.unique();               // 删除相邻重复元素（需先排序）

list<int> b = {7, 8, 9};
a.merge(b);               // 合并两个已排序链表，b变为空
a.splice(it, b);          // 将b的全部元素移动到it位置前

## 迭代器
一种类似指针的东西，为遍历不同容器提供了通用的接口，无论容器的内存分布是连续的还是分割的（vector/链表）
因为有运算符重载，使用起来就像指针，有++，解引用等操作

<iterator>头文件提供了很多迭代器相关的工具函数

### 简单遍历
begin(): 返回一个指向容器中第一个元素的迭代器。
end(): 返回一个指向容器中最后一个元素**之后位置**的迭代器。
（还有 rbegin(), rend()用于反向迭代，循环还是写迭代器++）
可以这样手写遍历：
```cpp
int main() {
    std::vector<int> vec = {10, 20, 30, 40, 50};
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";  // 把迭代器看成指针，解引用
    }

    // map 键值对，迭代器指向 pair
    std::map<int, std::string> students = {{1, "Alice"}, {2, "Bob"}, {3, "Charlie"}};
    for (auto it = students.begin(); it != students.end(); ++it) {
        // first,second = key,val
        std::cout << it->first << ":" << it->second << "  ";
    }

    return 0;
}
```
虽然更常用的是基于范围的for遍历，这里虽然底层实现还是迭代器，但是提供给用户的是元素的副本
```cpp
for (auto p : vec) {
    cout << p << " ";
}
```

对于vector deque array，提供的是**随机访问迭代器**，可以进行`it+2`这样跟指针运算一样的操作
但是对于list set map，只能用`std::advance(it, 2);`代替

### 插入删除
使用 back_inserter 向容器尾部插入
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
std::vector<int> dest;
std::copy(v.begin(), v.end(), std::back_inserter(dest));
// dest = {1, 2, 3, 4, 5}
```

使用 erase 删除当前迭代器的元素后返回下一个迭代器
```cpp
auto it = v.begin();
while (it != v.end()) {
    if (*it % 2 == 0)
        it = v.erase(it);  // erase 返回被删元素的下一个
    else
        ++it;
}
// v = {1, 3, 5}
```

迭代器失效
在对容器进行某些操作（插入、删除、扩容）后，已有的迭代器不再指向有效元素
```cpp
    // 在遍历过程中 erase，迭代器失效
    for (auto it = v.begin(); it != v.end(); ++it) {
        if (*it % 2 == 0)
            v.erase(it);  // it 失效，++it 是未定义行为
    }
```

## algorithm和numeric
需要给出迭代器作为参数来指定范围
提供条件需要的参数（或称为谓词）是一个函数/函数对象/lambda表达式，其参数类型为相应容器的元素类型，返回值类型为bool

(nums.begin(),nums.begin()+k,0) 迭代器左闭右开，实际上就是对前k个元素操作

### accumulate/reduce (c++17)
numeric头文件下
用于求和、求积、查找最大值/最小值、自定义逻辑合并（如拼接字符串、合并对象状态等）
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// 求和
int sum = std::accumulate(v.begin(), v.end(), 0);
std::cout << sum << std::endl;  // 15

// 求积（初始值要用 1）
int product = std::accumulate(v.begin(), v.end(), 1,
                              [](int a, int b) { return a * b; });
std::cout << product << std::endl;  // 120

// 拼接字符串
std::vector<std::string> words = {"Hello", "World", "C++"};
std::string result = std::accumulate(words.begin(), words.end(),
                                     std::string(""),
                                     [](const std::string &a, const std::string &b) {
                                         return a.empty() ? b : a + " " + b;
                                     });
std::cout << result << std::endl;  // Hello World C++
```

reduce和accumulate的区别是支持并行操作

### inner_product
```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = {4, 5, 6};

// 1*4 + 2*5 + 3*6 = 4 + 10 + 18 = 32
int dot = std::inner_product(a.begin(), a.end(), b.begin(), 0);
std::cout << dot << std::endl;  // 32
```

### iota 填充递增序列
```cpp
std::vector<int> v(5);
std::iota(v.begin(), v.end(), 1);   // v = {1, 2, 3, 4, 5}
std::iota(v.begin(), v.end(), 10);  // v = {10, 11, 12, 13, 14}
```

### sort 排序
sort(s.begin(),s.end()) 默认从小到大，递增序
需要从大到小，第三个参数写`std::greater`

第三个参数可以自定义cmp函数，或者lambda表达式，注意cmp返回值表达式不能包含等号：
```cpp
std::sort(v.begin(), v.end(), [](int a, int b) {
    return a > b;  // 降序
});
```

不稳定，如果需要保持相等元素的原始相对顺序，用`std::stable_sort`

### find find_if/count count_if
返回迭代器类型

find(s.begin(),s.end(), val)
查找特定值
find_if(s.begin(),s.end(),[](){})
用于在指定的迭代器范围内查找第一个满足特定条件的元素

count(InIt first, InIt last, T val)
返回符合值的元素个数
count_if(InIt first, InIt last, Pred cond)
返回范围内符合条件的元素个数

### copy/fill
```cpp
std::vector<int> src = {1, 2, 3, 4, 5};
std::vector<int> dst;

// 拷贝到 dst 尾部（需要先确保 dst 有空间或用 back_inserter）
std::copy(src.begin(), src.end(), std::back_inserter(dst));

// 拷贝前3个元素
std::vector<int> dst2(3);
std::copy(src.begin(), src.begin() + 3, dst2.begin());

// copy_if: 有条件拷贝
std::vector<int> evens;
std::copy_if(src.begin(), src.end(), std::back_inserter(evens),
             [](int x) { return x % 2 == 0; });  // evens = {2, 4}
```

```cpp
std::vector<int> v(5);
std::fill(v.begin(), v.end(), 42);          // v = {42, 42, 42, 42, 42}

std::fill_n(v.begin(), 3, 100);             // v = {100, 100, 100, 42, 42}
```

### reverse
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
std::reverse(v.begin(), v.end());           // v = {5, 4, 3, 2, 1}
```

### max_element/min_element
```cpp
std::vector<int> v = {3, 7, 2, 9, 5};

auto max_it = std::max_element(v.begin(), v.end());
auto min_it = std::min_element(v.begin(), v.end());

std::cout << "最大值: " << *max_it << std::endl;  // 9
std::cout << "最小值: " << *min_it << std::endl;  // 2
```