## 1.  什么是回调函数？

回调函数可以被简单地理解为：A函数作为参数传递给B函数，然后B函数在执行的某一时刻调用A函数。这里的A函数就是回调函数。

回调处理网络请求示例:

```cpp
#include <iostream>
#include <thread>
#include <functional>
#include <chrono>
 
// 模拟网络请求函数，接受一个回调函数
void asyncNetworkRequest(std::function<void(const std::string&)> callback) {
    std::this_thread::sleep_for(std::chrono::seconds(2)); // 模拟网络延迟
    std::string response = "Server response data";
    callback(response); // 调用回调函数并传递响应数据
}
 
// 回调函数
void onRequestCompleted(const std::string& response) {
    std::cout << "Network request completed with response: " << response << std::endl;
}
 
int main() {
    std::cout << "Sending network request..." << std::endl;
    
    // 异步发送网络请求，并传递回调函数
    std::thread requestThread(asyncNetworkRequest, onRequestCompleted);
    
    std::cout << "Doing other work while waiting for network response..." << std::endl;
    
    // 主线程可以继续做其他工作
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Still doing other work..." << std::endl;
    
    // 等待网络请求完成
    requestThread.join(); // 等待线程完成
    
    return 0;
}
```



## 2.  C++的多态？

**多态的概念:**通俗来讲，多态就是多种形态，具体来讲，就是去完成某个行为，当不同的对象去完成时会产生不同的状态。当基类指针指向子类对象时候，虚函数能实现运行时多态（多态指：同一个接口的不同实现方式）

**多态的分类:** 多态可以分为**编译时的多态**和**运行时的多态**。前者主要是指 **函数的重载**（包括运算符的重载）、对重载函数的调用，在编译时就能根据实参确定应该调用哪个函数，因此叫编译时的多态；而后者则和继承、虚函数等概念有关。

**多态的构成条件:**
1. 必须通过基类的指针或者引用调用虚函数。
2. 被调用的函数必须是虚函数，且派生类必须对基类的虚函数进行重写。
3. 必须存在继承关系

## 3.  C++之函数重载和函数重写
**函数重载：** 重载声明是指一个与之前已经在该作用域内声明过的函数或方法具有相同名称的声明，但是它们的参数列表和定义（实现）不相同。（在同一个作用域内，可以声明几个功能类似的同名函数，但是这些同名函数的形式参数（指参数的个数、类型或者顺序）必须不同。不能仅通过返回类型的不同来重载函数。）
**函数重载好处：** 函数重载通常用来命名一组功能相似的函数，这样做减少了函数名的数量，避免了名字空间的污染，对于程序的可读性有很大的好处。
**函数重载特征是：**
 （1）相同的范围（在同一个作用域中）；
 （2）函数名字相同；
 （3）参数不同；
 （4）返回值可以不同；

**函数重写（也称为覆盖 override）：** 函数重写是指子类重新定义基类的虚函数。

**特征是：**
 （1）不在同一个作用域（分别位于派生类与基类）；
 （2）函数名字相同；
 （3）参数相同；
 （4）基类函数必须有 virtual 关键字，不能有 static 。
 （5）返回值相同，否则报错；
 （6）重写函数的访问修饰符可以不同；

## 4.  虚函数

**虚函数：** 即被virtual修饰的类成员函数称为虚函数。
一旦定义了虚函数，该基类的派生类中同名函数也自动成为了虚函数。也就是说在派生类中有一个和基类同名的函数，只要基类加了virtual修饰，派生类不加virtual修饰也是虚函数。
这里有几点需要注意：

- **静态成员函数不能是虚函数**。这是因为静态成员函数并不依赖于对象实例，它们没有 "this" 指针，但是虚函数在实现多态时通过 "this" 指针访问虚表来找到正确的函数版本。因此，静态成员函数不能被声明为虚函数。

- **普通函数（也就是非成员函数）也不能是虚函数**。因为普通函数（全局函数或静态函数）不属于任何类，自然就没有继承的概念，因此也就不可能有虚函数。

- **构造函数不能声明为虚函数**。虚函数是通过虚函数表来实现的，每个对象都有一个虚函数表指针，指向该类的虚函数表。而**对象在构造函数中才初始化虚函数表指针**，如果构造函数是虚函数，就需要通过虚函数表来调用，但此时对象还没有实例化，也就没有虚函数表指针，无法找到虚函数表，因此构造函数不能是虚函数。



**虚函数的重写( 覆盖 )：** 派生类中有一个跟基类完全相同的虚函数(即派生类虚函数与基类虚函数的返回值类型、函数名字、参数列表完全相同)，称子类的虚函数重写了基类的虚函数。  

**虚析构函数：** 虚析构函数的作用是为了**避免内存泄露**，而且是当子类中会有指针 成员变量 时才会使用得到的。也就说虚析构函数使得在删除**指向子类对象的基类指针**时可以调用子类的析构函数达到释放子类中堆内存的目的，而防止内存泄露的。

当父类的析构函数不声明成虚析构函数的时候，当子类继承父类，父类的指针指向子类时，delete掉父类的指针，只调动父类的析构函数，而不调动子类的析构函数。

当父类的析构函数声明成虚析构函数的时候，当子类继承父类，父类的指针指向子类时，delete掉父类的指针，先调动子类的析构函数，再调动父类的析构函数。

**纯虚函数：** 基类中不对虚函数给出有意义的实现，它只是在派生类中有具体的意义。这时基类中的虚函数只是一个入口，具体的目的地由不同的派生类中的对象决定。这个虚函数称为纯虚函数。

