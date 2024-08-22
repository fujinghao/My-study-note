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