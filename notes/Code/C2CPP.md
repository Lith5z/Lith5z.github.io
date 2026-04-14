---
layout: default
title: "语法 从C到C++"
date: 2026-04-14
tags: [编程]
---

# 从C到C++ 语法整理

<details markdown="1">
  <summary>📑 点击展开目录</summary>
  
  1. 目录
  {:toc}
</details>

## 说在前面

学再多语法也**不如自己动手写**，千万**不要只听课不写当耐听王**！

cpp的语法很复杂，不自己写一下几天后就会全部忘掉，这里推荐一个简单cpp语法练习网站：
[HackerRank-cpp practice](https://www.hackerrank.com/domains/cpp)

尤其练习Classes和Inheritance板块，可以很好的练习简单的OOP语法

但是真正学习OOP还得做点货真价实的项目，这块我正在尝试，之后补充吧~

课程来自Cherno-C++

别人的[笔记](https://chocomintopia.github.io/)

49-51 跳过库管理
62-63 跳过线程、计时
72 74 76-84 跳

## 一些简单的差别
### include
cpp兼容c的标准库头文件 <br>
比如c的`#include<string.h>`就有一个cpp版本是`#include<cstring>` <br>
两者都可以使用，只不过前者给函数和宏放入全局命名空间，后者放入std空间，推荐后者

### 简单io
cin >> n <br>
相当于`scanf("%d",&n)`，只不过编译器自动推断了格式符，以及自动进行了取地址 <br>
对于string类型，就相当于`scanf("%s",s)`，所以遇到空白符就停止读入 <br>
需要读入整行，用`getline(cin,s)`，这个函数在<string>的头文件下。并且可以从文件流中读取数据

### using
有两个主要用途，引入命名空间、类型别名（类似typedef）

**命名空间 namespace** <br>
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

如果写`using namespace MyCompany`，这个语句作用域中的所有String等定义的东西都会默认是MyCompany下定义的 <br>
不推荐到处写`using namespace std`，要尽量缩减这样写的作用域，千万不要在头文件里这样写（会污染所有带了这个头文件的代码）

`using apple::print;` <br>
是针对性的，把 apple 命名空间里所有的print引入当前作用域，就在当前作用域中可以直接使用print而无需指明`apple::`



**类型别名** <br>
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
// 原题是利用RTTI进行类型验证

const int* const_ptr = &a;
int* non_const_ptr = const_cast<int*>(const_ptr); // 移除常量指针的const属性，不能对常量这样用，UB

char* char_ptr = reinterpret_cast<char*>(&num); // 直接在二进制层面重新解释数据
```

可以代替c的强制类型转换

## 现代c++的新特性
如果没有特别说明，则是c++11

### auto 自动类型推定
编译器会根据初始化时的值来自动推断变量类型，比如`auto i = 1`自动推断为int，但是必须要做初始化 <br>
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

常用的使用情况就是迭代器/for-each遍历，或者模板这种类型名非常长的情况下 <br>
对于一般的int这种基本类型，没必要用auto

### 通用的`{}`列表初始化
暂无

### lambda表达式
TODO 整理lambda的知识点

只需要使用一次的函数，可以在使用处定义，而且还能捕获上下文中的外部变量而不是传入参数 <br>
`auto lambda = []() { std::cout << "Hello"; };` <br>
常用在sort、find_if里

[]为捕获类型，决定对外部变量要如何访问 <br>
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

()内的是参数 <br>
{}内的是函数体，和一般函数一样

返回类型会由编译器自行决定，如果要指定类型，写
```cpp
auto lambda = []() -> double { 
    if (condition) return 42;   // 转换为 double
    return 3.14; 
};
```

每个 lambda 都有一个唯一的、不可命名的类型 <br>
如果要把lambda表达式传给其他函数，那个函数的参数有两种写法 <br>
1. 函数指针 只有不捕获变量的才行
比如`[](int &a){return a + 1;}` 类型可以被隐式转换为`int(*func)(int &a)`
2. function
引入`functional`头文件后，调用函数写`(const function<int(int&)> &func, ...)`，也会被转换
但是有虚函数等额外开销
3. 模板（最优）
`template <typename T>`之后，函数参数写`(T &&func, ...)`，完美转发，零开销

### 智能指针
用于自动管理动态内存的**类**模板，它们确保在适当的时候自动删除所指向的对象，防止内存泄漏 <br>
需要`#include <memory>`

实际上，智能指针不是个指针，而是一个类，只不过能实现指针的功能，而且会自动根据作用域进行delete <br>
也就是说，原本`entity *p`的p是个指针变量，如果打印出来，就是个地址；而`unique_ptr<entity> p`的p是个类的实例，不能直接打印 <br>
可以通过`.get()`得到真正的地址

**独占所有权的unique_ptr** <br>
当超出作用域的时候，自动delete <br>
性能与裸指针相当

初始化`unique_ptr<Entity> ptr1 = make_unique<Entity>();` 推荐，传入的参数会传递给Entity类的构造函数，而且异常安全 <br>
或`unique_ptr<Entity> ptr2(new Entity());`

不能拷贝，只能移动，也就是说`unique_ptr<Entity> ptr3 = ptr2`是报错的 <br>
（防止两个指针指向一个内存，造成多次释放）

**共享所有权的shared_ptr** <br>
当引用个数为0，自动delete <br>
额外维护引用计数的开销

初始化`std::shared_ptr<Entity> sharedE1 = std::make_shared<Entity>();` 推荐，一次分配，缓存友好 <br>
`std::shared_ptr<Entity> sharedE2(new Entity());` 这种写法先用new给实例分配空间，把指针传入ptr后，ptr再分配引用计数的空间，两次分配，而且内存不连续

```cpp
void func() {
    std::shared_ptr<Entity> sp1 = std::make_shared<Entity>();
    {
        std::shared_ptr<Entity> sp2 = sp1;  // 引用计数变为2
        // 使用 sp2...
    }  // sp2 销毁，引用计数变为1
    
    // 继续使用 sp1...
}  // sp1 销毁，引用计数变为0，Entity对象被删除
```

**weak_ptr** <br>
复制shared_ptr时，用这个不会增加引用次数

解决循环引用：在父子关系或双向引用中，一方使用 weak_ptr <br>
缓存/观察者模式：需要知道对象是否还存在，但不延长生命周期 <br>
避免悬空指针：通过 lock() 可以安全地检查对象是否还在

### numeric头文件
accumulate/reduce (c++17) <br>
用于求和、求积、查找最大值/最小值、自定义逻辑合并（如拼接字符串、合并对象状态等）

reduce(nums.begin(),nums.begin()+k,0) 迭代器左闭右开，实际上对前k个元素操作。从初始值为0开始 <br>
可选的第四个参数可以是函数，默认求和 <br>
和accumulate的区别是支持并行操作

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

## 指针和引用
### 内存分配 new/delete
`int *a = new int(6)`，括号内留空则是初始化为0，或者用c++11起通用的`{}`初始化 <br>
对于类 `Entity* e = new Entity()` 推荐这样写，除了调用类中的构造函数外，对于**int等类型也会初始化为0** <br>
如果`int *a = new int`，则不初始化，垃圾值

`int* nums = new int[5]{1, 2, 3, 4, 5}` 对于数组，不用自己写乘法计算大小 <br>
`delete a`以及对应数组的`delete[] arr;`

和malloc不同的是，如果分配失败，不用手动`if (p == NULL)`检查，会自动抛出异常 <br>
以及new/delete会自动调用构造函数

TODO placement new

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

## 运算符重载
运算符其实就是个自带函数名的函数，如同函数的重载，c++里可以给运算符定义自己需要的功能
```cpp
struct complexNumber {
    int re, im;
    complexNumber(int re, int im) 
        : re(re), im(im) {}
    complexNumber add(const complexNumber &other) const {
        return complexNumber(re+other.re,im+other.im);
    }
    complexNumber operator+(const complexNumber &other) const {
        return Add(other)
    }
}
```
这里就定义了运算符的重载，这样要两个复数相加的时候，就不用写成`result = a.add(b)`而是直接写`result = a + b`

也可以重载`<<` `==` 之类的运算符，关于流的运算符重载，需要注意重载的两种方法：
1. 在类中声明，类外定义，使用友元访问数据
```cpp
class Box {
private:
    int length, breadth, height;
public:
    int getLength() {return length;}
    int getBreadth() {return breadth;}
    int getHeight() {return height;}
    friend ostream& operator<<(ostream &os, Box &b);
    //参数必须写Box &b，友元不是类的方法，不会传入this指针
    //友元，使得<<能以b.的形式访问
};

ostream& operator<<(ostream &os, Box &b)  {
    //os << 而不是cout 为了支持多种流
    os << b.length << " " << b.breadth << " " << b.height;
    return os;
}
```
推荐这种做法

2. 类外声明，相当于一个全局函数
```cpp
//类内什么都不用写
ostream& operator<<(ostream &os, Box &b)  {
    os << b.getLength() << " " << b.getBreadth() << " " << b.getHeight();
    //用getter接口函数
    return os;
}
```

可以把运算符和有相同功能的函数都实现出来 使用的人可以自行选择

## 模板
模板函数若没调用是不会编译的，对于不同的类型，会根据模板生成不同函数

```cpp
template<typename T>
void Print(T value) {
    std::cout << value << std::endl;
}
Print(5);
Print("Hello"); //编译器会隐式推断类型
Print<int>(5); //也可以显示给出 STL就是这样实现的
```
因为cout支持多个类型，这样就可以直接调用了

```cpp
template <typename T, int N>
class MyArray {
private:
    T[N] m_array;
public:
    int size() const {
        return N;
    }
};
```
除了`typename`，还可以有其他参数 <br>
因为模板是在编译期生成代码的，数组的大小也需要在编译时决定，所以可以利用这个创建可变大小的栈数组

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
类是一个包含了数据（成员变量）和函数（方法）的封装，类类型的变量称为对象，新的对象变量称为实例 <br>
区分：局部变量、成员变量、全局变量

一般而言，成员变量用 `m_` 前缀命名，区分函数中出现的临时的局部变量（虽然常用于private的成员变量） <br>
也常用给局部变量前面加下划线的区分方法

可见性，类中的数据默认是私有/private的，加上public才能被外界访问，否则只能被类中的方法访问


类方法 <br>
如果是非静态的，在类外需要通过`obj.func()`来调用，类内可以直接调用 <br>
这时候类中的方法会默认传入`this`指针，来自调用该方法的那个对象，允许这个方法访问类的数据而不需要传参 <br>
静态方法static，在类外必须写`className::func()`来调用，类内可以直接写 <br>
（非静态的方法如果也不需要依据实例的话，推荐按静态方法的方式处理，避免命名空间问题）

实例化 <br>
`Entity a("name")` 调用带参数的构造 栈上 推荐 <br>
`Entity a` 调用默认构造，在栈上 <br>
`Entity e2 = Entity("123")` 栈上 拷贝初始化 不推荐 虽然c++17之后会直接初始化

`Entity *a = new Entity()` 堆上 推荐 除了调用类中的构造函数外，对于**int等类型也会初始化为0** <br>
`Entity *a = new Entity` 在堆上 不推荐

### this 指针
在类的（非静态）方法中，编译器会自动传入一个参数 this 指针，指向调用该方法的实例（不存在调用静态方法的实例，所以静态方法不传入this指针） <br>
类型是`className* const`，如果是const方法就是`const className* const`

有以下用途： <br>
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
**链式调用** <br>
比如`person.func1().func2()`这样的操作，这里的func1()就返回了`person&`类型，可以继续调用下一个函数（如果写成void，效果是一样的，但是就不能链式调用了）

### 结构体
c++中的**结构体也可以包含函数**，所以struct和class只有可见性的区别，struct默认是公开的，而class是私有的 <br>
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
private 只有类的作用域中（就是{}之间的代码）可以访问，（实际上就只能被该类自己的成员函数访问），**类的实例**和类外都不能访问，友元除外 <br>
protected 在基类和子类中可以访问，但是实例和类外不能 <br>
public 不限制访问

### 友元 friend
允许特定的**非成员函数**或**其他类**访问当前类的 `private` 和 `protected` 成员。 <br>
打破封装限制，用于紧密协作的类或运算符重载。

友元函数 <br>
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
这里相当于只是在类内做了一个身份的声明，允许printWidth访问Box类的数据，但是实际上这个函数就是一个普通的全局函数，不是类的方法 <br>
因此也不会传入this指针，因而参数要写`Box box`

友元类 <br>
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

友元成员函数 <br>
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
    Book(const char* _name);
    Book(const Book &other);
    ~Book();
    char* get_name() const;
    void set_name(const char* _name);

    static int get_cnt();

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
构造函数`Entity()`在**创建实例**的时候自动运行，反之不会，比如调用类的静态方法时

构造函数也可以带参数，通过重载实现多个构造函数
```cpp
//...
    Entity(float _X, float _Y) {
        X = _X;
        Y = _Y;
    }
```

如果不写构造函数，编译器也会自动创建一个默认构造函数，但是什么都不会做 <br>
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

析构函数里的内容一般针对的是类在堆上分配的数据 <br>
在栈上的对象会在离开作用域的时候调用析构函数（main中的会在程序结束运行时），所以可以不手动析构；堆（用new分配的）上的类对象会在delete的时候调用 <br>
写法是`~Entity()`

析构函数的调用顺序和析构函数正好相反，先构造的先析构

### 构造函数的初始化列表
我们可以这样写**构造函数**（普通函数不能用初始化列表）
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
2. 对类中声明的m_Name变量初始化，m_Name是string类型，会调用string的默认构造函数，初始化为空字符串（第一次构造）
3. 再进行构造函数里的赋值操作，这里是string类的拷贝（第二次赋值）

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

### 拷贝函数
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
这样Player就继承了Entity类的属性，注意**基类里只有Public才会被继承** <br>
`class Player : public Entity`里写的`public`意思是Entity类里各成员、方法继承可见性给Player（public还是public protected还是protected） <br>
如果用`private`，Entity 类中的 public 和 protected 成员，在 Player 类中全部变成 private。结果就是Player类的外部无法访问基类的这些成员，用于不希望使用者把派生类当作基类来用时（极少用）

子类的构造函数会自动调用基类的构造函数，就算不写出来编译器也会自动调用 <br>
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
这里必须在初始化列表里写出来调用的是带参数的构造函数，否则编译器自动调用默认的，而基类里只有一个含参的，报错！ <br>
（已经写了一个构造函数（即使带参数），编译器认为你明确想要控制对象的构造方式，所以不会再生成默认构造函数）

可以让一个类继承多个类，比如``class A : public B, public C {``

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
尽管entity指针的确指向了个Player类，但是entity指针的类型是Entity*，编译器只允许它访问Entity类的方法，所以会输出Entity <br>
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
}
```
注意虚函数是有额外成本的，基类中要有一个成员指针指向v表，以及每次调用虚函数时要遍历这个表，来确定要映射到哪个函数

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

但是如果我们**用一个父类的指针**（多态，通用接口） <br>
`Base* derived = new Derived();` //作为Derived子类被构造，调用两个构造函数 <br>
但是delete的时候作为Base类被析构，只会触发Base类的析构，只要子类里有堆分配就会造成内存泄漏

所以需要一个类似虚函数的东西
```cpp
class Base {
public:
    Base() { std::cout << "Base Constructor\n"; }
    virtual ~Base() { std::cout << "Base Destructor\n"; } //子类不需要override覆写
};
```
不是覆写析构函数 而是加上一个析构函数 <br>
会调用两个析构函数 会先调用派生类析构函数 然后调用基类析构函数（先构造基类再构造子类，析构顺序反过来）

**只要需要继承 就声明基类的析构函数为虚函数**

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
    virtual std::string GetName() = 0; //修改了
};
```
这样子Entity类就不能实例化，Player类继承Entity类，如果没有实现GetName函数，Player也不能实例化

纯虚函数类似其他语言的接口，C++里没有接口，这里依旧是个类

## static extern const mutable explicit
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
在A B文件链接的时候，extern关键字会使得变量a在外部文件中寻找，最后可以正常输出10 <br>
如果A文件中`static int a = 10;`，就会链接失败

TODO vscode多文件编译

### 结构体和类内的static
在类内声明一个static变量，意思是这个变量不属于某个实例，而是**属于整个类** <br>
往往用 `s_` 前缀命名，区分静态成员变量和非静态的 <br>
所有实例都共享这个变量，可以直接用`className::variables`来引用它 <br>
但是类内只是完成声明，静态成员变量需要在类外部进行定义和初始化，一般是全局部分（main等各个函数和类之外，因为static变量储存在全局/静态区）完成定义，比如`int entity::x;`

静态成员函数/类方法与之类似，因为属于整个类，可以不创建实例就用`className::func`去调用 <br>
而且这个函数不会传入this指针，只能访问静态成员变量

### 局部静态的static
静态局部变量，生存期基本上相当于整个程序的生存期，但作用域只在这个函数内 <br>
存储在全局区，所以所在函数调用后不会被摧毁

### const
变量的const <br>
const int* a或者int const* a <br>
const在*左边 表示指针指向的内容是常量 而指针本身可变

int* const a <br>
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
```cpp
class Entity {
private:
    std::string m_Name;
    int m_Age;
public:
    Entity(const std::string& name)
        : m_Name(name), m_Age(-1) {}
    
    explicit Entity(int age)    //here
        : m_Name("Unknown"), m_Age(age) {}
};

int main() {
    Entity a = "123";
    Entity b = 22; //编译器会将22自动转换为Entity，调用构造函数，但是由于explicit，这样做被禁止了
    std::cin.get();
}
```

## STL
array vector set map deque ...

STL 是 Standard Template Library（标准模板库）的缩写

常见的公共成员函数包括： <br>
size(): 返回容器中元素的数量。对于 std::array 这种固定大小的容器，size() 是一个编译时常量。 <br>
empty(): 检查容器是否为空（即 size() == 0）。这是一个更清晰、更推荐的判断容器是否为空的方式，而不是比较 size() == 0。 <br>
clear(): 删除容器中的所有元素，使其变为空。注意：std::array 因为大小固定，没有 clear() 函数。

迭代器: <br>
一种类似指针的东西，为遍历不同容器提供了通用的接口，无论容器的内存分布是连续的还是分割的（vector\链表） <br>
因为有运算符重载，使用起来就像指针，有++，解引用等操作

begin(): 返回一个指向容器中第一个元素的迭代器。 <br>
end(): 返回一个指向容器中最后一个元素**之后位置**的迭代器。 <br>
cbegin(): 返回一个 const_iterator，指向第一个元素。这意味着你不能通过这个迭代器修改它所指向的元素。 <br>
cend(): 返回一个 const_iterator，指向最后一个元素之后的位置。 <br>
（还有 rbegin(), rend(), crbegin(), crend() 用于反向迭代）

此外，STL是遵循RAII**自动管理内存**的，当容器对象销毁时（离开作用域、被 delete 等），其析构函数会自动释放所有分配的内存，不需要手动调用 free 或 delete。

### 静态数组 array
首先c++可以用c风格的数组，也就是`int arr[5]`这种

`std::array<int,3> arr1 = {1,2,3};` 适合小数据、固定大小、追求极致性能 <br>
储存在栈上，不初始化为垃圾值 <br>
`arr1`就是一个类对象，而不会转换成指针（c的数组名），因而支持直接拷贝`arr1 = arr2`（类型相同时） <br>
传参的时候不是传递指针，而是值传递对象本身。所以传递时要用引用 <br>
随时可以通过`arr1.size()`获取长度而不用单独传递长度参数（然而size是作为模板参数提供的，所以不会有额外内存消耗）

一些成员函数：

访问 <br>
at(index)：安全地访问元素。它会进行边界检查，如果索引超出范围，会抛出`std::out_of_range`异常 <br>
operator[] (index)：通过索引访问元素。不进行边界检查，行为类似 C 数组。速度快，但不安全（也就是通过arr[0]这样的c风格访问） <br>
front()：访问第一个元素 <br>
back（）：访问最后一个元素。

迭代器（begin，end，rbegin，rend ）:提供了标准的迭代器接口，可以用于范围for循环等现代 C++ 语法。 <br>
对于array这样内存布局是连续的，为了达到最优性能，它们的迭代器通常就是普通指针

data()：返回指向内部存储数据的原始指针（T*） <br>
和string同样，只能通过data来获得指针，array对象的名不会作为指向首元素的指针

### 动态数组 vector
`std::vector<int> vec = {1,2,3}` 适合大数据、大小未知的动态数组，可以很方便的扩容，频繁增删 <br>
数据储存在堆上，但是实例在栈上（指针，size等成员变量） <br>
不初始默认数据为0

初始化 <br>
vector<int> vec;    //分配了0空间 <br>
vector<int> vec(10);    //分配了10空间，并且所有数据初始化为0 <br>
注意这样做，除了分配空间，**还会调用默认构造函数创建默认的对象，相当于vector已经有了10个元素** <br>
如果只想预留空间，例如已知将来要插入 10 个元素，但想先构造后再push_back，用reserve <br>
vector<int> vec(10,2);  //分配了10空间，所有数据初始化为2

常用功能（成员函数）： <br>
at(index): 安全地访问元素，带边界检查，越界会抛出 std::out_of_range 异常。 <br>
operator[] (index): 通过索引访问元素，不进行边界检查。 <br>
front(): 访问第一个元素。 <br>
back(): 访问最后一个元素。 <br>

push_back(value): 在 vector 的末尾添加一个新元素。这是增加元素最常用的方法。 <br>
pop_back(): 删除 vector 的最后一个元素。

clear(): 删除 vector 中的所有元素，使其变为空。 <br>
insert(iterator, value): 在指定位置插入一个或多个元素。 <br>
erase(iterator) / erase(start_it, end_it): 删除指定位置或范围内的元素。 <br>
注意这里传入的是迭代器，比如删掉[2]元素，应当传入`.begin()+2`；删除[1]-[3]的元素，传入`.begin()+1, .begin()+4`，因为迭代器左闭右开，这里不会包括.begin()+4也就是[4] <br>
另外，erase之后，元素位置会自行前移

reserve(new_capacity): 预先分配至少能容纳 new_capacity 个元素的内存空间，以避免多次重新分配，提高性能 <br>
resize(new_size): 改变 vector 的大小。如果新大小更大，会用默认值填充新空间；如果更小，则会销毁多余的元素。 <br>
capacity(): 返回 vector 当前分配的存储空间能容纳多少元素。 <br>
size()：返回 vector 中已经有多少元素。 <br>
capacity()通常大于或等于 size()，以便为 push_back 等操作预留空间，减少内存重新分配的次数。

迭代器 (begin, end, rbegin, rend): 同样提供标准迭代器接口。

**使用优化** <br>
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
输出2个"Copied!" <br>
如果使用`a.emplace_back(1,2,3);`，直接传递参数给构造函数，在堆上创建对象，没有临时对象，仅涉及一次构造，不会发生拷贝

如果要新建一个临时元素放进去，用 emplace_back；如果要放入已有的元素，用 push_back

### string 字符串类
和c的有很大不同。不过`char *s`这种c风格的字符串也是支持的

1. 用+运算符来拼接，用==比较（运算符重载）
2. 用`s.length()`获取s的长度，虽然结尾依旧有'\0'，但是获取长度不依赖'\0'，所以串内可以储存的'\0'也会被完整打印（不是strlen那样低效率的遍历）
3. 用`s.substr(n,m)`来获取“从index为n开始，长度为m”的子串，如果只有n一个参数，就一直到结尾
4. 自动内存管理
5. 如果想用printf输出string，用`.cstr()`

注意，c++的string是类，没有指针算数操作，`string s`的s也**不能被视作指针**

string头文件下还有其他函数
1. `to_string(data)`函数将各类数字转换为string，注意浮点数默认有六位小数
2. `stoi()` `stod()`等各类函数可以把string转换为int/double/long/...

### 集合 set/哈希数组 unordered_set
一个元素递增且互异的数据结构，基于红黑树 <br>
初始化 <br>
set<int> s = {1,2,3,3};    //可以这样初始化，会自动排序+删除多余元素（这其实是现代c++引入的特性）

insert(data) <br>
erase(data) <br>
find(data)：查找，返回指针，会逐一遍历直到最后一个元素之后，也就是`s.find(4) != s.end()`成立说明4在s中

unordered_set基于哈希实现，插入搜索比set快，然而无法有序遍历，适合只需要插入搜索的情景

### 键值对 map/哈希表 unordered_map
存储 <Key, Value> 键值对，默认按 Key 升序排列，Key 必须唯一，重复插入会覆盖或失败 <br>
基于红黑树查找、插入、删除均为 O(log n)

初始化
```cpp
map<string,int> a = { {"Alice", 90}, {"Bob", 85} };
map<string,int> a(10); //创建10个桶

m["One"] = 1;	//Key 不存在则创建，存在则覆盖
m.insert({"Two", 2});	//若 Key 已存在，插入失败
m["One"] //这样访问，如果不存在会创建一个空键值对！
m.at("One")	//安全访问，Key 不存在抛异常

m.find("One")	//返回迭代器，未找到返回 end()
m.count("One")	//返回 1 (存在) 或 0 (不存在)

m.erase("One")	//删除指定 Key 的元素
m.clear()	//清空所有元素
```

如果不需要顺序，用unordered_map可实现O(1)的效率，基于哈希表

### 栈 stack/队列 queue
stack<int> s; <br>
栈只对栈顶的元素进行操作，后进先出 LIFO

s.push(val); 压栈 <br>
s.pop(); 出栈 <br>
s.top(); 访问栈顶，**返回栈顶元素的引用** <br>
s.empty(); 是否为空，返回bool

queue<int> q; <br>
队列中，新进入的元素在队尾，弹出的元素是队首，先进先出 FIFO

push(val) 加入队尾 <br>
pop() 移除队头元素 <br>
front()	返回队头元素的引用 <br>
back() 返回队尾元素的引用 <br>
empty()	是否为空，返回bool

### 双端队列 deque/优先级队列 priority_queue
可以在头部和尾部高效地进行插入和删除 <br>
支持通过下标 [] 或迭代器进行随机访问（类似 vector），但速度略慢于 vector <br>
由一系列连续的内存块组成（分段连续），因此扩容时不需要像 vector 那样移动所有元素

push_back(val)	尾部插入 <br>
push_front(val)	头部插入 (vector 不支持) <br>
pop_back()	尾部删除 <br>
pop_front()	头部删除 (vector 不支持) <br>
operator[] / at()	随机访问元素 <br>
begin(), end()	支持迭代器遍历

### 链表 list
TODO STL链表

## algorithm
### sort 排序
对于array和vector <br>
sort(s.begin(),s.end()) 默认递增序，第三个参数可以自定义cmp函数，但是往往用lambda表达式 <br>
如果s有3个元素，也可以写成`sort(s,s+3)`，前提是s是首元素的指针 <br>
cmp返回值表达式不能包含等号

不稳定，如果需要保持相等元素的原始相对顺序，用 std::stable_sort

### find_if
sort(s.begin(),s.end(),[](){}) <br>
最后一个参数叫谓词 一个可调用对象（函数指针、函数对象 functor、或 Lambda 表达式）。它接受一个元素作为参数，如果该元素满足条件则返回 true，否则返回 false <br>
用于在指定的迭代器范围内查找第一个满足特定条件的元素 <br>
返回迭代器类型