1、在定义纯虚函数时，不能定义虚函数的实现部分。
2、把函数名赋值为0，本质上是将指向函数体的指针值赋为初值0。与定义空函数不一样，空函数的函数体为空，即调用该函数时，不执行任何动作。没有在派生类重新定义这种虚函数之前，是不能调用这种纯虚函数的。
3、把至少包含一个纯虚函数的类，称为**抽象类**。这种类**只能作为派生类的基类**，不能用来创建对象。
其理由是明显的：纯虚函数在类的vftable表中对应的表项被赋值为0。也就是指向一个不存在的函数。由于编译器绝对不允许有调用一个不存在的函数的可能，所以该类不能生成对象。在它的派生类中，除非重写此函数，否则也不能生成对象。
4、在以抽象类作为基类的派生类中必须有纯虚函数的实现部分，即必须有重写纯虚函数的函数体。否则，这样的派生类也是不能产生对象的。 综上所述，可把纯虚函数归结为：**抽象类的唯一用途是为派生类提供基类，纯虚函数的作用是作为派生类中的成员函数的基础，并实现动态多态性**。

**抽象类：**C++接口时通过抽象类实现的,设计抽象类的目的,是为了给其他类提供一个可以继承的适当的基类.抽象类本类不能被用于实例化对象,只能作为接口使用。在C++中,我们把**只能用于被继承而不能直接创建对象的类**称之为**抽象类**,这种基类不能直接生成对象,而只有被继承后,并重写其虚函数后,才能使用。当抽象类的派生类实现了继承而来的纯虚函数后,才能实例化对象。

```cpp
class VirtualClass{
	public:
		virtual void fun1() = 0;//纯虚函数

		virtual ~VirtualClass();
};



class ClassA : public VirtualClass{
	public:
		void fun1(){
			printf("The is ClassA fun1\n");
		};

		virtual ~ClassA();
		
};


VirtualClass:: ~VirtualClass(){
}

ClassA:: ~ClassA(){
}

int main(){
	//编译报错，这个非法的。抽象类是不可以被实例化的，必须被继承，才能够被实例化
	VirtualClass * virtualClass = new VirtualClass();//error: cannot allocate an object of abstract type 'VirtualClass'
	
	VirtualClass * classA = new ClassA();
	classA->fun1();
	return 0;
}

```



## 4.  虚函数表

**什么是虚函数表：**

- 虚函数表是一个指针数组，其元素是虚函数的指针，每个元素对应一个虚函数的函数指针。每一个有虚函数的类（或者从有虚函数的类继承而来的类）都有一个相关联的虚函数表。需要指出的是，普通的函数即非虚函数，其调用并不需要经过虚表，所以虚表的元素并不包括普通函数的函数指针。
- 每个有虚函数的对象都包含一个指向其类的虚函数表的指针。这个指针通常被称为 vptr。
-   当调用一个对象的虚函数时，编译器使用对象的 vptr 来定位类的虚函数表。接着，从虚函数表中找到相应的虚函数指针并调用该函数。这个过程是在运行时进行的，因此可以实现多态行为。
- 虚表内的条目，即虚函数指针的赋值发生在编译器的编译阶段，也就是说在代码的编译阶段，虚表就可以构造出来了。
- 当一个类继承自另一个有虚函数的类，并且没有重写任何虚函数，该类的对象将使用父类的虚函数表。
- 当派生类重写了基类的虚函数，派生类的虚函数表中该函数的入口会被更新为指向派生类版本的函数。
- 如果派生类添加了新的虚函数，它们会被添加到虚函数表的末尾。
- 虚函数表是编译器生成的，程序运行时被载入内存。一个类的虚函数表中列出了该类的全部虚函数地址。

```cpp
#include <iostream>
using namespace std;
class A
{
public:
    int i;
    virtual void func() {}
    virtual void func2() {}
};
class B : public A
{
    int j;
    void func() {}
};
int main()
{
    cout << sizeof(A) << ", " << sizeof(B);  //输出 8,12
    return 0;
}
```

在 32 位编译模式下，程序的运行结果是：
8, 12

如果将程序中的 virtual 关键字去掉，输出结果变为：
4, 8

对比发现，有了虚函数以后，对象所占用的存储空间比没有虚函数时多了 4 个字节。实际上，任何有虚函数的类及其派生类的对象都包含这多出来的 4 个字节，这 4 个字节就是实现多态的关键——它位于对象存储空间的最前端，其中存放的是虚函数表的地址。

每一个有虚函数的类（或有虚函数的类的派生类）都有一个虚函数表，该类的任何对象中都放着该虚函数表的指针（可以认为这是由编译器自动添加到构造函数中的指令完成的）。

虚函数表是编译器生成的，程序运行时被载入内存。一个类的虚函数表中列出了该类的全部虚函数地址。例如，在上面的程序中，类 A 对象的存储空间以及虚函数表（假定类 A 还有其他虚函数）如图所示:

<div align=center><img src=".\fig\虚函数表1.webp"height="565"/> </div>

类 B 对象的存储空间以及虚函数表（假定类 B 还有其他虚函数）如图所示：

<div align=center><img src=".\fig\虚函数表2.webp"height="565"/> </div>

假设 pa 的类型是 A*，则 pa->func() 这条语句的执行过程如下：

