# C++基础语法 & 内存管理

### 1、[strcpy,sprintf,memcpy的区别](https://www.cnblogs.com/biyeymyhjob/archive/2012/07/12/2588658.html)
区别在于 **实现功能** 以及 **操作对象** 不同

（1）strcpy 函数操作的对象是 **字符串** ，完成 从 **源字符串** 到 **目的字符串** 的 拷贝 功能

（2）sprintf 函数操作的对象 **不限于字符串** ：虽然目的对象是字符串，但是源对象可以是字符串、也可以是任意基本类型的数据。这个函数主要用来**实现 （字符串或基本数据类型）向 字符串 的转换 功能**。如果源对象是字符串，并且指定 %s 格式符，也可实现字符串拷贝功能。

```c++
#include <stdio.h>
#include <math.h>

int main()
{
   char str[80];

   sprintf(str, "Pi 的值 = %f", M_PI);
   puts(str);
   
   return(0);
}

/*
OUTPUT:Pi 的值 = 3.141593
*/

//C 库函数 int sprintf(char *str, const char *format, ...) 发送格式化输出到 str 所指向的字符串。
```

（3）memcpy 函数顾名思义就是 **内存拷贝** ，实现 将一个 **内存块** 的内容复制到另一个 **内存块** 这一功能。**内存块由其首地址以及长度确定**。程序中出现的实体对象，不论是什么类型，其最终表现就是在内存中占据一席之地（一个内存区间或块）。因此，memcpy 的操作对象不局限于某一类数据类型，或者说可 适用于任意数据类型 ，只要能给出对象的起始地址和内存长度信息、并且对象具有可操作性即可。**鉴于 memcpy 函数等长拷贝的特点以及数据类型代表的物理意义，memcpy 函数通常限于同种类型数据或对象之间的拷贝，其中当然也包括字符串拷贝以及基本数据类型的拷贝。**

**（4）**

- 复制的内容不同。strcpy只能复制字符串，而memcpy可以复制任意内容，例如字符数组、整型、结构体、类等。
-  复制的方法不同。strcpy不需要指定长度，它遇到被复制字符的串结束符"\0"才结束，所以容易溢出。memcpy则是根据其第3个参数决定复制的长度。 
- 用途不同。通常在复制字符串时用strcpy，而需要复制其他类型数据时则一般用memcpy

### 2、[sizeof 与 strlen的区别](https://www.runoob.com/note/27755)

- **sizeof是运算符**，并不是函数，**结果在编译时得到而非运行中获得**；strlen是字符处理的库函数。
- sizeof参数可以是任何数据的类型或者数据（sizeof参数不退化）；strlen的参数只能是字符指针且结尾是'\0'的字符串。
- 因为sizeof值在编译时确定，所以不能用来得到动态分配（运行时分配）存储空间的大小。

```c++
int main(int argc, char const *argv[]){

      const char* str = "name";

      sizeof(str); // 取的是指针str的长度，是8
      strlen(str); // 取的是这个字符串的长度，不包含结尾的 \0。大小是4
      return 0;
}
```

### 3、sizeof综合题（类）

- **sizeof一个空类型的结果是什么，为什么？**

  - **定义一个空的类型，里面没有任何数据成员和成员函数，对该类型求sizeof，结果为1**；

  - 空类型的实例中不包含任何信息，本来求 sizeof应该是0,**但是当我们声明该类型的实例的时候，它必须在内存中占有一定的空间，需要区分不同的对象，否则无法使用这些实例**。
  - 至于占用多少内存，由编译器决定。 **Visual Studio及GCC编译器中每个空类型的实例占用1字节的空间**

- **如果在空类型的实例中添加一个构造函数和析构函数，再sizeof的结果是多少，为什么？**

  - 结果还是 1
  - 类所占内存的大小是由**成员变量（静态变量除外）决定的**，成员函数（非虚函数）是不计算在内的。
  - 调用构造函数和析构函数只需要知道**函数的地址**即可，而这些函数的地址只与类型相关，而与类型的实例无关，编译器也不会因为这两个函数而在实例内添加任何额外的信息

- **如果把析构函数标记为虚析构函数？**
  
  - **类中有虚函数的时候有一个指向虚函数的指针（vptr），在32位系统分配指针大小为4字节，64位系统指针大小为8字节**。无论多少个虚函数，只有这一个指针。//注意一般的函数是没有这个指针的，而且也不占类的内存。
  
- 当此类被继承时，**子类的大小是本身成员变量的大小加上父类的大小**

- 总结：
  - 空的类是会占用内存空间的，而且大小是1，原因是C++要求每个实例在内存中都有独一无二的地址。
    - （一）类内部的成员变量：
      普通的变量：是要占用内存的(类里面)，但是要注意对齐原则（这点和struct类型很相似）。
      
      static修饰的静态变量：**不占用内容，原因是编译器将其放在全局变量区**。
      
    - （二）类内部的成员函数：
      普通函数：不占用内存。类的非虚成员函数是不计算在内的,不管它是否静态
      
    - 静态成员函数的存放问题：**静态成员函数与一般成员函数的唯一区别就是没有this指针**，因此不能访问非静态数据成员。就像我前面提到的，**所有函数都存放在代码区，静态函数也不例外。所有有人一看到 static 这个单词就主观的认为是存放在全局数据区，那是不对的。**
    
    - 虚函数：要占用4个字节，用来指定虚函数的虚拟函数表的入口地址。所以一个类的虚函数所占用的地址是不变的，和虚函数的个数是没有关系的
  
- **综上所述**

  - 同一个类创建的多个对象，其数据成员是各用各的，互不相通（**静态成员变量是共享的**）。
  - **成员函数是共享共用的**，多个对象共用一份代码，**所有类成员函数和非成员函数代码存放在代码区**。
  - 不论成员函数在类内定义还是在类外定义，成员函数的代码段都用同一种方式存储。

### 4、拷贝（复制）构造函数

（1）用一个已经存在的对象去初始化一个刚刚创建的对象，会调用拷贝构造函数

（2）当函数的参数时类类型，再函数调用进行实参与形参结合时，会调用拷贝构造函数

（3）当函数的返回类型是类类型时，再函数调用结束后，执行return语句时，会调用拷贝构造函数

`A other`

`A my = other`

`A(const A& other)`拷贝构造函数

- 拷贝构造函数参数中的引用不可以去掉
  - 若没有使用引用，再调用拷贝构造函数时，会进行实参与形参的结合，会满足拷贝构造的调用时机，会一直循环调用，但是没有出口，并且函数参数会一直入栈，直到栈溢出
- const不可以去掉
  - 去掉const后，在进行拷贝构造传入实参是右值（临时对象、匿名对象、字面值常量【不能进行取地址】）时，会报错（非const左值引用不能绑定到右值）

### 5、赋值运算符函数

```c++
String &String::operator=(const String &rhs)
//返回值类型为该类型的引用，才可以允许连续赋值
//传入的参数声明为常量引用，可以避免一次拷贝构造函数，且不会改变传入的实例的状态
{
    cout << "operator=(const String &rhs)" << endl;
    if(this != &rhs)//1、判断是否自复制，是则不执行
    {
        delete [] _pstr;//2、释放左操作数（防止内存泄漏）
        _pstr = nullptr;

        _pstr = new char[strlen(rhs._pstr) + 1]();//3、深拷贝
        strcpy(_pstr, rhs._pstr);

    }

    return *this;//4、返回*this
}
```

更加安全的代码

```c++
String &String::operator=(const String &rhs)
//返回值类型为该类型的引用，才可以允许连续赋值
//传入的参数声明为常量引用，可以避免一次拷贝构造函数，且不会改变传入的实例的状态
{
    if (this != &rhs) {
        
        String strTemp(rhs);//拷贝构造

        char *pTemp = strTemp._pstr;
        strTemp._pstr = _pstr;
        _pstr = pTemp;
        //由于strTemp是一个局部变量，但程序运行到if的外面时也就出了该变量的作用域，就会自动调用strTemp的析构函数，把 strTemp._pstr所指向的内存释放掉，相当于自动调用析构函数释放实例的内存
    }

    return *this;
}
```

### 6、const和static的作用

**static**

- 不考虑类的情况
  - 隐藏。所有**不加static的全局变量和函数具有全局可见性，可以在其他文件中使用，加了之后只能在该文件所在的编译模块中使用**
  - **默认初始化为0，包括未初始化的全局静态变量与局部静态变量，都存在全局未初始化区**
  - 静态变量在函数内定义，始终存在，且只进行一次初始化，具有记忆性，其作用范围与局部变量相同，函数退出后仍然存在，但不能使用
- 考虑类的情况
  - static成员变量：只与类关联，不与类的对象关联。**定义时要分配空间，不能在类声明中初始化，必须在类定义体外部初始化，初始化时不需要标示为static；可以被非static成员函数任意访问**。
  - static成员函数：不具有this指针，无法访问类对象的非static成员变量和非static成员函数；**不能被声明为const、虚函数和volatile**；可以被非static成员函数任意访问

**const**

- 不考虑类的情况
  - c**onst常量在定义时必须初始化，之后无法更改**
  - const形参可以接收const和非const类型的实参，例如// i 可以是 int 型或者 const int 型void fun(const int& i){ //...}
- 考虑类的情况
  - const成员变量：**不能在类定义外部初始化，只能通过构造函数初始化列表进行初始化**，并且必须有构造函数；不同类对其const数据成员的值可以不同，所以不能在类中声明时初始化
  - const成员函数：**const对象不可以调用非const成员函数；非const对象都可以调用**；不可以改变非mutable（用该关键字声明的变量可以在const成员函数中被修改）数据的值

### 7、[C++中函数指针和指针函数的区别？](https://www.cnblogs.com/gmh915/archive/2010/06/11/1756067.html)

 ```c++
 int *p[10]
     //int *p[10]表示指针数组，强调数组概念，是一个数组变量，数组大小为10，数组内每个元素都是指向int类型的指针变量
 int (*p)[10]
     //int (*p)[10]表示数组指针，强调是指针，只有一个变量，是指针类型，不过指向的是一个int类型的数组，这个数组大小是10。
 int *p(int)//(指针函数)
     //int *p(int)是函数声明，函数名是p，参数是int类型的，返回值是int *类型的。
 int (*p)(int)//(函数指针)
     //int (*p)(int)是函数指针，强调是指针，该指针指向的函数具有int类型参数，并且返回值是int类型
 ```

注意指针函数与函数指针表示方法的不同，千万不要混淆。最简单的辨别方式就是看函数名前面的指针*号有没有被括号（）包含，如果被包含就是函数指针，反之则是指针函数。

