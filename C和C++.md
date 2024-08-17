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

## 2.内存对齐