1. 取出 pa 指针所指位置的前 4 个字节，即对象所属的类的虚函数表的地址（在 64 位编译模式下，由于指针占 8 个字节，所以要取出 8 个字节）。如果 pa 指向的是类 A 的对象，则这个地址就是类 A 的虚函数表的地址；如果 pa 指向的是类 B 的对象，则这个地址就是类 B 的虚函数表的地址。

2. 根据虚函数表的地址找到虚函数表，在其中查找要调用的虚函数的地址。不妨认为虚函数表是以函数名作为索引来查找的，虽然还有更高效的查找方法。如果 pa 指向的是类 A 的对象，自然就会在类 A 的虚函数表中查出 A::func 的地址；如果 pa 指向的是类 B 的对象，就会在类 B 的虚函数表中查出 B::func 的地址。类 B 没有自己的 func2 函数，因此在类 B 的虚函数表中保存的是 A::func2 的地址，这样，即便 pa 指向类 B 的对象，`pa->func2();`这条语句在执行过程中也能在类 B 的虚函数表中找到 A::func2 的地址。

3. 根据找到的虚函数的地址调用虚函数。

## 5. 智能指针(C++11)

C++11 中引入了智能指针（Smart Pointer），它利用了一种叫做 RAII（资源获取即初始化）的技术将普通的指针封装为一个**栈对象**。当栈对象的**生存周期**结束后，会在析构函数中释放掉申请的内存，从而防止内存泄漏。这使得**智能指针实质是一个对象**，行为表现的却像一个指针。

### 5.1. unique_ptr

1. 基于排他所有权模式：两个指针不能指向同一个资源
2. 不能被复制，但可以被移动，这意味着所有权可以转移，但不能共享。无法进行左值unique_ptr复制构造，也无法进行左值复制赋值操作，但允许临时右值赋值构造和赋值
3. 保证指向某个对象的指针，当它本身离开作用域时会自动释放它指向的对象。
4. 在容器中保证指针是安全的无法进行左值复制赋值操作，但允许临时右值赋值构造和赋值

#### 5.1.1 使用示例
**无法进行左值复制赋值操作，但允许临时右值赋值构造和赋值:**

```cpp
unique_ptr<string> p1(new string("I'm Li Ming!"));
unique_ptr<string> p2(new string("I'm age 22."));
	
cout << "p1：" << p1.get() << endl;
cout << "p2：" << p2.get() << endl;

p1 = p2;					// 禁止左值赋值
unique_ptr<string> p3(p2);	// 禁止左值赋值构造

unique_ptr<string> p3(std::move(p1));
p1 = std::move(p2);	// 使用move把左值转成右值就可以赋值了，效果和auto_ptr赋值一样

cout << "p1 = p2 赋值后：" << endl;
cout << "p1：" << p1.get() << endl;
cout << "p2：" << p2.get() << endl;
```

**在 STL 容器中使用unique_ptr，不允许直接赋值:**

```cpp
vector<unique_ptr<string>> vec;
unique_ptr<string> p3(new string("I'm P3"));
unique_ptr<string> p4(new string("I'm P4"));

vec.push_back(std::move(p3));
vec.push_back(std::move(p4));

cout << "vec.at(0)：" << *vec.at(0) << endl;
cout << "vec[1]：" << *vec[1] << endl;

vec[0] = vec[1];	/* 不允许直接赋值 */
vec[0] = std::move(vec[1]);		// 需要使用move修饰，使得程序员知道后果

cout << "vec.at(0)：" << *vec.at(0) << endl;
cout << "vec[1]：" << *vec[1] << endl;

```
#### 5.1.2 手写unique_ptr

```cpp
#include <iostream>

template<typename T>
class unique_ptr {
public:
    unique_ptr(T* ptr = nullptr) : ptr(ptr) {} //这是类的构造函数，接受一个可选的指针参数ptr，默认值为nullptr。

    ~unique_ptr() {
        delete ptr; //析构函数
    }

    unique_ptr(unique_ptr&& other) : ptr(other.ptr) {
        other.ptr = nullptr; //这是移动构造函数，接受一个右值引用的unique_ptr对象other。它将other的内部指针赋值给当前对象的指针ptr，然后将other的指针置为nullptr，实现资源的转移所有权。
    }

    unique_ptr& operator=(unique_ptr&& other) {
        if (this!= &other) {
            delete ptr;
            ptr = other.ptr;
            other.ptr = nullptr;
        }
        return *this; //这是移动赋值运算符重载函数。首先检查当前对象是否与传入的对象不是同一个对象。如果不是，先释放当前对象管理的资源，然后将传入对象的指针赋值给当前对象的指针，并将传入对象的指针置为nullptr，实现资源的转移所有权，最后返回当前对象的引用（考虑到多个=的情况）。
    }

    T& operator*() const {
        return *ptr; // 这是重载的解引用运算符*。当通过unique_ptr对象使用解引用运算符时，返回内部管理的指针所指向的对象的引用，使得可以像使用原始指针一样访问所指向的对象。
    }

    T* operator->() const {
        return ptr; // 这是重载的箭头运算符->。当通过unique_ptr对象使用箭头运算符时，直接返回内部管理的指针，使得可以像使用原始指针一样访问所指向对象的成员。
    }
    //释放unique_ptr对资源的所有权，并返回内部管理的指针。它先将内部指针保存到临时变量temp中，然后将内部指针置为nullptr，表示不再管理该资源，最后返回临时变量。
    T* release() {
        T* temp = ptr;
        ptr = nullptr; 
        return temp;
    }

    T* get() const {
        return ptr; //返回内部管理的指针，用于获取当前unique_ptr对象管理的资源的指针。
    }
//在类的私有部分，定义了一个成员变量ptr用于存储管理的资源指针。同时，将复制构造函数和复制赋值运算符显式地删除，以确保unique_ptr不能进行复制操作，只能进行移动操作，保证资源的独占性。
private:
    T* ptr;
    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;
};

int main() {
    unique_ptr<int> up1(new int(42));
    std::cout << *up1 << std::endl;

    unique_ptr<int> up2 = std::move(up1);
    if (up1.get() == nullptr) {
        std::cout << "up1 is now empty." << std::endl;
    }
    std::cout << *up2 << std::endl;

    int* raw_ptr = up2.release();
    std::cout << "Released pointer value: " << *raw_ptr << std::endl;

    delete raw_ptr;

    return 0;
}
```