- 指针函数：
  - 当一个函数声明其返回值为一个指针时，实际上就是返回一个地址给调用函数，以用于需要指针或地址的表达式中。

- 函数指针是指向函数的指针变量，即本质是一个指针变量。
  - int (*f) (int x); /* 声明一个函数指针 */*
  - *f=func; /* 将func函数的首地址赋给指针f */
  - 把函数的地址赋值给函数指针，可以采用下面两种形式：
        **fptr=&Function;**
        fptr=Function;
      取地址运算符&不是必需的，因为单单一个函数标识符就标号表示了它的地址，如果是函数调用，还必须包含一个圆括号括起来的参数表。
      可以采用如下两种方式来通过指针调用函数：
        **x=(*fptr)();**
        x=fptr();
      第二种格式看上去和函数调用无异。但是有些程序员倾向于使用第一种格式，**因为它明确指出是通过指针而非函数名来调用函数的。**

```c++
//例子
void (*funcp)();
        void FileFunc(),EditFunc();

        main()
        {
            funcp=FileFunc;
            (*funcp)();
            funcp=EditFunc;
            (*funcp)();
        }

        void FileFunc()
        {
            printf(FileFunc\n);
        }

        void EditFunc()
        {
            printf(EditFunc\n);
        }

        程序输出为：
            FileFunc
            EditFunc
```



### 8、malloc/free与new/delete之间的异同点？【⭐】

**相同点**

- **都可用于内存的动态申请和释放**

**不同点**

- **前者是C++运算符，后者是C/C++语言标准库函数**
- **new自动计算要分配的空间大小，malloc需要手工计算**
- **new是类型安全的，malloc不是**。例如：

```cpp
int *p = new float[2]; //编译错误
int *p = (int*)malloc(2 * sizeof(double));//编译无错误Copy to clipboardErrorCopied
```

- new调用名为**operator new**的标准库函数分配足够空间并调用相关对象的**构造函数**，delete对指针所指对象运行适当的**析构函数**；然后通过调用名为**operator delete**的标准库函数释放该对象所用内存。后者均没有相关调用
- 后者需要库文件支持，前者不用
- **new是封装了malloc，直接free不会报错，但是这只是释放内存，而不会析构对象**
- **malloc申请的是原始未初始化空间，new申请的是已初始化空间**

### 9、指针和引用的区别【⭐】

- **指针是一个变量，存储的是一个地址**，**引用**跟原来的变量实质上是同一个东西，**是原变量的别名**
- 指针可以有**多级**，引用只有**一级**
- **指针可以为空，引用不能为NULL且在定义时必须初始化**
- **指针在初始化后可以改变指向，而引用在初始化之后不可再改变**
- **sizeof指针得到的是本指针的大小，sizeof引用得到的是引用所指向变量的大小**
- 当把指针作为参数进行传递时，也是**将实参的一个拷贝传递给形参，两者指向的地址相同，但不是同一个变量，在函数中改变这个变量的指向不影响实参，而引用却可以**。
- 引用本质是一个指针，同样会占4字节内存；指针是具体变量，需要占用存储空间（，具体情况还要具体分析）。

```c++
void test(int *p)
{
　　int a=1;
　　p=&a;
　　cout<<p<<" "<<*p<<endl;
}

int main(void)
{
    int *p=NULL;
    test(p);
    if(p==NULL)
    cout<<"指针p为NULL"<<endl;
    return 0;
}
//运行结果为：
//0x22ff44 1
//指针p为NULL


void testPTR(int* p) {
    int a = 12;
    p = &a;

}

void testREFF(int& p) {
    int a = 12;
    p = a;

}
void main()
{
    int a = 10;
    int* b = &a;
    testPTR(b);//改变指针指向，但是没改变指针的所指的内容
    cout << a << endl;// 10
    cout << *b << endl;// 10

    a = 10;
    testREFF(a);
    cout << a << endl;//12
}
```

### 10、[引用作为函数返回时为什么不能返回局部变量？](https://blog.csdn.net/u010177010/article/details/50802787)

**因为返回的都是值拷贝!    返回引用事实上是返回变量的地址。**

- **局部变量的作用域只在函数内部，在函数返回后，局部变量的内存已经释放了**。因此，如果函数返回的是局部变量的值，不涉及地址，程序不会出错。但是如果返回的是局部变量的地址(指针)的话，程序运行后会出错。因为函数只是把指针复制后返回了，但是指针指向的内容已经被释放了，这样指针指向的内容就是不可预料的内容，调用就会出错。准确的来说，**函数不能通过返回指向栈内存的指针(注意这里指的是栈，返回指向堆内存的指针是可以的)**。
- **所谓的不可以返回局部变量的引用或指针，指的是不能返回局部变量的引用或地址给引用或指针。事实上还是看该地址的位置是否在该函数的栈区，若是在栈区，函数调用结束，该地址就被释放了。尽管会出现栈地址上的值没被销毁的问题，可能是该栈区还没被其他的函数堆栈掉。**

```c++
#include <stdio.h> 
char *returnStr() 
{ 
    char *p="hello world!"; 
    return p; 
} 
int main() 
{ 
    char *str; 
    str=returnStr(); 
    printf("%s\n", str); 
    return 0; 
}
 
这个没有任何问题，因为"hello world!"是一个字符串常量，存放在只读数据段，
把该字符串常量存放的只读数据段的首地址赋值给了指针，所以returnStr函数退出时，
该该字符串常量所在内存不会被回收，故能够通过指针顺利无误的访问。

=======================================
#include <stdio.h> 
char *returnStr() 
{ 
    char p[]="hello world!"; 
    return p; 
} 
int main() 
{ 
    char *str; 
    str=returnStr(); 
    printf("%s\n", str); 
    return 0; 
} 
 
"hello world!"是局部变量存放在栈中。当returnStr函数退出时，栈要清空，
局部变量的内存也被清空了，所以这时的函数返回的是一个已被释放的内存地址，
所以有可能打印出来的是乱码。 
    
=========================================
int func()
{
      int a;
      ....
      return a;    //允许
}                   
 
int * func()
{
      int a;
      ....
      return &a;    //无意义，不应该这样做
} 
 
局部变量也分局部自动变量和局部静态变量，由于a返回的是值，因此返回一个局部变量是可以的，无论自动还是静态，
 
因为这时候返回的是这个局部变量的值，但不应该返回指向局部自动变量的指针，因为函数调用结束后该局部自动变量
 
被抛弃，这个指针指向一个不再存在的对象，是无意义的。但可以返回指向局部静态变量的指针，因为静态变量的生存
 
期从定义起到程序结束。
    
=========================================
int  & fun()

{

	int a = 10;

	return a;

}      

不可以，尝试返回 a 的地址给引用变量，a是存在栈里的，函数结束调用栈被销毁。
```

通常，变量的意义在于，它给一块内存存储区提供名字，方便程序对这块内存进行读写。变量包含两个值：左值和右值。**左值是内存存储区的名字，右值是存放存储区中的值**。

### 11、extern"C"的用法

为了能够**正确的在C++代码中调用C语言**的代码：在程序中加上extern "C"后，相当于告诉编译器这部分代码是C语言写的，因此要按照C语言进行编译，而不是C++；

哪些情况下使用extern "C"：

（1）C++代码中调用C语言代码；

（2）在C++中的头文件中使用；

（3）在多个人协同开发时，可能有人擅长C语言，而有人擅长C++；

**在C语言的头文件中，对其外部函数只能指定为extern类型，C语言中不支持extern "C"声明，在.c文件中包含了extern "C"时会出现编译语法错误。**所以使用**extern "C"全部都放在于cpp程序相关文件或其头文件中**。

```c++
//C++调用C函数
//xx.h
extern int add(...)

//xx.c
int add(){

}

//xx.cpp
extern "C" {
    #include "xx.h"
}

=======================
//C函数调用C++
//xx.h
extern "C"{
    int add();
}
//xx.cpp
int add(){    
}
//xx.c
extern int add();
```

### 12、内联函数和宏定义的区别

**内联函数：**在函数调用时候，用函数体进行替代函数

- 在使用时，**宏只做简单字符串替换**（编译前）。**而内联函数可以进行参数类型检查（编译时），且具有返回值。**
- 内联函数在编译时**直接将函数代码嵌入到目标代码中**，省去函数调用的开销来**提高执行效率**，并且进行参数类型检查，具有返回值，可以实现重载。
- **宏定义时要注意书写（参数要括起来）否则容易出现歧义**，**内联函数不会产生歧义**
- 内联函数有类型检测、语法判断等功能，而宏没有

**内联函数适用场景:**

- 使用宏定义的地方都可以使用 inline 函数。
- 作为类成员接口函数来读写类的私有成员或者保护成员，会提高效率。

### 13、C++内存分为哪几块？有什么作用？（内存管理相关）【⭐】

![image-20210912221304283](C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210912221304283.png)

**栈**：在执行函数时，函数内**函数参数、局部变量**的存储单元都可以在栈上创建，**函数执行结束时这些存储单元自动被释放**。栈**内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限**

**堆**：就是那些**由 `new`分配的内存块**，他们的释放编译器不去管，由我们的应用程序去控制，**一般一个`new`就要对应一个 `delete`。如果程序员没有释放掉，那么在程序结束后，操作系统会自动回收**

**自由存储区**：如果说堆是操作系统维护的一块内存，那么自由存储区就是C++中通过new和delete动态分配和释放对象的抽象概念。需要注意的是，自由存储区和堆比较像，但不等价。

**全局/静态存储区**：**全局变量和静态变量被分配到同一块内存中**，在以前的C语言中，全局变量和静态变量又分为初始化的和未初始化的，在C++里面没有这个区分了，它们共同占用同一块内存区，**在该区定义的变量若没有初始化，则会被自动初始化，例如int型变量自动初始为0**

**常量存储区**：这是一块比较特殊的存储区，这里面存放的是**常量，不允许修改**

**代码区**：存放函数体的**二进制代码**

### 14、静态变量什么时候初始化

- **初始化只有一次**，但是可以**多次赋值**，在主程序（main函数）之前，**编译器已经为其分配好了内存**。

- 静态局部变量和全局变量一样，数据都**存放在全局区域**，所以在主程序之前，编译器已经为其分配好了内存，但在C和C++中静态局部变量的初始化节点又有点不太一样。在C中，初始化发生在代码执行之前，编译阶段分配好内存之后，就会进行初始化，所以我们看到在C语言中无法使用变量对静态局部变量进行初始化，在程序运行结束，变量所处的全局内存会被全部回收。

