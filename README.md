## **无指针class设计要点**
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
// r2实际上绑定了一个临时变量.
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

1. **初始化 > 赋值**;

2. 函数**重载**与**默认值**不应产生**二义性**(编译器会不知该如何选择);

3. ctor也可以放在private区，这是一种编程模式(Singleton);

    * 用于只在外界创建**一个对象**的情景;

### Inline function(内联函数)

1. 尽量写成内联函数，**提高效率**;

2. 尽管对于**复杂的函数体**，效率**也许没有提升**;

3. 但是**让编译器来做决定(是否使用真正的内联机制)**永远不会是坏事情;

### Pass by value  vs.  Pass by reference(to const)

1. 尽量所有的参数都以**传递引用或指针(效率高)**;

2. 若考虑细节, **4个字节以下，传值更快**(一般可不用考虑);

3. 若不希望值被更改，在前面增加`const`;

### Return by reference

1. 函数返回的是**内部新创建的local变量**时, 不应返回reference;

    * 例如用非成员函数定义加法时, 不能返回reference;
    
2. 传递者**无需知道**接受者是以reference形式接收, 

    * 若返回**指针**, 则接收端也**必须**为**指针**;
    
    * 若返回**引用**, 则接收端可以是**引用/指针内容/变量本身**;

### Friend

1. friend function(友元函数)

    * 直接使用**private成员**, 效率高;
    
    * 不宜过多，**破坏封装性**(C++的特性与优点);
    
    * 友元的声明仅仅指定了**访问权限**;
    
    * 友元声明与类应放在**同一个头文件中**(类的外部);
    
    * 类内仅通过`friend`声明**权限**, 此外还需在类外提供**独立的声明**;
    
2. friend(友元)

    * 相同class的各个objects自动**互为friends(友元)**;
    
    * 看似打破封装，实则成立合理；

-----------

## **有指针class设计要点**
>* **Big Three: 拷贝构造函数, 拷贝赋值函数, 析构函数;**
>* **对于含指针类(Class with pointer), 必须含有copy ctor & copy op;**
>* **static 和 Singleton;**

### Copy Ctor (拷贝构造函数)

1. **要点**

   * 新构造的对象: new 新地址;
   
   * 深度拷贝 = 拷贝值 + 拷贝地址;

2. **代码示例**

```cpp
// Declaration
String(const String& str);

// Definition
inline String::String(const String& str)
{
	m_data = new char[strlen(str.m_data) + 1];
	strcpy(m_data, str.m_data);
}
```

### Copy Operator (拷贝赋值函数)
        
1. **要点**

	* 判断是否自我赋值(self assignment);

		* if True: return self;

	* 被赋值对象: delete 原地址;

	* 新构造的对象: new 新地址;

	* 深度拷贝;
	
2. **代码示例**

```cpp
// Declaration
String& operator=(const String& str);

// Definition
inline String& String::operator=(const String& str)
{
	if (this == &str)	// Self Assignment
		return *this;
	delete[] m_data;
	m_data = new char[strlen(str.m_data) + 1];
	strcpy(m_data, str.m_data);
	return *this;
}
```

### Stack & Heap(栈 & 堆)

1. Stack: 

	* scope(作用域)内的一块**内存空间**, **function body**内声明的变量(static除外);

	* 但是static object的内存在scope结束之后**仍然存在**,直至整个程序结束才消亡;

2. Heap: 

	* 由操作系统提供的一块**global**内存空间, 可**动态分配内存(dynamic allocate)**;

	* global object在**整个程序结束之后**才结束生命周期;
		
### New的内部工作原理
 
* **先分配 memory, 再调用 ctor**

* ep: 
```cpp
Complex * pc = new Complex(1, 2);
```

* 编译器转化为:

	1. **分配内存**, 内部调用 `malloc(n)`
	
		```cpp
		void * mem = operator new( sizeof(Complex) );
		```

	2. **转换类型**, 将 mem指针 从 `void*` 转型成 `Complex*`
	
		```cpp
		pc = static_cast(mem);
		```

	3. **重新指向**, pc 指向新创建对象的头部
	
		```cpp
		pc -> Complex::Complex(1, 2);
		```
		
### Delete的内部工作原理
                
* 先调用 dtor, 再释放 memory

* ep:
```cpp
String * ps = new String("Hello");
...
delete ps;
```

* 编译器转化为:

	1. **析构内容**, 删除**动态分配的内存**(指针指向的内容)
	
	```cpp
	String::~String(ps);
    ```
	
	2. **删除指针**, 内部调用 **`free(ps)`**, 删除指针
	
	```cpp
	operator delete(ps);
	```
	
### Static

1. **无static**

	* **成员变量**: 根据创建出对象的不同，可以有**多份地址**;

	* **成员函数**: **地址只有一份**，通过this指针来调用不同的对象;
	
2. **含static**

	* **static成员变量**

		* 地址**只有一份**;

		* 需在**类外初始化**;
		
		```cpp 
		Class_Name::static_data = value;
		```

		* 不能使用参数初始化表;
		
	* **static成员函数**

		* 地址只有一份, **没有this指针**;
		
		* 不能访问类中的**非静态变量**, 只能处理**static数据**;

		* 可用类或对象来调用;