### 5.2 shared_ptr

C++智能指针shared_ptr是一种可以自动管理内存的智能指针，它是C++11新增的特性之一。与传统指针不同，shared_ptr可以自动释放所管理的动态分配对象的内存，并避免了手动释放内存的繁琐操作，从而减少了内存泄漏和野指针的出现。
shared_ptr是一个模板类，通过引用计数器实现多个智能指针共享对一个对象的所有权。每次复制一个shared_ptr对象时，该对象的引用计数器会增加1，当一个shared_ptr对象被销毁时，引用计数器减1，如果引用计数器变为0，则释放所管理的对象的内存。
#### 5.2.1 使用示例
```cpp
// 构造
shared_ptr<int> up1(new int(10));// int(10) 的引用计数为1
shared_ptr<int> up2(up1);// 使用智能指针up1构造up2, 此时int(10) 引用计数为2
// 赋值
shared_ptrr<int> up1(new int(10));  // int(10) 的引用计数为1
shared_ptr<int> up2(new int(11));   // int(11) 的引用计数为1
up1 = up2;	// int(10) 的引用计数减1,计数归零内存释放，up2共享int(11)给up1, int(11)的引用计数为2
// 主动释放
shared_ptrr<int> up1(new int(10));
up1 = nullptr ;	// int(10) 的引用计数减1,计数归零内存释放 
// 或
up1 = NULL; // 作用同上 
//  获取引用计数
up1.use_count(); // 返回int(10)的引用计数
// 重置
p.reset(); // 将p重置为空指针，所管理对象引用计数 减1
// 交换
p1.swap(p2); // 交换p1 和p2 管理的对象，原对象的引用计数不变
```

#### 5.2.2 手写shared_ptr
```cpp
#include <iostream>

template<typename T>
class shared_pointer {
public:
//这是类的构造函数，接受一个可选的指针参数ptr，默认值为nullptr。在构造对象时，将传入的指针赋值给内部成员变量ptr，并创建一个新的整数指针ref_count，初始值为 1，表示当前只有一个shared_pointer对象管理这个资源。
    shared_pointer(T* ptr = nullptr) : ptr(ptr), ref_count(new int(1)) {}
//这是复制构造函数，接受一个常量引用的shared_pointer对象other。复制构造函数将other的指针和引用计数指针分别赋值给当前对象的指针和引用计数指针，然后增加引用计数，表示现在有多个shared_pointer对象共享同一个资源。
    shared_pointer(const shared_pointer& other) : ptr(other.ptr), ref_count(other.ref_count) {
        ++(*ref_count);
    }
//这是类的析构函数。当shared_pointer对象被销毁时，析构函数会被自动调用。首先减少引用计数，如果引用计数变为 0，表示没有其他shared_pointer对象在使用该资源，此时释放资源（即删除指针ptr所指向的内存和引用计数指针ref_count）。
    ~shared_pointer() {
        if (--(*ref_count) == 0) {
            delete ptr;
            delete ref_count;
        }
    }
//这是赋值运算符重载函数。首先检查当前对象是否与传入的对象不是同一个对象。如果不是，先减少当前对象的引用计数，如果引用计数变为 0，则释放资源。然后将传入对象的指针和引用计数指针赋值给当前对象，并增加引用计数。最后返回当前对象的引用。
    shared_pointer& operator=(const shared_pointer& other) {
        if (this!= &other) {
            if (--(*ref_count) == 0) {
                delete ptr;
                delete ref_count;
            }
            ptr = other.ptr;
            ref_count = other.ref_count;
            ++(*ref_count);
        }
        return *this;
    }
//这个函数用于获取当前资源的引用计数。如果引用计数指针不为nullptr，则返回引用计数的值，否则返回 0。
    int use_count() const {
        return ref_count ? *ref_count : 0;
    }
//这是重载的解引用运算符*。当通过shared_pointer对象使用解引用运算符时，返回内部管理的指针所指向的对象的引用，使得可以像使用原始指针一样访问所指向的对象。
    T& operator*() const {
        return *ptr;
    }
//这是重载的箭头运算符->。当通过shared_pointer对象使用箭头运算符时，直接返回内部管理的指针，使得可以像使用原始指针一样访问所指向对象的成员。
    T* operator->() const {
        return ptr;
    }
//在类的私有部分，定义了一个成员变量ptr用于存储管理的资源指针，以及一个整数指针ref_count用于存储资源的引用计数。
private:
    T* ptr;
    int* ref_count;
};

int main() {
    shared_pointer<int> p1(new int(10));
    shared_pointer<int> p2 = p1;
    shared_pointer<int> p3 = p1;
    std::cout << p1.use_count() << std::endl;
    std::cout << p2.use_count() << std::endl;
    std::cout << p3.use_count() << std::endl;
    return 0;
}
```
#### 5.2.3 循环引用问题
循环引用通常发生在两个对象互相持有对方的shared_ptr引用时。例如，如果类A中包含指向类B的shared_ptr，而类B中也包含指向类A的shared_ptr，那么这两个对象就会相互保持引用，导致它们的引用计数无法减少到零，从而无法触发析构函数释放资源。
为了解决循环引用问题，可以使用weak_ptr，这是一种不增加引用计数的智能指针。weak_ptr允许你引用一个由shared_ptr管理的对象，但不会对其引用计数产生影响。当shared_ptr的引用计数降到零时，即使还有weak_ptr指向该对象，对象也会被销毁。
```cpp
#include <memory>
#include <iostream>

class B; // 前向声明

class A {
public:
    std::shared_ptr<B> m_b;
    ~A() {
        std::cout << "~A()" << std::endl;
    }
};

class B {
public:
    std::weak_ptr<A> m_a; // 使用weak_ptr代替shared_ptr
    ~B() {
        std::cout << "~B()" << std::endl;
    }
};

int main() {
    std::shared_ptr<A> a(new A);
    std::shared_ptr<B> b(new B);
    a->m_b = b;
    b->m_a = a;
    // 当main函数结束时，a和b的引用计数都会降到零，触发析构函数
}
```