- 而在C++中，初始化时在执行相关代码时才会进行初始化，主要是由于C++引入对象后，**要进行初始化必须执行相应构造函数和析构函数**，在构造函数或析构函数中经常会需要进行某些程序中需要进行的特定操作，并非简单地分配内存。所以C++标准定为全局或静态对象是有首次用到时才会进行构造，并通过atexit()来管理。在程序结束，按照构造顺序反方向进行逐个析构。**所以在C++中是可以使用变量对静态局部变量进行初始化的**。

### 15、动态编译与静态编译

- 静态编译，编译器在**编译可执行文件时**，把需要**用到的对应动态链接库中的部分提取出来**，连接到可执行文件中去，**使可执行文件在运行时不需要依赖于动态链接库**；

- 动态编译的**可执行文件需要附带一个动态链接库**，**在执行时，需要调用其对应动态链接库的命令**。所以其**优点**一方面是**缩小了执行文件本身的体积**，另一方面是**加快了编译速度，节省了系统资源**。**缺点**是哪怕是很简单的程序，只用到了链接库的一两条命令，也需**要附带一个相对庞大的链接库**；二是**如果其他计算机上没有安装对应的运行库，则用动态编译的可执行文件就不能运行。**

### 16、C++和C的struct区别

- C语言中：struct是**用户自定义数据类型**（UDT）；C++中struct是**抽象数据类型**（ADT），**支持成员函数的定义，（C++中的struct能继承，能实现多态）**
- C中struct是没有权限的设置的，且struct中只能是**一些变量的集合体**，**可以封装数据却不可以隐藏数据**，而且成员**不可以是函数**
- C++中，s**truct增加了访问权限，且可以和类一样有成员函数，成员默认访问说明符为public**（为了与C兼容）
- struct作为类的一种特例是用来自定义数据结构的。一个结构标记声明后，在C中必须在结构标记前加上struct，才能做结构类型名（除：typedef struct class{};）;C++中结构体标记（结构体名）可以直接作为结构体类型名使用，此外结构体struct在C++中被当作类的一种特例

### 17、深拷贝与浅拷贝区别

**浅拷贝**

**浅拷贝只是拷贝一个指针，并没有新开辟一个地址**，拷贝的指针和原来的指针指向同一块地址，**如果原来的指针所指向的资源释放了，那么再释放浅拷贝的指针的资源就会出现错误。**

**深拷贝**

**深拷贝不仅拷贝值，还开辟出一块新的空间用来存放新的值，即使原先的对象被析构掉，释放内存了也不会影响到深拷贝得到的值。**在自己**实现拷贝赋值的时候**，如果有**指针变量的话是需要自己实现深拷贝的**。

### 18、零拷贝技术

零拷贝就是一种**避免 CPU 将数据从一块存储拷贝到另外一块存储**的技术。

零拷贝技术可以**减少数据拷贝和共享总线操作的次数**。

在C++中，vector的一个成员函数**emplace_back()**很好地体现了零拷贝技术，它跟push_back()函数一样可以将一个元素插入容器尾部，区别在于：**使用push_back()函数需要调用拷贝构造函数和转移构造函数，而使用emplace_back()插入的元素原地构造，不需要触发拷贝构造和转移构造**，效率更高。

```c++
#include <vector>
#include <string>
#include <iostream>
using namespace std;

struct Person
{
    string name;
    int age;
    //初始构造函数
    Person(string p_name, int p_age): name(std::move(p_name)), age(p_age)
    {
         cout << "I have been constructed" <<endl;
    }
     //拷贝构造函数
     Person(const Person& other): name(std::move(other.name)), age(other.age)
    {
         cout << "I have been copy constructed" <<endl;
    }
     //转移构造函数
     Person(Person&& other): name(std::move(other.name)), age(other.age)
    {
         cout << "I have been moved"<<endl;
    }
};

int main()
{
    vector<Person> e;
    cout << "emplace_back:" <<endl;
    e.emplace_back("Jane", 23); //不用构造类对象

    vector<Person> p;
    cout << "push_back:"<<endl;
    p.push_back(Person("Mike",36));
    return 0;
}
//输出结果：
//emplace_back:
//I have been constructed
//push_back:
//I have been constructed
//I am being moved.
```

### 19、new和malloc之间有什么区别？new申请内存空间的时候为什么不需要类型转化，底层做了些什么事情呢？（new的作用:实例化一个对象。申请内存空间，然后调用相关的构造函数）【⭐】

（1）

- malloc和free是标准库函数，支持覆盖；new和delete是运算符，不重载。
- malloc仅仅分配内存空间，free仅仅回收空间，不具备调用构造函数和析构函数功能，用malloc分配空间存储类的对象存在风险；new和delete除了分配回收功能外，还会调用构造函数和析构函数。
- malloc和free返回的是void类型指针（必须进行类型转换），new和delete返回的是具体类型指针。

（2）

- new的三个步骤
  - 调用名为operator new标准库函数，分配足够大的原始的未类型化的内存，以保存指定类型的对象
  - 运行该类型的一个构造函数初始化对象
  - 返回指向新分配并构造函数对象的指针

- delete的两个步骤
  - 调用析构函数，回收对象中数据成员所申请的资源
  - 调用名为operator delete的标准库函数释放该对象所用的内存空间

### 20、[C++的多态如何实现 (虚函数实现原理)](https://www.cnblogs.com/bewolf/p/9352116.html)【⭐】

在基类的函数前加上**virtual**关键字，在派生类中重写该函数，运行时将会根据所指对象的实际类型来调用相应的函数，如果对象类型是派生类，就调用派生类的函数，如果对象类型是基类，就调用基类的函数。

```c++
#include <iostream>
using namespace std;

class Base{
public:
    virtual void fun(){
        cout << " Base::func()" <<endl;
    }
};

class Son1 : public Base{
public:
    virtual void fun() override{
        cout << " Son1::func()" <<endl;
    }
};

class Son2 : public Base{

};

int main()
{
    Base* base = new Son1;
    base->fun();
    base = new Son2;
    base->fun();
    delete base;
    base = NULL;
    return 0;
}
// 运行结果
// Son1::func()
// Base::func()
```

例子中，Base为基类，其中的函数为虚函数。**子类1继承并重写了基类的函数，子类2继承基类但没有重写基类的函数，从结果分析子类体现了多态性，那么为什么会出现多态性，其底层的原理是什么？这里需要引出虚表和虚基表指针的概念。**

虚表：虚函数表的缩写，**类中含有virtual关键字修饰的方法时，编译器会自动生成虚表**

- 虚函数表里存放的内容是什么时候写进去的？（编译阶段）

虚表指针：在含有**虚函数的类实例化对象**时，**对象地址的前四个字节存储的指向虚表的指针**

![image-20210913201652567](C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210913201652567.png)

**阐述实现多态的过程：**

**（1）**编译器在发现基类中**有虚函数时，会自动为每个含有虚函数的类生成一份虚表，该表是一个一维数组，虚表里保存了虚函数的入口地址**

**（2）**编译器会在每个对象的前四个字节中保存一个虚表指针，即**vptr**，**指向对象所属类的虚表**。在构造时，根据对象的类型去初始化虚指针vptr，从而让vptr指向正确的虚表，从而在调用虚函数时，能找到正确的函数

**（3）**所谓的合适时机，在派生类定义对象时，程序运行会自动调用构造函数，在构造函数中创建虚表并对虚表初始化。**在构造子类对象时，会先调用父类的构造函数，此时，编译器只“看到了”父类，并为父类对象初始化虚表指针，令它指向父类的虚表；当调用子类的构造函数时，为子类对象初始化虚表指针，令它指向子类的虚表**

**（4）当派生类对基类的虚函数没有重写时，派生类的虚表指针指向的是基类的虚表；当派生类对基类的虚函数重写时，派生类的虚表指针指向的是自身的虚表；当派生类中有自己的虚函数时，在自己的虚表中将此虚函数地址添加在后面**

这样指向派生类的基类指针在运行时，就可以根据派生类对虚函数重写情况动态的进行调用，从而实现多态性。

### 21、**虚函数和纯虚函数区别**？

- 虚函数是为了实现动态编联产生的，目的是通过基类类型的指针指向不同对象时，自动调用相应的、和基类同名的函数（**使用同一种调用形式，既能调用派生类又能调用基类的同名函数**）。虚函数需要在基类中加上virtual修饰符修饰，**因为virtual会被隐式继承，所以子类中相同函数都是虚函数**。当一个成员函数被声明为虚函数之后，其派生类中同名函数自动成为虚函数，**在派生类中重新定义此函数时要求函数名、返回值类型、参数个数和类型全部与基类函数相同**。
- **纯虚函数只是相当于一个接口名，但含有纯虚函数的类不能够实例化**。

纯虚函数首先是虚函数，其次它没有函数体，**取而代之的是用“=0”**。

既然是虚函数，它的函数指针会被存在虚函数表中，由于纯虚函数并没有具体的函数体，因此它在虚函数表中的值就为0，而具有函数体的虚函数则是函数的具体地址。

**一个类中如果有纯虚函数的话，称其为抽象类。抽象类不能用于实例化对象，否则会报错。**抽象类一般用于定义一些公有的方法。**子类继承抽象类也必须实现其中的纯虚函数才能实例化对象。**

```c++
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void fun1()
    {
        cout << "普通虚函数" << endl;
    }
    virtual void fun2() = 0;
    virtual ~Base() {}
};

class Son : public Base
{
public:
    virtual void fun2() 
    {
        cout << "子类实现的纯虚函数" << endl;
    }
};

int main()
{
    Base* b = new Son;
    b->fun1(); //普通虚函数
    b->fun2(); //子类实现的纯虚函数
    return 0;
}
```

### 22、为什么析构函数一般写成虚函数【⭐】

由于类的多态性，基类指针可以指向派生类的对象，**如果删除该基类的指针，就会调用该指针指向的派生类析构函数，而派生类的析构函数又自动调用基类的析构函数，这样整个派生类的对象完全被释放**。

如果析构函数不被声明成虚函数，则编译器实施静态绑定，在**删除基类指针时，只会调用基类的析构函数而不调用派生类析构函数，这样就会造成派生类对象析构不完全，造成内存泄漏**。

所以将**析构函数声明为虚函数是十分必要**的。在实现多态时，当用基类操作派生类，在析构时防止只析构基类而不析构派生类的状况发生，要将基类的析构函数声明为虚函数。

