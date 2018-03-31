### 无指针class设计要点
>* **public：接口(interface)，public：数据细节;**
>* **首先考虑pass by reference(to const);**
>* **类内尽可能添加const;**
>* **ctor尽量使用初始值列表;**
>* **return by reference/value;**
>* **类声明中的内联函数定义，需放在同一个头文件当中(类的后面)，否则会产生链接错误**

### Reference to const 
1. **引用**
    * 一种 **别名** ,  一种**绑定关系**;
    * 不是一个`object`, 因此不存在`refence to reference`(**differ from pointers**);
    * 必须**初始化**;
    * 对常量的**引用**可简称为 *常量引用* (不规范);

2. **常量引用**
    * 注意区别:
```cpp
double pi = 3.14;
int &r1 = pi;       // 非法！
const int &r2 = pi;	// 合法！

// 实际上编译器执行如下：
const int temp_pi = pi;
const int &r2 = temp_pi;
// r2实际上 绑定 了一个 临时变量 .
```

### const pointer  &  pointer to const

1. **常量指针(const pointer, cp)**

    * 本身是个常量(其指向的**地址**不能改变);
    * 必须初始化;

2. **指向常量的指针(pointer to const, pc)**

    * 若指向常量：
        * 必须用pc，且**不能通过pc改变**该常量对象;
    * 若指向非常量：
        * 也可以用pc，**只是无法通过pc改变**非常量值;


### This指针
1. **功能:**
    * 是一种用来**访问**调用该成员函数的**对象的地址**的**隐式形参;**
2. **初始化:**
    * **即**用该成员函数的**对象的地址**来**初始化**this指针.
3. **何时使用:**
    * 任何**对类成员的直接访问**都被看作是**对this的隐式引用**；
4. **this的const属性:**
    * this是一个**常量指针(const pointer but not pointer to const)**,
        * 所指向的地址不允许被改变;
    * 默认情况下, this的类型为`Class_name const*`,
        * 即**不能**将 this指针**指向一个常量对象**;
    * 若额外将this声明为`const Class_name const*`,
        * 则**可以**将this指针**指向一个常量对象**;
        * 此时 应该将const放在**成员函数参数列表** 之后;
        * 使用形式举例: `void get_name() const {}`
        * 这种方式**声明**的函数被称作**常量成员函数**;

### Constructor(ctor, 构造函数)
1. **初始化 > 赋值**；

2. 函数**重载**与**默认值**不应产生**二义性**(编译器会不知该如何选择)；

3. ctor也可以放在private区，这是一种编程模式(Singleton)：
    * 用于只在外界创建**一个对象**的情景；

### Inline function(内联函数)
1. 尽量写成内联函数，**提高效率**；

2. 尽管对于**复杂的函数体**，效率**也许没有提升**；

3. 但是**让编译器来做决定(是否使用真正的内联机制)**永远不会是坏事情；

### Pass by value  vs.  Pass by reference(to const)
1. 尽量所有的参数都以**传递引用或指针(效率高)**；

2. 若考虑细节，**4个字节以下，传值更快**(一般可不用考虑)；

3. 若不希望值被更改，在前面增加`const`；

### Return by reference
1. 函数返回的是**内部新创建的local变量**时, 不应返回reference
    * 例如用非成员函数定义加法时, 不能返回reference；
2. 传递者**无需知道**接受者是以reference形式接收,
    * 若返回**指针**, 则接收端也**必须**为**指针**；
    * 若返回**引用**, 则接收端可以是**引用/指针内容/变量本身**；

### Friend
1. friend function(友元函数)
    * 直接使用**private成员**, 效率高;
    * 不宜过多，**破坏封装性**(C++的特性与优点);
    * 友元的声明仅仅指定了**访问权限**:
    * 友元声明与类应放在**同一个头文件中**(类的外部);
    * 类内仅通过`friend`声明**权限**, 此外还需在类外提供**独立的声明**
2. friend(友元)
    * 相同class的各个objects自动**互为friends(友元)**：
    * 看似打破封装，实则成立合理；