### 3. weak_ptr
std::weak_ptr 是一种智能指针，通常不单独使用，只能和 shared_ptr 类型指针搭配使用，可以视为 shared_ptr 指针的一种辅助工具。

weak_ptr可以从一个shared_ptr或者另一个weak_ptr对象构造，获得资源的观测权。但weak_ptr没有共享资源，它的构造不会引起指针引用计数的增加。使用weak_ptr的成员函数use_count()可以观测资源的引用计数，另一个成员函数expired()的功能等价于use_count()==0，但更快。表示被观测的资源(也就是shared_ptr的管理的资源)已经不复存在。

## 6. static关键字

### 6.1.面向过程设计中的static

#### 6.1.1 静态全局变量

在全局变量前，加上关键字static，该变量就被定义成为一个静态全局变量。全局变量和静态全局变量都存放于程序的全局数据区域，它们的生存周期都是程序的整个运行期。这两者的区别在于非静态全局变量的作用域是整个源程序， 当一个源程序由多个源文件组成时，非静态的全局变量在各个源文件中都是有效的。 而静态全局变量则限制了其作用域， 即只在定义该变量的源文件内有效。

定义静态全局变量的好处：

- 由于静态全局变量的作用域局限于一个源文件内，只能为该源文件内的函数公用，因此可以避免在其它源文件中引起错误。

#### 6.1.2 静态局部变量

在局部变量前，加上关键字static，该变量就被定义成为一个静态局部变量。通常，在函数体内定义了一个变量，每当程序运行到该语句时都会给该局部变量分配栈内存。但随着程序退出函数体，系统就会收回栈内存，局部变量也相应失效。但有时候我们需要在两次调用之间对变量的值进行保存。通常的想法是定义一个全局变量来实现。但这样一来，变量已经不再属于函数本身了，不再仅受函数的控制，给程序的维护带来不便。
静态局部变量正好可以解决这个问题。静态局部变量保存在全局数据区，而不是保存在栈中，每次的值保持到下一次调用，直到下次赋新值。

静态局部变量有以下特点：
- 该变量在全局数据区分配内存；
- 静态局部变量在程序执行到该对象的声明处时被首次初始化，即以后的函数调用不再进行初始化；
- 静态局部变量一般在声明处初始化，如果没有显式初始化，会被程序自动初始化为0；
- 它始终驻留在全局数据区，直到程序运行结束。但其作用域为局部作用域，当定义它的函数或语句块结束时，其作用域随之结束；

#### 6.1.3 静态函数

在函数的返回类型前加上static关键字,函数即被定义为静态函数。静态函数与普通函数不同，它只能在声明它的文件当中可见，不能被其它文件使用。

定义静态函数的好处：
• 静态函数不能被其它文件所用；
• 其它文件中可以定义相同名字的函数，不会发生冲突；

### 6.2 面向对象的static关键字

### 6.2.1 静态数据成员

在类内数据成员的声明前加上关键字static，该数据成员就是类内的静态数据成员。

```cpp
    class MyClass
    {
    public:
        MyClass(int a, int b, int c);
        void fun();
    private:
        int a,b,c;
        static int sum;//声明静态数据成员
    };
     
     int MyClass::sum = 0;
```



静态数据成员有以下特点：

- 对于非静态数据成员，每个类对象都有自己的拷贝。而静态数据成员被当作是类的成员。无论这个类的对象被定义了多少个，静态数据成员在程序中也只有一份拷贝，由该类型的所有对象共享访问。

- 静态数据成员存储在全局数据区。静态数据成员定义时要分配空间，所以不能在类声明中定义。

- 因为静态数据成员在全局数据区分配内存，属于本类的所有对象共享，所以，它不属于特定的类对象，在没有产生类对象时其作用域就可见，即在没有产生类的实例时，我们就可以操作它；

- 静态数据成员初始化与一般数据成员初始化不同。静态数据成员初始化的格式为：＜数据类型＞＜类名＞::＜静态数据成员名＞=＜值＞

- 类的静态数据成员有两种访问形式：＜类对象名＞.＜静态数据成员名＞ 或 ＜类类型名＞::＜静态数据成员名＞

优点：