```c++
#include <iostream>
using namespace std;

class Parent{
public:
    Parent(){
        cout << "Parent construct function"  << endl;
    };
    virtual ~Parent(){
        cout << "Parent destructor function" <<endl;
    }
};

class Son : public Parent{
public:
    Son(){
        cout << "Son construct function"  << endl;
    };
    ~Son(){
        cout << "Son destructor function" <<endl;
    }
};

int main()
{
    Parent* p = new Son();
    delete p;
    p = NULL;
    return 0;
}
//运行结果：
//Parent construct function
//Son construct function
//Son destructor function
//Parent destructor function

Son* son = new Son;
delete son 会不会运行类Parent的析构函数？
    
析构函数不管是不是虚函数，delete某个对象son的时候，都会同时调用son类的析构和他一层一层往上的所有父类析构函数，这是析构函数本身的行为。而将析构函数声明为虚函数，只是调整这一切的起点。

```

### 23、[虚继承是怎么实现？](https://www.cnblogs.com/hunter-w/p/13845899.html)（菱形继承）【⭐】

```c++
#include <iostream>
using namespace std;

class A{}
class B : virtual public A{};
class C : virtual public A{};
class D : public B, public C{};

int main()
{
    cout << "sizeof(A)：" << sizeof A <<endl; // 1，空对象，只有一个占位
    cout << "sizeof(B)：" << sizeof B <<endl; // 4，一个bptr指针，省去占位,不需要对齐
    cout << "sizeof(C)：" << sizeof C <<endl; // 4，一个bptr指针，省去占位,不需要对齐
    cout << "sizeof(D)：" << sizeof D <<endl; // 8，两个bptr，省去占位,不需要对齐
}
```

上述代码所体现的关系是，B和C虚拟继承A，D又公有继承B和C，这种方式是一种**菱形继承或者钻石继承**

![image-20210913214526967](C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210913214526967.png)

**虚拟继承的情况下，无论基类被继承多少次，只会存在一个实体。**虚拟继承基类的子类中，子类会增加某种形式的指针，或者指向虚基类子对象，或者指向一个相关的表格；表格中存放的不是虚基类子对象的地址，就是其偏移量，此类指针被称为bptr，如上图所示。如果既存在vptr又存在bptr，某些编译器会将其优化，合并为一个指针。

### 24、模板和实现可不可以不写在一个文件里面？为什么？

1) 模板定义很特殊。**由template<…>处理的任何东西都意味着编译器在当时不为它分配存储空间，它一直处于等待状态直到被一个模板实例告知。**在编译器和连接器的某一处，有一机制能去掉指定模板的多重定义。**所以为了容易使用，几乎总是在头文件中放置全部的模板声明和定义。**

2) 在分离式编译的环境下，编译器编译某一个.cpp文件时并不知道另一个.cpp文件的存在，也不会去查找（当遇到未决符号时它会寄希望于连接器）。这种模式在没有模板的情况下运行良好，但遇到模板时就傻眼了，因为模板仅在需要的时候才会实例化出来。所以，**当编译器只看到模板的声明时，它不能实例化该模板，只能创建一个具有外部连接的符号并期待连接器能够将符号的地址决议出来。然而当实现该模板的.cpp文件中没有用到模板的实例时，编译器懒得去实例化**，所以，整个工程的.obj中就找不到一行模板实例的二进制代码，于是连接器也黔驴技穷了。

### 25、左值引用、右值引用、移动语义【⭐】

C++11正是通过引入右值引用来优化性能，具体来说是通过移动语义来避免无谓拷贝的问题，通过move语义来将临时生成的左值中的资源无代价的转移到另外一个对象中去

1) 在C++11中所有的值必属于左值、右值两者之一，**右值又可以细分为纯右值、将亡值**。在C++11中**可以取地址的、有名字的就是左值，反之，不能取地址的、没有名字的就是右值（将亡值或纯右值）**。举个例子，int a = b+c, a 就是左值，其有变量名为a，通过&a可以获取该变量的地址；表达式b+c、函数int func()的返回值是右值，在其被赋值给某一变量前，我们不能通过变量名找到它，＆(b+c)这样的操作则不会通过编译。

2) C++11对C++98中的右值进行了扩充。在C++11中右值又分为纯右值（prvalue，Pure Rvalue）和将亡值（xvalue，eXpiring Value）。其中纯右值的概念等同于我们在C++98标准中右值的概念，指的是临时变量和不跟对象关联的字面量值；将亡值则是C++11新增的跟右值引用相关的表达式，这样表达式通常是将要被移动的对象（移为他用），比如返回右值引用T&&的函数返回值、std::move的返回值，或者转换为T&&的类型转换函数的返回值。将**亡值可以理解为通过“盗取”其他变量内存空间的方式获取到的值**。在**确保其他变量不再被使用、或即将被销毁时，通过“盗取”的方式可以避免内存空间的释放和分配，能够延长变量值的生命期**。

3) 左值引用就是对一个左值进行引用的类型。右值引用就是对一个右值进行引用的类型，事实上，由于右值通常不具有名字，我们也只能通过引用的方式找到它的存在。右值引用和左值引用都是属于引用类型。无论是声明一个左值引用还是右值引用，都必须立即进行初始化。而其原因可以理解为是引用类型本身自己并不拥有所绑定对象的内存，只是该对象的一个别名。左值引用是具名变量值的别名，而右值引用则是不具名（匿名）变量的别名。**左值引用通常也不能绑定到右值，但常量左值引用是个“万能”的引用类型**。它可以接受非常量左值、常量左值、右值对其进行初始化。不过常量左值所引用的右值在它的“余生”中只能是只读的。相对地，非常量左值只能接受非常量左值对其进行初始化。

4) **右值值引用通常不能绑定到任何的左值，要想绑定一个左值到右值引用，通常需要std::move()将左值强制转换为右值。**

- 总结：

  - 左值引用：传统的C++中引用被称为左值引用

  - 右值引用：C++11中增加了右值引用，**右值引用关联到右值时，右值被存储到特定位置，右值引用指向该特定位置，也就是说，右值虽然无法获取地址，但是右值引用是可以获取地址的，该地址表示临时对象的存储位置**

  - **右值引用的特点**

    - 特点1：**通过右值引用的声明，右值又“重获新生”，其生命周期与右值引用类型变量的生命周期一样长，只要该变量还活着，该右值临时量将会一直存活下去**
    - 特点2：右值引用独立于左值和右值。意思是**右值引用类型的变量可能是左值也可能是右值**
    - 特点3：T&& t在发生自动类型推断的时候，它是左值还是右值取决于它的初始化。


### 26、基类与父类的构造与析构函数的执行顺序【⭐】

派生类对象在实例化时，派生类构造函数先被执行，在执行过程中（在实例化派生类成员前），调用基类构造函数，然后（在基类成员实例化后）返回派生类构造函数实例化派生类成员。

### 27、c++重载与重写的区别【⭐】

- **重载overload：**在**同一个类中，函数名相同，参数列表不同**，编译器会根据这些函数的不同参数列表，将同名的函数名称做修饰，从而生成一些不同名称的预处理函数，**未体现多态**。

 

- **重写override：**也叫**覆盖**，**子类重新定义父类中有相同名称相同参数的虚函数，主要是在继承关系中出现的，被重写的函数必须是virtual的**，重写函数的访问修饰符可以不同，尽管virtual是private的，子类中重写函数改为public,protected也可以，**体现了多态**。

 

- **重定义redefining：**也叫**隐藏**，**子类重新定义父类中有相同名称的非虚函数**，**参数列表可以相同可以不同，会覆盖其父类的方法，未体现多态。**
  - a、如果**派生类的函数和基类的函数同名**，但是**参数不同**，此时，不管有无virtual，**基类的函数被隐藏**。
  - b、如果**派生类的函数与基类的函数同名**，并且**参数也相同**，但是**基类函数没有vitual关键字**，此时，**基类的函数被隐藏**。（如果有virtual就成重写了）

```c++
#include <iostream>  
using namespace std;  
class Base  
{  
private:  
    virtual void display() { cout<<"Base display()"<<endl; }  
    void show(){ cout<<"Base show()"<<endl; }  
public:  
    void exec(){ display(); show(); }  
    void fun(string s) { cout<<"Base fun(string)"<<endl; }  
    void fun(int a) { cout<<"Base fun(int)"<<endl; }//overload:两个fun函数在Base类的内部被重载  
    virtual int function(){}  
};  
class ClassA:public Base  
{  
public:  
    void display() { cout<<"ClassA display()"<<endl; }//override:基类中display为虚函数，且参数列表一致，故此处为重写  
    void fun(int a,int b) { cout<<"ClassA fun(int,int)"<<endl; }//redefining:fun函数在Base类中不为虚函数，故此处为重定义  
    void show() { cout<<"ClassA show()"<<endl; }//redefining:理由同上  
    int function(int a){}//overload:注意这是重载而不是重写，因为参数列表不同，在编译时ClassA中其实还有个编译器自己偷偷加上的从Base继承来的int function(){};  
};  
int main(){  
    ClassA a;  
    Base *base=&a;  
    base->exec();//display()是ClassA的，因为覆盖了，show()是Base自己的  
    a.exec();//结果同上  
    a.show();//show()是ClassA重定义的  
    base->fun(1);//fun()是Base自己的，因为直接从对象base调用  
    a.fun(1, 1);//fun()是ClassA重定义的  
    return 0;  
}  
```

### 28、public，private，protected的区别，继承方法与访问权限【⭐】

访问范围：

private: 只能由该类中的函数、其友元函数访问,不能被任何其他访问，该类的对象也不能访问. 
protected: 可以被该类中的函数、子类的函数、以及其友元函数访问,但不能被该类的对象访问 
public: 可以被该类中的函数、子类的函数、其友元函数访问,也可以由该类的对象访问

注：友元函数包括两种：设为友元的全局函数，设为友元类中的成员函数

父类与其直接子类的访问关系如上，无论是哪种继承方式（private继承、protected继承、public继承）。

对于三种继承关系的不同：

public继承：public继承后，从父类继承来的函数属性不变（private、public、protected属性不变，）。

private继承：private继承后，从父类继承来的函数属性都变为private

protected继承：protected继承后，从父类继承过来的函数，public、protected属性变为protected，private还是private。

### 29、能不能在构造函数和析构函数中调用虚函数？【⭐】

```c++
可以，但是达不到想要的效果，应该尽可能避免在构造函数和析构函数中调用虚函数。
    
class base{
public:
    base(){
        cout<<The size is size()<<endl; 
   }
private:
    virtual size_t size(){
        return sizeof(*this);
    }
};
 
class derived : public class base{
public:
    derived(){
        cout<<The size of is size()<<endl;
    }
private:
    size_t size(){
        return sizeof(*this);
    }
```