### Singleton - 将Ctor放在Private区的设计模式

* 代码示例

```cpp
class A
{
public:
	static A& getInstance();   
	setup() {...}
private:
	A();
	A(const A& rhs);
	// ...        
};

// 只有调用该静态方法，才会创建static变量，且只能创建一份
A& A::getInstance()
{
static A a;
return a;
}
```

### Class Template 与 Function Template

* 类模板实例化: 需要**指定类型**;

* 函数模板实例化: 由编译器进行**自动类型推断(argument deduction)**;

* 代码示例

	1. 函数模板

	```cpp
	template <typename T>
	int compare(const T& v1, const T& v2)
	{
		if (v1 < v2) return -1;
		if (V1 > V2) return 1;
		return 0;
	}
	
	// 实例化输出int compare(const int&, const int&)
	cout << compare(1, 0) << endl;	// T is int
	```
	
	2. 类模板

	```cpp
	template <typename T> class A
	{
		// ...
	}
	
	// 实例化类模板
	A<int> a;
	```
	
### Namespace
        
* 不同文件为相同namespace编写的内容会被自动结合到一起;

* 两种使用方式:

	* `using namespace std;`
	* `using std::cout;`
	
### 内存管理

<div align=center><img width="500" height="300" src="https://github.com/ZhouJialiang48/Notes_cpp/raw/master/mem_manage_1.jpg"/></div>
<div align=center><img width="500" height="300" src="https://github.com/ZhouJialiang48/Notes_cpp/raw/master/mem_manage_2.jpg"/></div>	
<div align=center><img width="500" height="300" src="https://github.com/ZhouJialiang48/Notes_cpp/raw/master/mem_manage_3.jpg"/></div>

-----------	

## OOP思想 -- 类与类之间的关系:

1. Inheritance 继承
2. Composition 复合
3. Delegation 委托
	
### Composition 复合(**has-a**)

1. **Adapter**设计模式
	
2. A类(Container)复合B类(Component)，并能够使用B类的一些方法
	
3. 构造与析构
	
	* **构造由内而外**，首先调用Component的默认构造函数，才执行自己
	```cpp
	Container::Container(...): Component() {...}
	```
	* **析构由外而内**，先执行自己，再调用Component的析构函数
	```cpp
	Container::~Container(...): ~Component() {...}
	```
4. 代码示例:
```cpp
template <typename T>
class queue
{
	// ...
protected:
	deque<T> c;	// 底层容器
public:
	// 下列方法完全通过底层容器c的操作函数来完成
	bool empty() const { return c.empty; }
	size_type size() const { return c.size; }
	// ...
}
```

### Delegation委托(Composition by Reference，用指针复合)

1. **pImpl**(Pointer to Implementation)

	* 通过**指针**来执行操作, **编译防火墙**
	
2. 可以看做 **"虚"一些的复合**

3. **Handle**与**Body**寿命不同步

4. 代码示例:
```cpp
class StringRep;
class String
{
public:
	String();
	String(const char* s);
	String(const String& str);
	String& operator=(const String& str);
	~String();
// ...
private:
	StringRep* rep; // pimpl
}
```

### Inheritance 继承(**is-a**)

* 语法(**class derivation list, 类派生列表**)
```cpp
class sub_A : public/private/protected A
```
1. 子类(Derived,派生类)的对象**包含**父类(Base,基类)的成分

2. 构造与析构
	
	2.1 构造**由内而外**，首先调用Base的默认构造函数，才执行自己
	```cpp
	Derived::Derived(...): Base() {...};
	```
	2.2 析构**由外而内**，先执行自己，再调用Base的析构函数
	```cpp	
	Derived::~Derived(...): ~Base() {...};
	```
	2.3 父类的dtor**一定是virtual**，否则出现**undefined behavior**
	
3. 代码示例:
```cpp
struct _List_node_base
{
	_List_node_base* _M_next;
	_List_node_base* _M_brev;
};

template<typename T>
struct _List_node_
	: public _List_node_base
{
	_Tp _M_data;
}
```

## Virtual Function(虚函数)

1. **non-virtual function:** 不希望被**override(复写)**

2. **virtual function:** 希望被**override(复写)**, 且本身**有默认定义**

3. **pure virtual function:** 必须被**override(复写)**, 且本身**没有默认定义**

4. 代码示例:
```cpp
class Shape
{
public:
	virtual void draw() const = 0;	// pure virtual
	virtual void error(const std::string& msg);	// impure virtual
	int objectID() const;	// non-virtual
	// ...
}

class Rectangle
	: public Shape 
{
public:
	virtual void draw() const { ... }
};

int main()
{
	Rectangle r;
	// ...
	r.draw()
}
```
5. Template Method (设计模式)

	* Application framework + Application
	
### 种关系下(继承 + 复合)的构造顺序

1. Derived -> Base -> Component

	* 子类派生于父类, 父类又含有复合类;
	
	* 构造顺序: Component, Base, Derived
	
2. Base <- Derived -> Component

	* 子类派生于父类, 且子类又含有复合类;
	
	* 构造顺序: 待验证
