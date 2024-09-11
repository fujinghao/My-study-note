## 1. 段错误（Segmentation Fault）

(1) 修改只读内容

```c
int main(){
    char *c = "hello world";
    c[1] = 'H';//编译没有问题，但是运行时弹出Segmentation Fault
} 
```

此例中，”hello world”作为一个常量字符串，在编译后会被放在.rodata节（GCC），最后链接生成目标程序时.rodata节会被合并到text segment与代码段放在一起，故其所处内存区域是只读的。

(2) 访问了不属于进程地址空间的内存:

```c
int main(){
  int* p = (int*)0xC0000fff;
  *p = 10;
}
```

(3) 访问了不存在的内存：

```c
int main(){
  int *p = NULL;
  *p = 1;
}
```

(4) 试图把一个整数按照字符串的方式输出:

```c
int main() {
    int b = 10;
    printf("%s\n", b);
    return 0;
}　
```

在打印字符串的时候，实际上是打印某个地址开始的所有字符，但是当你想把整数当字符串打印的时候，这个整数被当成了一个地址，然后printf从这个地址开始去打印字符，直到某个位置上的值为\0。所以，如果这个整数代表的地址不存在或者不可访问，自然也是访问了不该访问的内存

## 2. 内存对齐
### 2.1 什么是内存对齐?
理论上计算机对于任何变量的访问都可以从任意位置开始，然而实际上系统会对这些变量的存放地址有限制，通常将变量首地址设为某个数N的倍数，这就是内存对齐。

### 2.2 为什么要内存对齐?
- 硬件平台限制，内存以字节为单位，不同硬件平台不一定支持任何内存地址的存取，一般可能以双字节、4字节等为单位存取内存，为了保证处理器正确存取数据，需要进行内存对齐。
- 提高CPU内存访问速度，一般处理器的内存存取粒度都是N的整数倍，假如访问N大小的数据，没有进行内存对齐，有可能就需要两次访问才可以读取出数据，而进行内存对齐可以一次性把数据全部读取出来，提高效率。
## 3. C语言中sizeof()和strlen()的区别
- strlen 计算的是字符串的实际长度，遇到\0即停止；sizeof 计算整个字符串所占内存字节数的大小，当然\0也要+1计算；
- sizeof()在编译时计算好了，strlen()在运行时计算；
- sizeof()的参数类型多样化（数组，指针，对象，函数都可以），strlen()的参数必须是字符型指针（传入数组时自动退化为指针）
```cpp
int main() {
	string s = "abcd";
	cout<<sizeof(s)<<" "<<strlen(s.c_str())<<endl;// 32 4，
	s = "abcdefg";
    cout << sizeof(s) << endl; // 32 ,32位机下为4
    const char* str = "abcd";
    cout << sizeof(str) << " " << strlen(str) << endl;// 8 4
    char a[] = "abcd";
    cout << sizeof(a) << " " << strlen(a) << endl;// 5 4 sizeof会计算\0的
}
```
## 4. C语言实现多态和继承
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 基类结构体
typedef struct {
    void (*speak)(void*); // 函数指针
} Animal;

// 派生类结构体
typedef struct {
    Animal base; // 继承自 Animal
    char name[20];
} Dog;

// 基类的 speak 函数
void animalSpeak(void* self) {
    printf("Some generic animal sound\n");
}

// 派生类的 speak 函数
void dogSpeak(void* self) {
    Dog* dog = (Dog*)self;
    printf("%s says: Woof!\n", dog->name);
}

// 基类的构造函数
void animalInit(Animal* animal) {
    animal->speak = animalSpeak;
}

// 派生类的构造函数
void dogInit(Dog* dog, const char* name) {
    animalInit(&dog->base); // 初始化基类部分
    dog->base.speak = dogSpeak; // 重写 speak 函数
    strncpy(dog->name, name, sizeof(dog->name) - 1);
    dog->name[sizeof(dog->name) - 1] = '\0';
}

int main() {
    Animal genericAnimal;
    Dog myDog;

    animalInit(&genericAnimal);
    dogInit(&myDog, "Buddy");

    // 多态行为演示
    genericAnimal.speak(&genericAnimal); // 输出: Some generic animal sound
    myDog.base.speak(&myDog); // 输出: Buddy says: Woof!

    return 0;
}
```

只实现多态：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 定义结构体
typedef struct {
    void (*speak)(void*); // 函数指针
} Animal;

// Dog 的 speak 函数
void dogSpeak(void* self) {
    printf("Dog says: Woof!\n");
}

// Cat 的 speak 函数
void catSpeak(void* self) {
    printf("Cat says: Meow!\n");
}

// 初始化 Animal 结构体
void animalInit(Animal* animal, void (*speakFunc)(void*)) {
    animal->speak = speakFunc;
}

int main() {
    Animal dog;
    Animal cat;

    // 初始化不同的动物
    animalInit(&dog, dogSpeak);
    animalInit(&cat, catSpeak);

    // 多态行为演示
    dog.speak(&dog); // 输出: Dog says: Woof!
    cat.speak(&cat); // 输出: Cat says: Meow!

    return 0;
}
```
## 5. 指针和引用的区别
- 指针是一个变量，存储的是一个地址，指向内存的一个存储单元。而引用跟原来的变量实质上是同一个东西，只不过是原变量的一个别名而已。
- 指针的值可以为空，但是引用的值不能为NULL，并且引用在定义的时候必须初始化；
- 指针的值在初始化后可以改变，即指向其它的存储单元，而引用在进行初始化后就不会再改变了，从一而终。
## 6. new 和 malloc 的区别
- 语法不同：malloc/free是一个C语言的函数，而new/delete是C++的运算符。
- 分配内存的方式不同：malloc只分配内存，而new会分配内存并且调用对象的构造函数来初始化对象。
- 返回类型不同：malloc返回void*，需要强制类型转换，而new返回对象的指针，不需要强制类型转换。
- malloc 需要传入需要分配的大小，而 new 编译器会自动计算所构造对象的大小
- new内存分配失败时，会抛出bac_alloc异常，它不会返回NULL；malloc分配内存失败时返回NULL。

### 6.1 new分配的内存可以调用free释放吗？
不能。new实际过程中做了两步操作，第一步是分配内存空间，第二步是调用类的构造函数；delete也同样是两步，第一步是调用类的析构函数，第二步才是释放内存；而malloc()和free()仅仅是分配内存与释放内存操作；那么如果通过new分配的内存，再用free去释放，就会少一步调用析构函数的过程。