当定义一个derived实例对象时，在**base的构造函数中调用size()会被静态的决议为base::size()而不是derived::size()**。

可以这么理解，**当在构造base部分时，derived并不一个完整的实例对象，derived部分的成员变量甚至没有被初始化**，如果在构造base期间调用的是derived的虚函数并且该虚函数引用了尚未构造好的成员变量，试想会发生什么。所以，从安全性考虑，经由构造中的对象来调用一个虚函数，其函数就是正在构造的对象的所属函数。

同时析构函数也如此。当正在析构base部分时，derived部分已经被析构完毕，成员变量已经无效，调用的虚函数是所属base的。


# STL

### 基本概念六大组件

- 容器 装数据的，数据结构
  - 序列式容器
  - 关联式容器
  - 无序关联式容器
- 迭代器  广义的指针
- 适配器  
- 算法   操作数据
- 函数对象 （仿函数）
- 空间配置器 allocator  空间的申请与释放

### 1、vector删除其中的元素，迭代器如何变化？为什么是两倍扩容？释放空间？【⭐】

（1）erase迭代器不仅使所指向被删除的迭代器失效，而且使被删元素之后的所有迭代器失效(list除外)，**所以不能使用erase(it++)的方式，但是erase的返回值是下一个有效迭代器**；

```c++
for (auto it = c.begin(); it != c.end())//顺序容器使用erase的正确方式
{
    if (*it == 6)
    {
        it = c.erase(it);//从顺序容器中删除非头尾的元素
        //删除it指向的元素，之后元素向前移动，it不动
    }
    else
    {
        ++it;
    }
}
```

（2）为什么是两倍扩容？https://blog.csdn.net/yangshiziping/article/details/52550291

- 不同的编译器，vector有不同的扩容大小。在vs下是1.5倍，**在GCC下是2倍**；

- 空间和时间的权衡。简单来说， **空间分配的多，平摊时间复杂度低，但浪费空间也多**。

- 使用k=2增长因子的问题在于，每次扩展的新尺寸必然刚好大于之前分配的总和，也就是说，之前分配的内存空间不可能被使用。这样对内存不友好。**最好把增长因子设为(1,2)**

- **对比可以发现采用成倍方式扩容，可以保证常数的时间复杂度，而增加指定大小的容量只能达到O(n)的时间复杂度，因此，使用成倍的方式扩容。**

（3）

- 由于**vector的内存占用空间只增不减**，比如你首先分配了10,000个字节，然后erase掉后面9,999个，留下一个有效元素，但是内存占用仍为10,000个。**所有内存空间是在vector析构时候才能被系统回收**。**empty()用来检测容器是否为空的，clear()可以清空所有元素。但是即使clear()，vector所占用的内存空间依然如故，无法保证内存的回收。**

-  **如果需要空间动态缩小，可以考虑使用deque**。如果vector，**可以用swap()来帮助你释放内存**。

```c++
vector(Vec).swap(Vec); //将Vec的内存清除； 
vector().swap(Vec); //清空Vec的内存；
```

### 2、[vector与list区别](https://blog.csdn.net/Lucien_zhou/article/details/39973165)

- vector的随机访问效率高，但在插入和删除时（不包括尾部）需要挪动数据，不易操作。
- list的访问要遍历整个链表，它的随机访问效率低。但对数据的插入和删除操作等都比较方便，改变指针的指向即可。
- vector中的迭代器在使用后就失效了，而list的迭代器在使用之后还可以继续使用。

### 3、[map、set是怎么实现的，红黑树是怎么能够同时实现这两种容器？ 为什么使用红黑树](https://blog.csdn.net/weixin_32569511/article/details/113717303)？

1) 他们的底层都是以红黑树的结构实现，因此插入删除等操作都在 O(logn）时间内完成，因此可以完成高效的插入删除；
2) 底层是红黑树，实现map的红黑树的节点数据类型是key+value，而实现set的节点数据类型是value
3) 因为map和set要求是自动排序的，红黑树能够实现这一功能，而且时间复杂度比较低。

### 4、vector、list、deque的选择

使用原则：
（1）如果需要**高效的随机存取**，而**不在乎插入和删除的效率**，使用vector；
（2）如果需要**大量高效的删除插入**，而**不在乎存取时间**，则使用list；
（3）如果需要**高效的随机存取**，还要**大量的首尾的插入删除**则建议使用deque，它是list和vector的折中；

### 5、string的三种实现方式【⭐】（特别是COW）

虽然历史上的实现有多种，但基本上有三种方式：

- 浅拷贝（不可用）

```c++
String::String(const String &s) 
    : _str(s._str)
    , _size(s._size)
    , _capacity(s._capacity)
{}

String & String::operator=(const String &s)  //浅拷贝
{
    if (_str != s._str)
    {
        delete[] _str;
        _str = s._str;
        _size = s._size;
        _capacity = s._capacity;
    }
    return *this;
}
/*
虽然这种方法非常简单，但这种方法存在很大的漏洞，这种方法拷贝构造以及赋值运算符的重载后的对象与被拷贝的对象指向同一块空间，如果使用其中一个对象对其中的值进行修改时会导致另一个对象中的值也会发生改变，所以一般情况下不会使用浅拷贝这种拷贝方式。
*/
```

- Eager Copy(深拷贝)

```c++
class String
{
public:
    String(const char* str = "");
    String(const String& s);
    String& operator=(const String& s);
    ~String();
private:
    char* _str;
};

//传统写法完成String深拷贝
//String::String(const char* str) //（有参构造）
//  : _str(new char[strlen(str) + 1])
//{
//  strcpy(_str, str);
//}
//
//String::String(const String& s) // 必须用引用否则会无限的递归（拷贝构造）
//  : _str(new char[strlen(s._str) + 1]) // 开辟s._str大小的空间，+1是因为‘/0’
//{
//  strcpy(_str, s._str);
//}
//
//String& String::operator=(const String& s)
//{
//  if (this != &s) // 判断是否相同
//  {
//      if(_str)
//          delete[] _str; // _str不为空，则需要先释放空间
//      _str = new char[strlen(s._str) + 1];
//      strcpy(_str, s._str);
//  }
//  return *this;
//}
//
//String::~String()
//{
//  if (NULL != _str)
//  {
//      delete[] _str;
//  }
//}

//现代写法完成String深拷贝
String::String(const char* str)
    : _str(new char[strlen(str) + 1])
{
    strcpy(_str, str);
}

String::String(const String& s)
{
    String tmp(s._str); // 用s._str定义一个tmp对象
    swap(_str, tmp._str); // 将指针进行交换【⭐】
}

String& String::operator=(String s) // 实参与形参结合时，会调用拷贝构造函数，再调用构造函数
{
    swap(_str, s._str); // 将指针进行交换【⭐】
    return *this;
}

String::~String()
{
    if (NULL != _str)
    {
        delete[] _str;
    }
}
/*
相比较两种方法而言，更推荐现代写法，原因如下:

现代写法代码量相对而言较少，效率更高
现代写法的赋值运算符重载中调用了拷贝构造、拷贝构造中调用了构造函数，故现代写法中的复用性更强，更易于管理
【⭐】传统写法中的赋值运算符重载会先释放原先开辟的空间，这样如果下面重新开辟空间失败不能拷贝时，那么原有数据不会被保留

深拷贝虽然会再次开辟空间增加内存的占用，但这样可以解决浅拷贝不能做到的问题。
*/
```

- COW（Copy-On-Write 写时复制）【⭐】

写时拷贝较上面方法更为复杂，所以在这里先进行说明：写时拷贝需要一个**引用计数**，因此要**增加一个存放一个引用计数的位置**，我们将这个位置**放在_str前面的四个字节**，每当**拷贝一次就将引用计数++一次**，进行**修改时先判断一下引用计数是否为1，若为1则直接进行修改**；**否则先重新开辟一块空间，将原有引用计数–，再进行修改**。
<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918104139635.png" alt="image-20210918104139635" style="zoom:67%;" />

```c++
class String
{
public:
    String() //无参
    : _pstr(new char[5]() + 4) // 多开辟4个字节来存放引用计数，还有将指针向后偏移四个字节（指向字符串）【⭐】
    {
        cout << "String()" << endl;
        initRefcount();
    }

    //String s1("hello")
    String(const char *pstr)
    : _pstr(new char[strlen(pstr) + 5]() + 4) // 多开辟4个字节来存放引用计数，还有将指针向后偏移四个字节（指向字符串）【⭐】
    {
        cout << "String(const char *)" << endl;
        strcpy(_pstr, pstr);
        initRefcount();
    }

    //String s2 = s1
    String(const String &rhs) //拷贝构造
    : _pstr(rhs._pstr) //浅拷贝
    {
        cout << "String(const String &)" << endl;
        increaseRefcount(); //引用计数加一
    }

    //String s3 = s1
    //s3 = s2
    //
    //String s3("world")
    String &operator=(const String &rhs) //赋值
    {
        cout << "String &operator=(const String &)" << endl;
        if(this != &rhs)//1、自复制
        {
            release();//2、释放左操作数

            _pstr = rhs._pstr;//3、浅拷贝
            increaseRefcount();
        }

        return *this;//4、返回*this
    }

    //s3 = s1;
    //s3[0] = 'H';
    char &operator[](size_t idx) //用下标获取字符或修改
    {
        if(idx < size())
        {
            if(getRefcount() > 1)//共享时，获取下标字符先拷贝一份
            {
                char *ptmp = new char[size() + 5]() + 4;
                strcpy(ptmp, _pstr);
                dereaseRefcount();
                _pstr = ptmp;
                initRefcount();
            }
            return _pstr[idx];
        }
        else //非法下标，返回'\0'
        {
            static char nullchar = '\0';
            return nullchar;
        }
    }

    //s3 = s1;
    ~String()
    {
        cout << "~String()" << endl;
        release();
    }

    void release()
    {
        dereaseRefcount(); //先引用计数减1
        if(0 == getRefcount()) //若引用计数为0，说明此字符串内存空间没有共享，需要释放
        {
            delete [] (_pstr - 4);//整个空间释放
        }

    }

    size_t size()
    {
        return strlen(_pstr);
    }

    void initRefcount() //引用计数初始化
    {
        *(int *)(_pstr - 4) = 1; 
    	//先指向引用计数，强转为int类型指针，再解引用（四个字节）使其为1【⭐】
    }

    void increaseRefcount()
    {
        ++*(int *)(_pstr - 4); //引用计数加一
    }

    void dereaseRefcount()
    {
        --*(int *)(_pstr - 4); //引用计数减一
    }

    int getRefcount()
    {
        return *(int *)(_pstr - 4); //获取引用计数
    }

    const char *c_str() const
    {
        return _pstr; //获取字符串
    }

    friend std::ostream &operator<<(std::ostream &os, const String &rhs); //重载
private:
    char *_pstr;
};

std::ostream &operator<<(std::ostream &os, const String &rhs)
{
    if(rhs._pstr)
    {
        os << rhs._pstr;
    }

    return os;
}
```