- 对该类的多个对象来说，静态数据成员只分配一次内存，供所有对象共用，更节省空间。
- 静态数据成员没有进入程序的全局名字空间，因此不存在与程序中其它全局名字冲突的可能性；
- 可以实现信息隐藏。静态数据成员可以是private成员，而全局变量不能

### 6.2.2 静态成员函数

静态成员函数与静态数据成员一样，都是类的内部实现，属于类定义的一部分。它无法访问属于类对象的非静态数据成员，也无法访问非静态成员函数，它只能调用其余的静态成员函数。

```cpp
#include <iostream>
using namespace std;
class Myclass {
public:
    Myclass(int a, int b, int c);
    static void GetSum();
// 声明静态成员函数 
private :
    int a, b, c;
    static int Sum; //声明静态数据成员
};
int Myclass::Sum = 0; //定义并初始化静态数据成员
Myclass::Myclass(int a, int b, int c) {
	this->a = a;
	this->b = b;
	this->c = c;
	Sum += a + b + c; //非静态成员函数可以访问静态数据成员
}
void Myclass::GetSum() //静态成员函数的实现
{
	// cout<<a<<endl; //错误代码，a是非静态数据成员
	cout << "Sum=" << Sum << endl;
}
int main() {
	Myclass M(1, 2, 3);
	M.GetSum();//Sum=6
	Myclass N(4, 5, 6);
	N.GetSum();//Sum=21
	Myclass::GetSum();//Sum=21
}
```

关于静态成员函数，可以总结为以下几点：
- 出现在类体外的函数定义不能指定关键字static；
- 静态成员之间可以相互访问，包括静态成员函数访问静态数据成员和访问静态成员函数；
- 非静态成员函数可以任意地访问静态成员函数和静态数据成员；
- 静态成员函数不能访问非静态成员函数和非静态数据成员；
- 由于没有this指针的额外开销，因此静态成员函数与类的全局函数相比速度上会有少许的增长；
- 调用静态成员函数，可以用成员访问操作符(.)和(->)为一个类的对象或指向类对象的指针调用静态成员函数，也可以直接使用如下格式：
＜类名＞::＜静态成员函数名＞（＜参数表＞）调用类的静态成员函数。

## 7. STL

### 7.1顺序容器中的at()和[]

如果stl非空，那么at和下标索引没有区别。如果v为空，调用at会抛出异常，而operator[]索引的行为未定义。c++标准不要求operator[]进行下标越界检查，原因是为了效率，下标越界检查会增加程序的性能开销。

```cpp
int main() {
    vector<int> vec;
    cout<<vec[0];//Segmentation fault
    cout<<vec.at(0);
    /*terminate called after throwing an instance of 'std::out_of_range'
  what():  vector::_M_range_check: __n (which is 0) >= this->size() (which is 0)
Aborted*/
}
int main() {
    vector<int> vec = {1};
    cout<<vec[1];//打印出0
    cout<<vec.at(1);
    /*terminate called after throwing an instance of 'std::out_of_range'
  what():  vector::_M_range_check: __n (which is 1) >= this->size() (which is 1)
Aborted*/
}
```
### 7.2 顺序容器中的push_back()和emplace_back()
emplace_back() (C++11)和 push_back() 的区别: push_back() 向容器尾部添加元素时，首先会创建这个元素，然后再将这个元素拷贝或者移动到容器中（如果是拷贝的话，事后会自行销毁先前创建的这个元素）；而 emplace_back() 在实现时，则是直接在容器尾部创建这个元素，省去了拷贝或移动元素的过程。
```cpp
#include <vector> 
#include <iostream> 
using namespace std;
class testDemo
{
public:
    testDemo(int num):num(num){
        std::cout << "调用构造函数" << endl;
    }
    testDemo(const testDemo& other) :num(other.num) {
        std::cout << "调用拷贝构造函数" << endl;
    }
    testDemo(testDemo&& other) :num(other.num) {
        std::cout << "调用移动构造函数" << endl;
    }
private:
    int num;
};

int main()
{
    cout << "emplace_back:" << endl;
    std::vector<testDemo> demo1;
    demo1.emplace_back(2);  

    cout << "push_back:" << endl;
    std::vector<testDemo> demo2;
    demo2.push_back(2);
}
```
输出结果：
```cpp
emplace_back:
调用构造函数
push_back:
调用构造函数
调用移动构造函数
```
在此基础上，将 testDemo 类中的移动构造函数注释掉，再运行程序会发现，运行结果变为：
```cpp
emplace_back:
调用构造函数
push_back:
调用构造函数
调用拷贝构造函数
```
由此可以看出，push_back() 在底层实现时，会优先选择调用移动构造函数，如果没有才会调用拷贝构造函数。
使用emplace_back() 函数可以减少一次拷贝或移动构造的过程，提升容器插入数据的效率。
### 7.3 vector扩容机制
vector有两个重要的变量：size和capacity。size表示当前实际存储元素的个数，而capacity表示当前容器在不扩容的情况下最多可以存储的元素个数。当size达到capacity时，vector会重新分配内存，将原有的元素拷贝到新的内存空间中，然后释放原内存空间。
resize()函数是改变vector的size，而reserve()函数是改变vector的capacity。
#### 7.3.1 vector扩容为什么是1.5倍或者2倍
如果新空间大小为旧空间大小+1，也就是边插入边扩容，这样每一次插入都要进行拷贝，时间复杂度为O(n)，效率非常低下。
如果新空间大小为旧空间大小+k，其中k是一个固定的增量，那么在每次扩容时，新空间的大小会增加k个单位。假设原始空间大小为n，进行m次扩容后，新空间的大小为n + m*k。平摊下来每次插入的时间复杂度还是O(n)，效率非常低下。
如果新空间大小为旧空间大小的n倍，那么在每次扩容时，新空间的大小会增加n倍。假设原始空间大小为n，进行m次扩容后，新空间的大小为n * n^m = n * n^(m-1) * n = n * 2^m。平摊下来每次插入的时间复杂度为O(1)，效率非常高。
扩容原理为：申请新空间，拷贝元素，释放旧空间，理想的分配方案是在第N次扩容时如果能复用之前N-1次释放的空间就太好了，如果按照2倍方式扩容，第i次扩容空间大小如下：
```cpp
1, 2, 4, 8, 16, 32
```
可以看到，每次扩容时，前面释放的空间都不能使用`8 > 1 + 2 + 3`。
而使用1.5倍（k=1.5）扩容时，在几次扩展以后，可以重用之前的内存空间了。或者说这个数列的增长速度不能超过Fibonacci数.　因为有F(0) +.. +F(n-1) = F(n+1)-1。理论上最好的增长速度是黄金分割比例，即1.618倍。
#### 7.3.2 vector为什么不原地扩容
标准库没有原地扩容否则失败的函数，标准库的realloc是尝试原地扩容，否则新开辟一块，然后memcpy，memcpy只能用于POD类型（通俗地讲，一个类、结构、共用体对象或非构造类型对象能通过二进制拷贝（如 memcpy()）后还能保持其数据不变正常使用的就是POD类型的对象。），对于非POD类型，需要调用拷贝构造函数，析构函数，这样就会导致原地扩容失败。
## 8. 深拷贝和浅拷贝，左值和右值，移动构造函数，移动语义，完美转发 (c++11)
### 8.1 深拷贝和浅拷贝
示例：
```cpp
class Vector{
    int num;
    int* a;
public:
    void ShallowCopy(Vector& v);
    void DeepCopy(Vector& v);
};
```
浅拷贝：只复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存（只是增加了一个指针指向已存在的内存地址）。
```cpp
//浅拷贝
void Vector::ShallowCopy(Vector& v){
    this->num = v.num;
    this->a = v.a;
}

```
深拷贝：会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象（增加了一个指针并且申请了一个新的内存，使这个增加的指针指向这个新的内存）。
```cpp  
//深拷贝
void Vector::DeepCopy(Vector& v){
    this->num = v.num;
    this->a = new int[num];
    for(int i=0;i<num;++i){a[i]=v.a[i]}
}

```

