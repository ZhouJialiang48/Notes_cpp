# C++ 面向对象 Boolan

## 无指针class

* **public：接口(interface)，public：数据细节;**

* **首先考虑pass by reference(to const);**

* **类内尽可能添加const;**

* **ctor尽量使用初始值列表;**

* **return by reference/value;**

* **类声明中的内联函数定义，需放在同一个头文件当中(类的后面)，否则会产生链接错误**

### Reference to const 

* **引用**是一种 **别名** ，代表一种  **绑定关系**，它不是一个`object`，因此不存在`refence to reference`(**differ from pointers**).

* **引用**必须有**初始化**；

* 对常量的 __*引用*__ 可简称为 __*常量引用*__ (不规范)；

* 注意区别：
```cpp
double pi = 3.14;

int &r1 = pi;       // 非法！

const int &r2 = pi;	// 合法！
 ```

实际上编译器执行如下：
 ```cpp

const int temp_pi = pi;

const int &r2 = temp_pi;
```

即 r2实际上**绑定**了一个**临时变量**.