- SSO(Short String Optimization-短字符串优化)

通常来说，一个程序里用到的字符串大部分都很短小，而在64位机器上，一个char*指针就占用了8个字
节，所以SSO就出现了，其核心思想是：发生拷贝时要复制一个指针，对小字符串来说，为啥不直接复
制整个字符串呢，说不定还没有复制一个指针的代价大。  

<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918112405107.png" alt="image-20210918112405107" style="zoom:50%;" />

```c++
class string
{
	union Buffer
	{
		char * _pointer;
		char _local[16];
	};
	Buffer _buffer;
	size_t _size;
	size_t _capacity;
};
```

当字符串的**长度小于等于15个字节**时，**buffer直接存放整个字符串**；当字符串**大于15个字节时**，**buffer**
**存放的就是一个指针，指向堆空间的区域**。这样做的好处是，当字符串较小时，直接拷贝字符串，放在
string内部，不用获取堆空间，开销小。  

- 最佳策略

以上三种方式，都不能解决所有可能遇到的字符串的情况，各有所长，又各有缺陷。综合考虑所有情况
之后，**facebook开源的folly库中**，**实现了一个fbstring**, 它根据字符串的不同长度使用不同的拷贝策略，
最终每个fbstring对象占据的空间大小都是24字节。

1. 很短的（0~22）字符串用SSO，23字节表示字符串（包括'\0'）,1字节表示长度
2. 中等长度的（23~255）字符串用eager copy，8字节字符串指针，8字节size，8字节capacity.
3. 很长的(大于255)字符串用COW, 8字节指针（字符串和引用计数），8字节size，8字节capacity.  

- 线程安全性  

两个线程同时对同一个字符串进行操作的话, 是不可能线程安全的, 出于性能考虑, C++并没有为string实
现线程安全, 毕竟不是所有程序都要用到多线程。

但是两个线程同时对独立的两个string操作时, 必须是安全的. **COW技术实现这一点是通过原子的对引用**
**计数进行+1或-1操作。**

CPU的**原子操作**虽然比mutex锁好多了, 但是仍然会带来性能损失, 原因如下:

- 阻止了CPU的乱性执行.
- 两个CPU对同一个地址进行原子操作, 会导致cache失效, 从而重新从内存中读数据.
- 系统通常会lock住比目标地址更大的一片区域，影响逻辑上不相关的地址访问

这也是**在多核时代，各大编译器厂商都选择了SS0实现**的原因吧。  

### 6、vector的底层实现【⭐】

<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918143305691.png" alt="image-20210918143305691" style="zoom:50%;" />

<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918143330597.png" alt="image-20210918143330597" style="zoom:50%;" />

- vector的迭代器失效【⭐】

  - insert的时候有可能会扩容（**指针指向已经不受控制的区域**），为防止报错，可以每次都将 it = nums.begin() 进行重新置为 =》 所以**不推荐在vector中间进行insert操作**，在头insert一个元素可以但不为O1
  - erase删除的时候元素会向前挪动，**使用迭代器进行删除时要注意**，不要再++it，会错过一个元素
- **clear & shrink_to_fit**
  - clear : size=0, capacity还是原来的
  - shrink_to_fit : 将capacity缩小到size大小

### 7、deque底层实现

<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918152945937.png" alt="image-20210918152945937" style="zoom:50%;" />

<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918153029863.png" alt="image-20210918153029863" style="zoom:50%;" />

- 小片段内部连续，片段之间分散
- 所以在**deque使用insert插入到中间位置时时间复杂度非常高，不要使用**

### 8、set、map、及multiset、multimap【底层都为红黑树】【红黑树的特点⭐】

- 只有map支持下标访问【⭐】
  - 存在key，返回value
  - key不存在，插入key，返回空value
  - 修改key下的value值

<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918161015451.png" alt="image-20210918161015451" style="zoom: 67%;" />

- 红黑树的操作时间跟二叉查找树的时间复杂度是一样的，执行**查找、插入、删除等操作的时间复杂度为O（logn）**。
- 红黑树是特殊的AVL树，遵循红定理和黑定理 
  - 红定理：不能有两个相连的红节点 
  - 黑定理：根节点必须是黑节点，而且所有节点通向NULL的路径上，所经过的黑节点的个数必须相等

- **关联式容器**尽量**使用其自身提供的find()函数查找指定的元素，效率更高**，因为STL提供的find()函数是一种顺序搜索算法。

### 9、unordered_set、unordered_map【底层哈希表】

<img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918162132478.png" alt="image-20210918162132478" style="zoom:67%;" />



- hashtable中的bucket所维护的list既不是list也不是slist，而是其自己定义的由hashtable_node数据结构组成的linked-list，而bucket聚合体本身使用vector进行存储。hashtable的迭代器只提供前进操作，不提供后退操作

- 在hashtable设计bucket的数量上，其内置了28个质数[53, 97, 193,...,429496729]，在创建hashtable时，会根据存入的元素个数选择大于等于元素个数的质数作为hashtable的容量（vector的长度），其中每个bucket所维护的linked-list长度也等于hashtable的容量。如果插入hashtable的元素个数超过了bucket的容量，就要进行重建table操作，即找出下一个质数，创建新的buckets vector，重新计算元素在新hashtable的位置。

- **O1的查找时间**【⭐】

### 10、容器适配器（stack、queue、priority_queue）

- 不可用迭代器的方式进行遍历

- **优先级队列底层：堆排序**【⭐】

  - heap（堆）并不是STL的容器组件，是priority queue（优先队列）的底层实现机制，因为binary max heap（大根堆）总是最大值位于堆的根部，优先级最高。binary heap本质是一种complete binary tree（完全二叉树）

  <img src="C:\Users\10690\AppData\Roaming\Typora\typora-user-images\image-20210918164055569.png" alt="image-20210918164055569" style="zoom:67%;" />

  - STL中heap的实现
    - 四种算法：
      - push_heap（插入新节点到末尾【完全二叉树最后】，接着向上建堆）
      - pop_heap（删除根节点【与末尾节点替换】，接着向下建堆）
      - sort（堆排序）
      - make_heap(建堆)

### 11、迭代器（类似于指针）

- 随机访问-》双向 -》 前向 
- -》输出迭代器 
- -》输入迭代器
- 输出流迭代器(ostream_iterator) + (结合copy算法)

```c++
ostream_type* _M_stream;
const _CharT* _M_string;

//osi(cout, "\n")
__s = &cout;
__c = "\n";
ostream_iterator(ostream_type& __s, const _CharT* __c) 
: _M_stream(&__s), _M_string(__c) 
{
    
}

template <class _InputIter, class _OutputIter>
inline _OutputIter copy(_InputIter __first, _InputIter __last,
                        _OutputIter __result) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  return __copy_aux(__first, __last, __result, __VALUE_TYPE(__first));
}

template <class _InputIter, class _OutputIter, class _Tp>
inline _OutputIter __copy_aux(_InputIter __first, _InputIter __last,
                              _OutputIter __result, _Tp*) {
  typedef typename __type_traits<_Tp>::has_trivial_assignment_operator
          _Trivial;
  return __copy_aux2(__first, __last, __result, _Trivial());
}

template <class _InputIter, class _OutputIter>
inline _OutputIter __copy_aux2(_InputIter __first, _InputIter __last,
                               _OutputIter __result, __false_type) {
  return __copy(__first, __last, __result,
                __ITERATOR_CATEGORY(__first),
                __DISTANCE_TYPE(__first));
}

__first = number.begin();
__last = number.end();
__result = osi
template <class _InputIter, class _OutputIter, class _Distance>
inline _OutputIter __copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result,
                          input_iterator_tag, _Distance*)
{
  for ( ; __first != __last; ++__result, ++__first)
    *__result = *__first;
  return __result;
}
       f                l
1  3   6   8   9   2
    
    
*__result = 3
*osi = 3
osi = 3
    
 ostream_iterator<_Tp>& operator=(const _Tp& __value)
{ 
    *_M_stream << __value;//cout << 3
    if (_M_string) *_M_stream << _M_string;//cout << "\n" 
    return *this;
  }

//cout << 1
//cout << "\n" 
//cout << 3
//cout << "\n" 
```

- 输入流迭代器(istream_iterator) + (结合copy算法)

```c++
istream_type* _M_stream;
 _Tp _M_value;
bool _M_ok;

//isi(std::cin)
__s = cin;
istream_iterator(istream_type& __s) 
: _M_stream(&__s) 
{ 
    _M_read();
}

 void _M_read() {
    _M_ok = (_M_stream && *_M_stream) ? true : false;
    if (_M_ok) {
      *_M_stream >> _M_value;//cin >>  _M_value  _M_value = 2
      _M_ok = *_M_stream ? true : false;   // _M_ok = 1
    }
  }
__first = isi
__last =istream_iterator<int>()
__result = back_inserter(number)
template <class _InputIter, class _OutputIter, class _Distance>
inline _OutputIter __copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result,
                          input_iterator_tag, _Distance*)
{
  for ( ; __first != __last; ++__result, ++__first)
    *__result = *__first;
  return __result;
}

*__result = __M_value

    istream_iterator& operator++() { 
    _M_read(); 
    return *this;
  }
    
back_insert_iterator<_Container>&
  operator=(const typename _Container::value_type& __value) { 
    container->push_back(__value);
    return *this;
  }


    
template <class _Container>
inline back_insert_iterator<_Container> back_inserter(_Container& __x) {
  return back_insert_iterator<_Container>(__x);
}

back_insert_iterator(_Container& __x) 
: container(&__x) 
{
    
}

back_insert_iterator<_Container>& operator*() { return *this; }
back_insert_iterator<_Container>& operator++() { return *this; }
back_insert_iterator<_Container>& operator++(int) { return *this; }
```

### 12、remove_if算法

remove_if为何不直接将元素删掉呢？？？

为了设计成通用型算法，适应vector、list的特点

### 13、空间配置器（allocator）=》（运用在vector中）