### 8.2 左值和右值
- 左值（lvalue） ：表达式结束后依然存在的持久对象，可以取地址。
- 左值引用（lvalue reference） ：“&”表示的引，左值引用只能绑定到左值，不能绑定到右值。
- 右值（rvalue） ：表达式结束后就不再存在的临时对象，不能取地址。字面量（字符字面量除外）、临时的表达式值、临时的函数返还值这些短暂存在的值都是右值。
- 右值引用（rvalue reference） ：“&&”表示的引用，右值引用只能绑定到右值，不能绑定到左值。
```cpp
//左值引用形参=>匹配左值
void Vector::Copy(Vector& v){
    this->num = v.num;
    this->a = new int[num];
    for(int i=0;i<num;++i){a[i]=v.a[i]}
}

//右值引用形参=>匹配右值
void Vector::Copy(Vector&& temp){
    this->num = temp.num;
    this->a = temp.a;
}
```
### 8.3 移动构造函数
C++11引入了移动构造函数，默认**移动**构造函数会执行浅拷贝操作，与**拷贝**构造函数不同的是，在拷贝结束后，移动构造函数会把源对象内的指针置空。移动构造函数的目的是为了提高效率，减少内存拷贝的次数，提高程序的性能。
```cpp
//拷贝构造函数：这意味着深拷贝
Vector::Vector(Vector& v){
    this->num = v.num;
    this->a = new int[num];
    for(int i=0;i<num;++i){a[i]=v.a[i]}
}
//移动构造函数：这意味着浅拷贝
Vector::Vector(Vector&& temp){
    this->num = temp.num;
    this->a = temp.a;
    temp.a = nullptr;    //实际上Vector一般都会在析构函数来释放指向的内存，所以需赋值空地址避免释放
}
```

### 8.4 移动语义(std::move)
为了将某些左值当成右值使用，C++11 提供了 std::move 函数以用于将某些左值转成右值，以匹配右值引用类型。主要用于调用移动构造函数和移动赋值函数。
```cpp
void func(){
    Vector result;
    Vector ans(std::move(result));  //调用移动构造函数
    Vector ans2(result);            //调用拷贝构造函数
    return;
}
```
### 8.5 完美转发 (C++11)
完美转发是C++11引入的一种技术，用于在模板函数中保持参数的值类别（左值或右值）不变地传递给另一个函数。完美转发的主要目的是避免不必要的拷贝和移动操作，从而提高程序的性能。
#### 8.5.1 万能引用
C++11中引入了右值引用，但是右值引用只能绑定到右值，为了解决这个问题，C++11引入了万能引用，也叫做转发引用，它是一种特殊的引用类型，可以同时绑定左值和右值。
```cpp  
template<typename T>
void func(T&& t){
    return;
}
int main() {
    Vector a,b;
	func(a);                //OK
	func(std::move(b));     //OK
}
``` 
#### 8.5.2 引用折叠
引用折叠是C++11中引入的一种规则，用于解决万能引用的问题。引用折叠规则如下：
- T& & 折叠为 T&
- T& && 折叠为 T&
- T&& & 折叠为 T&
- T&& && 折叠为 T&&

