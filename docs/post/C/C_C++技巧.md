# C/C++ 技巧

## (char *)0

先看代码

``` c
int i = 0;
int j = 11;
char* p = (char*)0;
char* pp = (char*)i;
char* ppp = (char*)j;
char* pNULL = NULL;

printf("p addres:     0x%x\n", &p);
printf("pp addres:    0x%x\n", &pp);
printf("ppp addres:   0x%x\n", &ppp);
printf("pNULL addres: 0x%x\n", &pNULL);

printf("*******************************\n");

printf("p value:     0x%x\n", p);
printf("pp value:    0x%x\n", pp);
printf("ppp value:   0x%x\n", ppp);
printf("pNULL value: 0x%x\n", pNULL);

printf("*******************************\n");

printf("p value:     %d\n", p);
printf("pp value:    %d\n", pp);
printf("ppp value:   %d\n", ppp);
printf("pNULL value: %d\n", pNULL);
```

结果为：

``` bash
p addres:     0x863b4ac0				# 每次执行都不一样
pp addres:    0x863b4ab8				# 每次执行都不一样
ppp addres:   0x863b4ab0				# 每次执行都不一样
pNULL addres: 0x863b4aa8				# 每次执行都不一样
*******************************
p value:     0x0
pp value:    0x0
ppp value:   0xb
pNULL value: 0x0
*******************************
p value:     0
pp value:    0
ppp value:   11
pNULL value: 0
```

`(char *)0`，它确保产生一个类型为 `char *` 的空指针。理论上它不指向任何 `char` 类型的对象，它的实际地址由实现定义，可以是任何地址，不一定是 `0x0` 。（参考：[What does (char*) 0 mean?](https://stackoverflow.com/questions/36931022/what-does-char-0-mean)）

在我的测试机中，它是一个**指向地址为 `0x0` 的 `char *` 类型的指针**。 与 `NULL` 是一样的。

##  结构体成员偏移量

先看代码

``` c
#include <stdio.h>
#include <string.h>

#define PRINT_D(intValue)		printf(#intValue" is %d\n", (intValue));     #define PRINT_D(intValue)		printf(#intValue" is %d\n", (intValue));    #define PRINT_D(intValue)		printf(#intValue" is %d\n", (intValue));
#define OFFSET(struct,member)   ((char *)&((struct *)0)->member - (char *)0)
#define OFFSET(struct,member) 	((size_t) &((struct *) 0)->member)

#pragma pack(4)

typedef struct
{
    char	sex;
    short	score;
    int		age;
} student;

int main(void)
{
    PRINT_D(sizeof(student))
    PRINT_D(OFFSET(student, sex))
    PRINT_D(OFFSET(student, score))
    PRINT_D(OFFSET(student, age))
    return 0;      
}
```

`OFFSET` 是用来**判断结构体中成员的偏移位置**。下面来一步步分析该宏定义的意义。

`(struct *)0` 表示一个 `struct *` 类型的空指针。
`((struct *)0)->member` 表示取结构体中的成员。（`->` 的优先级高于 `&` ）
`&((struct *)0)->member` 表示取结构体中的成员的地址。
`(char *)&((struct *)0)->member` 表示将结构体的成员地址强制转换为 `char *` 类型。
`(char *)&((struct *)0)->member - (char *)0` 表示结构体成员的地址的偏移量。

另外的一个的解析：

`(struct *)0` 表示一个 `struct *` 类型的指针，其地址为 `0x0`。其作用就是把从地址 0 开始的存储空间映射为一个 struct_t 类型的对象。
`((struct *)0)->member` 表示取结构体中的成员。（`->` 的优先级高于 `&` ）
`&((struct *)0)->member` 表示取结构体中的成员的地址。**由于对象的起始地址为 0，所以成员的地址其实就是相对于对象首地址的成员的偏移地址**。
`(size_t)&((struct *)0)->member` 表示将偏移地址转换为 `size_t` 类型。

参考：

- [理解＃define offsetof(struct_t,member) ((int)&((struct_t *)0)->member)](https://www.cnblogs.com/zhangfeionline/p/6212375.html)
- [#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)](<https://blog.csdn.net/u014114046/article/details/52122693>)

## 全局静态变量

上代码

```cpp
///////////////////////////////////////////////
/// a.h
///////////////////////////////////////////////
#ifndef TEST10_A_H
#define TEST10_A_H

static int a = 100;
//extern int a;

int getA();
void setA(int parame);

#endif		/// TEST10_A_H


///////////////////////////////////////////////
/// a.cc
///////////////////////////////////////////////
#include <stdio.h>
#include <stdlib.h>
#include "a.h"

//int a = 100;

int getA()
{
    return a;
}

void setA(int parame)
{
    printf("before setA fnction: a address = 0x%x, a = %d, parame=%d\n", &a, a, parame);
    a = parame;
    printf("after setA fnction: a address = 0x%x, a = %d, parame=%d\n", &a, a, parame);
}


///////////////////////////////////////////////
/// main.cc
///////////////////////////////////////////////
#include <stdio.h>
#include <stdlib.h>
#include "a.h"

void test()
{
    setA(99);
    printf("AAAAA test function a address = 0x%x, a = %d\n", &a, a);
    a = 98;
    printf("BBBBB test function a address = 0x%x, a = %d\n", &a, a);
}

int main(void)
{
    test();
    printf("main funcion a address = 0x%x, a = %d, getA() = %d\n", &a, a, getA());
    
    return 0;
}
```

变量 `a` 为静态全局变量，分别在不同的文件中使用到。
最初，我以为 `a` 是全局变量，`main` 函数中最后打印的 `a` 与 `getA` 的结果应该是一致。 

其真正的结果如下：

```bash
before setA fnction: a address = 0x60102c, a = 100, parame=99
after setA fnction: a address = 0x60102c, a = 99, parame=99
AAAAA test function a address = 0x601030, a = 100
BBBBB test function a address = 0x601030, a = 98
main funcion a address = 0x601030, a = 98, getA() = 99
```

从结果中我们可以看出，在函数 `setA` 内的静态全局变量 `a`，与在函数 `test` 内 `a` 是不一样的。（地址不一致）。而在函数 `test` 与在函数 `main` 中的 `a` 是一致的。

这是因为：

- static修饰的全局变量的声明与定义同时进行，即当你在头文件中使用static声明了全局变量，同时它也被定义了。
- static修饰的全局变量的作用域只能是本身的编译单元。在其他编译单元使用它时，只是简单的把其值复制给了其他编译单元，其他编译单元会另外开个内存保存它，在其他编译单元对它的修改并不影响本身在定义时的值。
- 多个地方引用静态全局变量所在的头文件，不会出现重定义错误，因为在每个编译单元都对它开辟了额外的空间进行存储。

因此，**一般定义static 全局变量时，都把它放在.cpp文件中而不是.h文件中，这样就不会给其他编译单元造成不必要的信息污染**。

如果需要在多文件使用的全局变量，则可以使用 `extern` 。
在头文件中使用 `extern`，然后在 .cpp 文件中定义。(代码中注释掉的部分)