```c++
//vector代码实现
#include <iostream>
#include <memory>

using std::cout;
using std::endl;

template <class T>
class Vector
{
public:
    /* typedef T * iterator; */
    /* typedef T * iterator; //C语言中的重定义*/
    using iterator = T *;//C++11的重定义方式

    Vector()
    : _start(nullptr)
    , _finish(nullptr)
    , _end_of_storage(nullptr)
    {
    }

    ~Vector();
    void push_back(const T &t);
    void pop_back();

    iterator begin()
    {
        return _start;
    }

    iterator end()
    {
        return _finish;
    }

    int size() const
    {
        return _finish - _start;

    }

    int capacity() const
    {
        return _end_of_storage - _start;

    }
private:
    void reallocator();
private:
    static std::allocator<T> _alloc;//静态数据成员，放在类外初始化
    T *_start;
    T *_finish;
    T *_end_of_storage;

};

template <class T>  
std::allocator<T> Vector<T>::_alloc;

template<class T>
Vector<T>::~Vector()
{
    if(_start)
    {
        while(_finish != _start)
        {
            _alloc.destroy(--_finish);//对象的销毁

        }
        _alloc.deallocate(_start, capacity());//空间的释放

    }

}

template <class T>
void Vector<T>::push_back(const T &t)
{
    if(size() == capacity())
    {
        reallocator();//当size() == capacity(),需要扩容
    }
    if(size() < capacity())
    {
        _alloc.construct(_finish++, t);

    }

}

template <class T>
void Vector<T>::pop_back()
{
    if(size() > 0)
    {
        _alloc.destroy(--_finish);
    }

}

template <class T>
void Vector<T>::reallocator()
{
    int oldCapacity = capacity();
    int newCapacity = 2 * oldCapacity > 0 ? 2 * oldCapacity : 1;

    T *ptmp = _alloc.allocate(newCapacity);//开辟空间，该空间上是没有对象
    if(_start)
    {
        /* copy(_start, _finish, ptmp); //copy算法中需要对象进行解引用，对象进行赋值*/
        std::uninitialized_copy(_start, _finish, ptmp);
        while(_finish != _start)
        {
            _alloc.destroy(--_finish);//对象的销毁

        }
        _alloc.deallocate(_start, capacity());//空间的释放

    }
    //三个指针，需要重新设置
    _start = ptmp;
    _finish = _start + oldCapacity;
    _end_of_storage = _start + newCapacity;

}

template <class Container>
void display(const Container &vec)
{
    cout << "vec.size() = " <<vec.size() << endl
         << "vec.capacity() = " << vec.capacity() << endl;

}

void test()
{
    Vector<int> numbers;
    display(numbers);

    cout << endl;
    numbers.push_back(1);
    display(numbers);

    cout << endl;
    numbers.push_back(2);
    display(numbers);

    cout << endl;
    numbers.push_back(3);
    display(numbers);

    cout << endl;
    numbers.push_back(4);
    display(numbers);

    cout << endl;
    numbers.push_back(5);
    display(numbers);

    cout << endl;
    for(auto &elem : numbers)
    {
        cout << elem << " ";

    }
    cout<< endl;;

}
int main(int argc, char **argv)
{
    test();
    return 0;
}
```

```c++
//空间的申请
_Tp* allocate(size_type __n, const void* = 0) 
{
    return __n != 0 ? static_cast<_Tp*>(_Alloc::allocate(__n * sizeof(_Tp))) : 0;
  }

  // __p is not permitted to be a null pointer.
  void deallocate(pointer __p, size_type __n)
    { _Alloc::deallocate(__p, __n * sizeof(_Tp)); }



//对象的构建
void construct(pointer __p, const _Tp& __val) 
{ 
    new(__p) _Tp(__val); //定位new表达式，指定的空间上构建对象
}

//对象的销毁
void destroy(pointer __p) { __p->~_Tp(); }



//STL中空间配置器的源码解读

typedef alloc _Alloc;

# ifdef __USE_MALLOC  //一级空间配置器

typedef malloc_alloc alloc;

typedef __malloc_alloc_template<0> malloc_alloc;

template <int __inst>
class __malloc_alloc_template 
{
public:
	static void* allocate(size_t __n)
	{
		void* __result = malloc(__n);
		if (0 == __result) 
			__result = _S_oom_malloc(__n);

      return __result;
	}
	
	static void deallocate(void* __p, size_t /* __n */)
	{
		free(__p);
	}
};


# else  //二级空间配置器
	
typedef __default_alloc_template<__NODE_ALLOCATOR_THREADS, 0> alloc;
template <bool threads, int inst>
class __default_alloc_template
{

public:
	static void* allocate(size_t __n)
	{
		if(n > 128)
		{
			malloc(n)
		}
		else
		{
			//内存池 + 16个自由链表
			_Obj* __STL_VOLATILE* __my_free_list
          = _S_free_list + _S_freelist_index(__n);//_S_free_list[3]
		  
			_Obj* __RESTRICT __result = *__my_free_list;
          if (__result == 0)
        __ret = _S_refill(_S_round_up(__n));
          else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
		}
	   }
	}
};

union _Obj
{
	 union _Obj* _M_free_list_link;
	 char _M_client_data[1];    /* The client sees this.        */		
};

template <bool __threads, int __inst>
typename __default_alloc_template<__threads, __inst>::_Obj* __STL_VOLATILE
__default_alloc_template<__threads, __inst> ::_S_free_list[
# if defined(__SUNPRO_CC) || defined(__GNUC__) || defined(__HP_aCC)
    _NFREELISTS
# else
    __default_alloc_template<__threads, __inst>::_NFREELISTS
# endif
] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, };



__bytes = 32
static  size_t _S_freelist_index(size_t __bytes)
{
        return (((__bytes) + (size_t)_ALIGN-1)/(size_t)_ALIGN - 1);
		//return   (32 + 7)/8 - 1
}
(32 + 7)/8 - 1 = 39/8 - 1 = 3
  
  
template <bool __threads, int __inst>
char* __default_alloc_template<__threads, __inst>::_S_start_free = 0;

template <bool __threads, int __inst>
char* __default_alloc_template<__threads, __inst>::_S_end_free = 0;

template <bool __threads, int __inst>
size_t __default_alloc_template<__threads, __inst>::_S_heap_size = 0;




 volative   mutable
 
 


//向上取整，得到8的倍数
 _S_round_up(size_t __bytes) //32   25 --- 32
    { return (((__bytes) + (size_t) _ALIGN-1) & ~((size_t) _ALIGN - 1)); }
	
	(32 + 7) & ~ (8 - 1) = 39 & ~ 7
	39 = 32 + 4 + 2 + 1 = 0010 0111
	7 = 4 + 2 + 1 = 0000 0111
	
	0010 0111
&	1111 1000
    0010 0000
	
    0010 0000
&	1111 1000
    0010 0000
	
	
    0001 1111
&	1111 1000
    0001 1000
	
//解读方法
1、先申请32字节

__size = 32
__nobjs = 20
char*
__default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
	char* __result;
    size_t __total_bytes = __size * __nobjs = 32 * 20 = 640;
    size_t __bytes_left = _S_end_free - _S_start_free = 0;
	
	else {
        size_t __bytes_to_get = 
	  2 * __total_bytes + _S_round_up(_S_heap_size >> 4)
	   = 2 * 640 + 0= 1280 ;
	   
	   _S_start_free = (char*)malloc(__bytes_to_get) = malloc(1280);
	   
	    _S_heap_size += __bytes_to_get = 1280;
        _S_end_free = _S_start_free + __bytes_to_get;
        return(_S_chunk_alloc(__size, __nobjs));//递归调用
	}
	
	//递归调用
	char* __result;
    size_t __total_bytes = __size * __nobjs = 32 * 20 = 640;
    size_t __bytes_left = _S_end_free - _S_start_free = 1280;
	
	if (__bytes_left >= __total_bytes) {
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }

}

void* __default_alloc_template::_S_refill(size_t __n)//__n = 32
{
    int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs);//32 20
    _Obj* __STL_VOLATILE* __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	
	__my_free_list = _S_free_list + _S_freelist_index(__n);
	//_S_free_list[3]

}


static void* allocate(size_t __n)//n = 32
{
	else{
		 _Obj* __STL_VOLATILE* __my_free_list
          = _S_free_list + _S_freelist_index(__n);//_S_free_list[3]
     
      _Obj* __RESTRICT __result = *__my_free_list;
      if (__result == 0)
        __ret = _S_refill(_S_round_up(__n));
      else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
      }
		
	}
}
2、在申请64字节
__size = 64
__nobjs = 20

char*
__default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
    char* __result;
    size_t __total_bytes = __size * __nobjs = 1280;
    size_t __bytes_left = _S_end_free - _S_start_free = 640;
	
	else if (__bytes_left >= __size) {
        __nobjs = (int)(__bytes_left/__size) = 10;
        __total_bytes = __size * __nobjs = 64 * 10 = 640;
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }

}



void*
__default_alloc_template::_S_refill(size_t __n)//__n = 64
{
    int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs);
    _Obj* __STL_VOLATILE* __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	//_S_free_list[7]
	__my_free_list = _S_free_list + _S_freelist_index(__n);
	//640字节切成10份挂在64下面

}




 static void* allocate(size_t __n)  //__n = 64
  {
    void* __ret = 0;

    else {
      _Obj* __STL_VOLATILE* __my_free_list
          = _S_free_list + _S_freelist_index(__n);//_S_free_list[7]
		  
		  _Obj* __RESTRICT __result = *__my_free_list;
      if (__result == 0)
        __ret = _S_refill(_S_round_up(__n));
      else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
      }
	}
  }
3、接着申请96字节
__size = 96 
__nobjs=  20
char*
__default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
    char* __result;
    size_t __total_bytes = __size * __nobjs = 96 * 20 = 1920;
    size_t __bytes_left = _S_end_free - _S_start_free = 0;
	
	
	else {
        size_t __bytes_to_get = 
	  2 * __total_bytes + _S_round_up(_S_heap_size >> 4)
	  = 2 * 1920 + _S_round_up(1280>>4) = 3840 + 80 = 3920;
	}
	
	_S_start_free = (char*)malloc(__bytes_to_get)
	                 = malloc(3920);
     _S_heap_size += __bytes_to_get = 1280 + 3920 = 5200;
     _S_end_free = _S_start_free + __bytes_to_get;
     return(_S_chunk_alloc(__size, __nobjs));	//递归调用				 

     //递归调用
	char* __result;
    size_t __total_bytes = __size * __nobjs = 96 * 20 = 1920;
    size_t __bytes_left = _S_end_free - _S_start_free = 5200;
	
	 if (__bytes_left >= __total_bytes) {
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }

}




void* __default_alloc_template::_S_refill(size_t __n)//__n = 96
{
    int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs) = 1920;
    _Obj* __STL_VOLATILE* __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	 __result = (_Obj*)__chunk;
      *__my_free_list = __next_obj = (_Obj*)(__chunk + __n);
      for (__i = 1; ; __i++) {
        __current_obj = __next_obj;
        __next_obj = (_Obj*)((char*)__next_obj + __n);
        if (__nobjs - 1 == __i) {
            __current_obj -> _M_free_list_link = 0;
            break;
        } else {
            __current_obj -> _M_free_list_link = __next_obj;
        }
      }

}
static void* allocate(size_t __n)//__n = 96
  {
    void* __ret = 0;

    else {
      _Obj* __STL_VOLATILE* __my_free_list
          = _S_free_list + _S_freelist_index(__n);//_S_free_list[11]

      _Obj* __RESTRICT __result = *__my_free_list;
      if (__result == 0)
        __ret = _S_refill(_S_round_up(__n));
      else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
      }
    }

    return __ret;
  }


4、在申请72字节，此时堆空间与内存池都是空的（不足以分配72字节）
//从96哪里借的96字节，并且将96切分为72 与24（留到内存池）
__size = 72
__nobjs = 20
char*
__default_alloc_template::_S_chunk_alloc(size_t __size, int& __nobjs)
{
    char* __result;
    size_t __total_bytes = __size * __nobjs = 1440;
    size_t __bytes_left = _S_end_free - _S_start_free = 0;
	
	else {
        size_t __bytes_to_get = 
	  2 * __total_bytes + _S_round_up(_S_heap_size >> 4)   > 2880 ;
	   _S_start_free = (char*)malloc(__bytes_to_get); 

	    if (0 == _S_start_free) {
            size_t __i;
            _Obj* __STL_VOLATILE* __my_free_list;
			_Obj* __p;
           
            for (__i = __size;
                 __i <= (size_t) _MAX_BYTES;
                 __i += (size_t) _ALIGN) {
					 //_S_free_list[8]
                __my_free_list = _S_free_list + _S_freelist_index(__i);
                __p = *__my_free_list;
                if (0 != __p) {
                    *__my_free_list = __p -> _M_free_list_link;
                    _S_start_free = (char*)__p;
                    _S_end_free = _S_start_free + __i;
                    return(_S_chunk_alloc(__size, __nobjs));//递归调用
                  
                }
            }
	    
         
        }
	   //递归调用
	   char* __result;
       size_t __total_bytes = __size * __nobjs = 72 * 20 = 1440;
       size_t __bytes_left = _S_end_free - _S_start_free = 96;
	   
	   else if (__bytes_left >= __size) {
        __nobjs = (int)(__bytes_left/__size) = 96/72 = 1;
        __total_bytes = __size * __nobjs = 72 * 1 = 72;
        __result = _S_start_free;
        _S_start_free += __total_bytes;
        return(__result);
    }
	   
	}

}



__n = 72
void*
__default_alloc_template::_S_refill(size_t __n)
{
    int __nobjs = 20;
    char* __chunk = _S_chunk_alloc(__n, __nobjs);
    _Obj* __STL_VOLATILE* __my_free_list;
    _Obj* __result;
    _Obj* __current_obj;
    _Obj* __next_obj;
    int __i;
	
	
	 if (1 == __nobjs) return(__chunk);

}



 static void* allocate(size_t __n)//__n = 72
  {
    void* __ret = 0;

    else {
      _Obj* __STL_VOLATILE* __my_free_list
          = _S_free_list + _S_freelist_index(__n);

      _Obj* __RESTRICT __result = *__my_free_list;
      if (__result == 0)
        __ret = _S_refill(_S_round_up(__n));
      else {
        *__my_free_list = __result -> _M_free_list_link;
        __ret = __result;
      }
    }

    return __ret;
  };



 static void deallocate(void* __p, size_t __n)//__n = 96
  {
   
    else {
      _Obj* __STL_VOLATILE*  __my_free_list
          = _S_free_list + _S_freelist_index(__n);
      _Obj* __q = (_Obj*)__p;


      __q -> _M_free_list_link = *__my_free_list;
      *__my_free_list = __q;
      // lock is released here
    }
  }
```