简单来说，任何类型的引用与左值引用组合时，结果都是左值引用；只有右值引用与右值引用组合时，结果才是右值引用。
#### 8.5.3 std::forward
std::forward是一个用于完美转发的函数，它可以将传入的参数原封不动的传递给下一个函数，不会丢失参数的左值或右值属性。
```cpp
#include <iostream>
#include <utility> // std::forward

// 一个简单的函数，用于展示参数的值类别
void process(int& x) {
    std::cout << "Lvalue reference: " << x << std::endl;
}

void process(int&& x) {
    std::cout << "Rvalue reference: " << x << std::endl;
}

// 使用万能引用和完美转发的模板函数
template<typename T>
void forwarder(T&& arg) {
    process(std::forward<T>(arg));
}

int main() {
    int a = 10;
    forwarder(a);        // 传递左值
    forwarder(20);       // 传递右值
    forwarder(std::move(a)); // 传递右值

    return 0;
}
```

## 9.仿函数（C++11）
仿函数又称为函数对象, 是一个能行使函数功能的类，它通过重载 operator() 运算符来实现。
**优点**：有时候我们希望在函数中存储一些的状态，并在调用中使用这些状态，按照以往的经验，我们可以考虑如下可能途径：
- 局部变量：只能在这个函数内部修改，不能在调用的地方传入，可拓展性不强
- 全局变量：会污染全局命名空间，不利于维护
- 函数传参：某些情况下可能不适用。
- 成员变量：仿函数通过将状态存储为类的成员变量，并通过构造函数传递参数，实现了状态的灵活管理和使用。
```cpp
#include <iostream>

// 普通函数
int multiply(int a, int b) {
    return a * b;
}

// 仿函数类
class Multiplier {
public:
    // 构造函数，初始化状态
    Multiplier(int factor) : factor(factor) {}

    // 重载 operator() 运算符
    int operator()(int a, int b) const {
        return a * b * factor; // 使用存储的状态
    }

private:
    int factor; // 存储状态
};

int main() {
    // 使用普通函数
    int result1 = multiply(3, 4);
    std::cout << "Result using normal function: " << result1 << std::endl;

    // 使用仿函数
    Multiplier multiplier(2); // 创建仿函数对象，并初始化状态为2
    int result2 = multiplier(3, 4); // 像调用函数一样使用仿函数对象
    std::cout << "Result using functor: " << result2 << std::endl; // 输出结果

    // 改变仿函数的状态
    Multiplier multiplier3(3); // 创建另一个仿函数对象，并初始化状态为3
    int result3 = multiplier3(3, 4); // 使用新的仿函数对象
    std::cout << "Result using functor with different state: " << result3 << std::endl; // 输出结果

    return 0;
}
```
## 10. lambda表达式（C++11）
lambda表达式是C++11引入的一种新特性，利用lambda表达式可以编写内嵌的匿名函数，用以替换独立函数或者函数对象，并且使代码更可读。
使用匿名函数的好处：
- 代码更加简洁，不需要额外定义一个函数
- 可以捕获外部变量，使得函数更加灵活

lambda表达式的语法如下：
```cpp
[capture list] (parameter list) -> return type { function body }
```
- capture list：捕获列表，用于捕获外部变量。[] 表示不捕获任何外部变量，[&] 表示以引用的方式捕获所有外部变量，[=] 表示以值的方式捕获所有外部变量，[a, &b] 表示以值的方式捕获 a 变量，以引用的方式捕获 b 变量。[this] 表示捕获当前对象中的所有成员变量。
- parameter list：参数列表，用于传递参数
- return type：返回类型，用于指定返回类型
- function body：函数体，用于编写函数的具体逻辑
## 11. nullptr(C++11)
nullptr是C++11引入的新关键字，用于表示空指针。
**优点**：
- 类型安全：nullptr 的类型是 std::nullptr_t，它只能转换为指针类型或 bool 类型，不能转换为整数类型。这确保了类型安全，避免了将空指针误用为整数的情况。
- 避免歧义：在函数重载和模板编程中，使用 nullptr 可以避免将空指针误解为整数，从而减少歧义和错误。
```cpp
#include <iostream>

// 函数重载示例
void foo(int x) {
    std::cout << "foo(int) called with " << x << std::endl;
}

void foo(void* ptr) {
    std::cout << "foo(void*) called" << std::endl;
}

int main() {
    // 使用 NULL 或 0 可能导致歧义
    foo(0); // 调用 foo(int)，而不是 foo(void*)

    // 使用 nullptr 明确表示空指针
    foo(nullptr); // 调用 foo(void*)

    return 0;
}
```
## explicit关键字
在 C++ 中，explicit 是一个关键字，用于修饰构造函数(作用于单个参数的构造函数，因为无参构造函数和多参构造函数本身就是显示调用的。再加上explicit关键字也没有什么意义)。它的主要作用是防止该构造函数用于隐式转换。
## noexcept关键字
noexcept 是 C++11 引入的一个关键字，用于指示函数不会抛出异常。C++中的异常处理是在运行时而不是编译时检测的。为了实现运行时检测，编译器创建额外的代码，然而这会妨碍程序优化。当一个函数声明为noexcept时，编译器可以假设该函数不会抛出异常，从而避免生成与异常处理相关的额外代码。这可以减少程序的运行时开销.