### 14、迭代器失效【⭐】

- vector

  - insert的时候有可能会扩容（**指针指向已经不受控制的区域**），为防止报错，可以每次都将 it = nums.begin() 进行重新置为 =》 所以**不推荐在vector中间进行insert操作**，在头insert一个元素可以但不为O1
  - erase删除的时候元素会向前挪动，**使用迭代器进行删除时要注意**，不要再++it，会错过一个元素

- list

  -  基于list（for循环）删除元素,迭代器失效的问题
  -  对于链表式容器(如list)，删除当前的iterator，仅仅会使当前的iterator失效，这是因为list之类的容器，使用了链表来实现，插入、删除一个结点不会对其他结点造成影响。**只要在erase时，递增当前iterator即可**，并且**erase方法可以返回下一个有效的iterator**。
  
  ```c++
   iter = List.erase(iter);  //erase删除元素，返回下一个迭代器
  ```



# C++11新特性

### 1、C++11有哪些新特性

- nullptr替代 NULL
- 引入了 auto 和 decltype 这两个关键字实现了类型推导
- 基于范围的 for 循环for(auto& i : res){}
- 类和结构体的中初始化列表
- Lambda 表达式（匿名函数）
- std::forward_list（单向链表）
- 右值引用和move语义
- 智能指针

### 2、[lambda函数](https://blog.csdn.net/sgh666666/article/details/89000215)【⭐】

1) 利用lambda表达式可以编写内嵌的**匿名函数**，用以**替换独立函数或者函数对象**；

2) 每当你定义一个lambda表达式后，编译器会自动生成一个匿名类（这个类当然重载了()运算符），我们称为闭包类型（closure type）。那么在运行时，这个lambda表达式就会返回一个匿名的闭包实例，其实一个右值。所以，我们上面的lambda表达式的结果就是一个个闭包。闭包的一个强大之处是其可以通过传值或者引用的方式捕捉其封装作用域内的变量，前面的方括号就是用来定义捕捉模式以及变量，我们又将其称为lambda捕捉块。

3) **lambda表达式的语法定义如下**：

```c++
[capture] （parameters） mutable -> return type {statement};

/*
capture：表示捕获列表，是一个lambda所在函数中定义的局部变量列表
parameter：表示参数列表
return type：返回类型
statement：函数体
*/
```

4) lambda必须使用尾置返回来指定返回类型，**可以忽略参数列表和返回值**，**但必须永远包含捕获列表和函数体；**

### 3、C++中的智能指针？三种指针解决的问题以及区别？【⭐】

（1）

- C++11中引入了智能指针的概念，**方便管理堆内存**。使用普通指针，容易造成堆内存泄露（忘记释放），二次释放，程序发生异常时内存泄露等问题等，使用智能指针能更好的管理堆内存。

- 智能指针在C++11版本之后提供，包含在头文件<memory>中，**shared_ptr、unique_ptr、weak_ptr**。

（2）

- **shared_ptr多个指针指向相同的对象**。shared_ptr使用引用计数，每一个shared_ptr的拷贝都指向相同的内存。每使用他一次，内部的引用计数加1，每析构一次，内部的引用计数减1，减为0时，自动删除所指向的堆内存。shared_ptr内部的引用计数是线程安全的，但是对象的读取需要加锁。

- **unique_ptr“唯一”拥有其所指对象，同一时刻只能有一个unique_ptr指向给定对象**（通过禁止拷贝语义、只有移动语义来实现）。**转移一个unique_ptr将会把所有权全部从源指针转移给目标指针，源指针被置空**；所以unique_ptr不支持普通的拷贝和赋值操作，不能用在STL标准容器中；
- weak_ptr：弱引用。 **引用计数有一个问题就是互相引用形成环（环形引用），这样两个指针指向的内存都无法释放。需要使用weak_ptr打破环形引用。**weak_ptr是一个弱引用，它是为了配合shared_ptr而引入的一种智能指针，**它指向一个由shared_ptr管理的对象而不影响所指对象的生命周期**，也就是说，**它只引用，不计数**。如果一块内存被shared_ptr和weak_ptr同时引用，**当所有shared_ptr析构了之后，不管还有没有weak_ptr引用该内存，内存也会被释放**。所以**weak_ptr不保证它指向的内存一定是有效的，在使用之前使用函数lock()检查weak_ptr是否为空指针。**

### 4、weak_ptr如何解决shared_ptr的循环引用问题？

- 循环引用（环形引用）是指使用多个智能指针share_ptr时，**出现了指针之间相互指向，从而形成环的情况**，有点类似于死锁的情况，这种情况下，**智能指针往往不能正常调用对象的析构函数，从而造成内存泄漏**。

```c++
#include <iostream>
using namespace std;

template <typename T>
class Node
{
public:
    Node(const T& value)
        :_pPre(NULL)
        , _pNext(NULL)
        , _value(value)
    {
        cout << "Node()" << endl;
    }
    ~Node()
    {
        cout << "~Node()" << endl;
        cout << "this:" << this << endl;
    }

    shared_ptr<Node<T>> _pPre;
    shared_ptr<Node<T>> _pNext;
    T _value;
};

void Funtest()
{
    shared_ptr<Node<int>> sp1(new Node<int>(1));
    shared_ptr<Node<int>> sp2(new Node<int>(2));

    cout << "sp1.use_count:" << sp1.use_count() << endl;
    cout << "sp2.use_count:" << sp2.use_count() << endl;

    sp1->_pNext = sp2; //sp1的引用+1
    sp2->_pPre = sp1; //sp2的引用+1

    cout << "sp1.use_count:" << sp1.use_count() << endl;
    cout << "sp2.use_count:" << sp2.use_count() << endl;
}
int main()
{
    Funtest();
    system("pause");
    return 0;
}
//输出结果
//Node()
//Node()
//sp1.use_count:1
//sp2.use_count:1
//sp1.use_count:2
//sp2.use_count:2
```

- 解决：**弱指针用于专门解决shared_ptr循环引用的问题，weak_ptr不会修改引用计数，即其存在与否并不影响对象的引用计数器**。循环引用:就是**两个对象互相使用一个shared_ptr成员变量指向对方**。弱引用并不对对象的内存进行管理，在功能上类似于普通指针，然而一个比较大的区别是，弱引用能检测到所管理的对象是否已经被释放，从而避免访问非法内存。​​

### 5、function 与 bind

使用std::function<T>和bind实现回调函数，可以产生与纯虚函数和继承一样的多态效果

