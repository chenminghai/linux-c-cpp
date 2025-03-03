# 02_open-close-read-write

## 2. open函数

```c
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <errno.h>

#include <stdlib.h>

#define print_error(str) \
do{\
    fprintf(stderr, "File %s, Line %d, Function %s error\n",__FILE__, __LINE__, __func__);\
    perror(str);\
    exit(-1);\
}while(0);

int main(void)
{
    int fd = 0;
    fd = open("file.txt", O_RDWR);
    if(fd < 0){
        print_error("open fail");
    }else{
        printf("open successfully!\n");
    }
    
    char buf_w[] = "hello wolrd";
    int ret = write(fd, (void *)buf_w, strlen(buf_w));
    if(ret < 0){
        print_error("write error");
    }else{
        printf("write successgully!\n");
    }
    
    // 读之前需要把文件指针拨回到文件头
    lseek(fd, SEEK_SET, 0);
    
    char buf_r[30];
    read(fd, buf_r, sizeof(buf_w));
    
    printf("buf_r = %s\n", buf_r);
    
    close(fd);
    return 0;
}
```

### 2.1 函数原型

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

// 功能：只能打开存在的文件，如果文件不存在就返回-1报错。
int open(const char *pathname, int flags);

// 功能：如果文件存在就直接打开，如果文件不存在，就按照mode指定的文件权限，创建一个该名字的新文件。
int open(const char *pathname, int flags, mode_t mode);
```

也就是说三个参数时，不仅包含打开已存在文件的功能，还多了一个创建文件的功能。

### 2.2 open函数返回值

+ 如果打开成功，返回一个`非负整数`的文件描述符  
+ 如果打开失败，返回`-1`，并且`设置错误号给系统定义的全局变量errno`，用于标记函数到底出了什么错误  

### 2.3 open函数的参数

+ 参数1：`pathname`, 表示路径名，很简单,传入字符串指针即可,不作介绍
+ 参数2：`flags`, 文件的打开方式，重点讲解
+ 参数3：`mode`, 创建文件时，用于指定文件的原始权限，其实就是`rwxrwxr--`，用默认值也行

#### 2.3.1 flags 之 O_RDONLY、O_WRONLY、O_RDWR、O_TRUNC、O_APPEND

+ flags的作用  
  flags用于指定文件的打开方式，这些宏还可以使用|组合，比如O_RDONLY | O_APPEND，同时指定多个宏
  
  不要被这些宏吓到了，理解了，用多了，自然就熟悉了，不用可以去记住，想不起来了，使用man查看open函数就能知道。

  这些宏对应的就是一些整形数，#define O_RDONLY 2。

+ 这些宏被定义在了那里？  
  定义在了open所需要的头文件中，使用open函数时，必须要包含对应的头文件，否者，这些宏你就用不了。


+ （1）宏的含义
  + （a）`O_RDONLY`：只读方式打开文件，只能对文件进行读     
  + （b）`O_WRONLY`：只写方式打开文件，只能对文件记性写
  + （c）`O_RDWR`：可读可写方式打开文件，既能读文件，也能写文件
      以上这三个在指定时，`只能唯一指定，不可以组合`，比如O_RDONLY|O_WRONLY
  + （d）`O_TRUNC`：打开时将文件内容全部清零空
  + （e）`O_APPEND`：打开文件后，写数据时，原有数据保留，新写的数据追加到文件末尾，此选项很重要
      APPEND本来就是追加的意思, 如果不指定这个选项的话，新写入的数据会从文件头上开始写，覆盖原有的数据，所有以后会经常使用这个选项
  + （f）`O_NONBLOCK`和`O_ASYNC`：后面的课程用到后，再来介绍

+ （2）代码演示

#### 2.3.2 flags参数 之 `O_CREAT`、`O_EXCL`

+ （1）`O_CREAT`
  + 1）open两个参数时的缺点
    只能用于打开已经存在的文件，如果文件不存在就返回-1报错
    
  + 2）O_CREAT的作用
    可以解决两个参数的缺点，指定O_CREAT时，如果：
    + （a）文件存在：直接打开
    + （b）文件不存在：创建该“名字”的文件。
    不过指定O_CREAT，需要给open指定第三个参数mode，用于指定新创建文件的原始权限。
    ```c
    int open(const char *pathname, int flags, mode_t mode);
    ```
    这个权限就是文件属性中的`rwxrwxr--`

+ （2）`O_EXCL`(EXist CLear)
    **当O_EXCL与O_CREAT同时被指定**，打开文件时，**如果文件之前就存在的话，就报错**  

    意义：**保证每次open的是一个新的文件**，如果文件以前就存在，提醒你open的不是一个新文件. 后面具体用到了，你自然就知道O_EXCL的实用价值了

#### 2.3.3 mode参数：文件的读写和执行权限

> 用16进制来制指定文件权限(读、写、执行)，常用的参数如下：

+ `S_IRWXU`  `00700`  user (file owner) has read, write and execute permission
+ `S_IRUSR`  `00400`  user has read permission
+ `S_IWUSR`  `00200`  user has write permission
+ `S_IXUSR`  `00100`  user has execute permission
+ `S_IRWXG`  `00070`  group has read, write and execute permission
+ `S_IRGRP`  `00040`  group has read permission
+ `S_IWGRP`  `00020`  group has write permission
+ `S_IXGRP`  `00010`  group has execute permission
+ `S_IRWXO`  `00007`  others have read, write and execute permission
+ `S_IROTH`  `00004`  others have read permission
+ `S_IWOTH`  `00002`  others have write permission

### 2.4 详论文件描述符

#### 2.4.1 什么是文件描述符

open成功就会返回一个非负整数（0、1、2、3...）的文件描述符，比如我们示例程序中open返回的文件描述符是3  

文件描述符指向了打开的文件，后续的read/write/close等函数的文件操作，都是通过文件描述符来实现的  

#### 2.4.2 文件描述符池 

> 文件描述符池是进程独立地，即每个进程最多都可以有1024个文件描述符，范围为0~1023

每个程序运行起来后，就是一个进程，系统会给每个进程分配`0~1023`的文件描述符范围，也就是说每个进程打开文件时，open所返回的文件描述符，是在0~1023范围中的某个数字  

0~1023这个范围，其实就是文件描述符池。

1023这个上限可不可以改？  
可以，但是没有必要，也不会介绍如何去改，因为一个进程基本不可能出现，同时打开1023个文件的情况，文件描述符的数量百分百够用。

#### 2.4.3 在我们的例子中，为什么open返回的是3 

open返回文件描述符是由规则的：  
规则就是，open返回文件描述符池中，当前最小没用的哪一个  
进程一运行起来，0/1/2默认就被使用了，最小没被用的是3，所以返回3  
如果又打开一个文件，最小没被用的就应该是4，所以open返回的应该是4

疑问：0、1、2被用来干嘛了，后面解释

#### 2.4.4 文件关闭后，被文件用的描述符怎么办

会被释放，等着下一次open时，被重复利用。演示：

#### 2.4.5 open的文件描述符 与 fopen的文件指针

+ （1）open：Linux 的系统函数（文件io函数） 
  open成功后，返回的文件描述符，指向了打开的文件。

+ （2）fopen：C库的标准io函数
  ```c
  #include <stdio.h>
  FILE *fopen(const char *path, const char *mode);
  ```

  fopen成功后，返回的是`FILE *`的文件指针，指向了打开的文件。

+ （3）对于Linux的C库来说，fopen 这个C库函数，最终其实还是open函数来打开文件的  
  fopen只是对open这个函数做了二次封装  
  
   ```c
        应用程序
            |
            |
  FILE* fopen标准io函数（c库函数）
            |
            |
  int open文件io函数（Linux系统函数）
   ```


  也就是说，fopen的文件指针，最终还是会被换成open的文件描述符，然后用于去操作打开的文件。
  讲第3章-C库的标准io函数时，会详细介绍文件指针与文件描述符之间的关系。

### 2.5 errno和错误号

在我们的例子中，如果open失败了，只是笼统的打印出“打开文件失败了”，但是并没有提示具体出错的原因，没有详细的出错 原因提示，遇到比较难排查的错误原因时，很难排查出具体的函数错误。				

open失败，如何具体打印出详细的出错信息呢？这就不得不提errno的作用了。

#### （1）什么是ernno？

函数调用出错时，Linux系统使用错误编号（整形数）来标记具体出错的原因，每个函数有很多错误号，每个错误号代表了一种错误，产生这个错误时，会自动的将错误号赋值给errno这个全局变量  

errno是Linux系统定义的全局变量，可以直接使用  

错误号和errno全局变量被定义在了哪里?  
都被定义在了errno.h头文件，使用errno时需要包含这个头文件, 在Linux上的路径为`/usr/include/asm-generic/errno.h`

man errno，就可以查到errno.h头文件  


#### （2）打印出具体的出错原因

+ 1）printf打印出错误号
  使用errno时，编译提示‘errno’ undeclare的错误，表示找不到errno全局变量  
  错误号确实标记了具体的出错原因，但是我们并不知道这个错误号，具体到底代表的是什么错误。就好像光给你某人的身份证号，仅凭这个身份证号你无法判断是那一个人，需要换成名字才成  

+ 2）perror函数
  perror函数可以自动将“错误号”换成对应的文字信息，并打印出来，方便我们理解
  
  ```shell
  man perror
  ```
 
  perror是一个C库函数，不是一个系统函数  
  
  perror的工作原理？  
  调用perror函数时，它会自动去一张对照表，将errno中保存的错误号，换成具体的文字信息并打印出来，我们就能知道函数的具体错误原因了。

+ 3）`man open`，可以查看open函数，都有哪些错误号   
    每个错误号代表了一种函数可能的出错情况，比如：  
    EACCES：不允许你访问文件而出错  
    Liunx为了让错误号能够见名识意，都给这些整形的错误号定义了对应的宏名，这些宏定义都被定义在了error.h头文件中  
    man perror这个函数，也可以看到这个头文件  
    
    疑问：我是不是要必须记住这些错误号？  
    答：反正我是记不住，记不住怎么办？  

    根本不需要记住，使用perror函数，它可以自动翻译，我们讲错误号，只是希望你理解错误号这个东西，后面的课程会经常见到东西，到了后面我就不再介绍。
			
## 3. close、write、read、0/1/2这三个文件描述符

### 3.1 close函数

#### 3.1.1 功能

关闭打开的文件。

```c
close(fd);
```

就算不主动的调用close函数关闭打开的文件，进程结束时，也会自动关闭进程所打开的所有的文件。

但是如果因为某种需求，你需要在进程结束之前关闭文件的话，就主动的调用close函数来实现。

Linux c库的标准io函数fclose，向下调用时，调用就是close系统函数。

#### 3.1.2 close关闭文件时做了什么 

+ （1）open打开文件时，会在进程的task_struct结构中，创建相应的结构体，以存放打开文件的相关信息  
+ （2）结构体的空间开辟于哪里  
  open函数会通过调用类似malloc的函数，在内存中开辟相应的结构体空间，如果文件被关闭，存放该文件被打开时信息的结构体空间，就必须被释放，类似free(空间地址);不过malloc和free是给C应用程序调用的库函数，Linux系统内部开辟和释放空间时，用的是自己特有的函数  
  
  如果不释放，当前进程的内存空间，会被一堆的站着茅坑不拉屎的垃圾信息所占用，随后会导致进程崩溃，甚至是Linux系统的崩溃  
  
  这就好比一个挺大的仓库，每次废弃的物品都不及时清理，最后整个空间全被垃圾塞满，仓库还是那个仓库，但是仓库瘫痪了，无法被正常使用  
  
  因此close文件时，会做一件非常重要的事情，释放存放文件打开信息的结构体空间  

+ （3）有关task_stuct结构体  
   + 1）这个结构体用于存放进程在运行的过程中，所涉及到的各种信息，其中就包括进程所打开文件的相关信息。

   + 2）task_stuct结构体，什么时候开辟的？  
     进程（程序）开始运行时，由Linux系统调用自己的系统函数，在内存中开辟的，

     代码定义个各种变量（int a、数组、结构体、对象等），开辟这些空间时，这些空间都是来自于内存。

     每一个进程都有一个自己的task_stuct结构体，用于记录自己的所有相关信息，这就好比每一个人，在公安局都有一份属于自己的档案信息，task_stuct结构体记录的就是，进程在活着时的一切档案信息
     
   + 3）什么时候释放  
    进程运行结束了，Linux系统会调用自己的系统函数，释放这个结构体空间，如果不释放的话，每运行一个进程就开辟一个，但是进程结束后都不释放，最后会导致Linux系统自己的内存空间不足，从而使得整个Linux系统崩溃  

### 3.2 write

#### 3.2.1 函数原型

```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t count);
```

+ （1）功能：向fd所指向的文件写入数据 
+ （2）参数  
  +  1）fd：指向打开的文件  
  +  2）**buf：保存数据的缓存空间的`起始地址`**  
  +  3）count：从起地址开始算起，把缓存中count个字符，写入fd指向的文件  
    数据中转的过程：  
    
    ```txt
    应用缓存（buf）————>open打开文件时开辟的内核缓存——————>驱动程序的缓存——————>块设备上的文件
    ```

+ （3）返回值
  + 调用成功：返回所写的字符个数  
  + 调用失败：返回-1，并给errno自动设置错误号，使用perror这样的函数，就可以自动的将errno中的错误号翻译为文字

#### 3.2.2 使用write

+ （1）
  ```c
  #include <stdio.h>
  #include <string.h>
  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  #include <unistd.h>
  #include <errno.h>

  #include <stdlib.h>

  #define print_error(str) \
  do{\
      fprintf(stderr, "File %s, Line %d, Function %s error\n",__FILE__, __LINE__, __func__);\
      perror(str);\
      exit(-1);\
  }while(0);

  int main(void)
  {
      int fd = 0;
      fd = open("file.txt", O_RDWR | O_CREAT, 0664);
      if(fd < 0){
          print_error("open fail");
      }else{
          printf("open successfully! fd = %d\n", fd);
      }

      char buf_w[] = "hello wolrd";
      // 返回成功写入了多少个字节
      int ret = write(fd, (void *)buf_w, strlen(buf_w));

      // int ret = write(fd, (void *)buf_w + 1, strlen(buf_w)); // 数组越位,会报错

      // int ret = write(fd, (void *)buf_w + 1, strlen(buf_w) - 1); // 少写入一个字符
      if(ret < 0){
          print_error("write error");
      }else{
          printf("write successgully! ret = %d\n", ret);
      }

      close(fd);
      return 0;
  }
  ```
+ （2）思考1：`write(fd, buf+1, 10)` 写入时，是一个什么情况
+ （3）思考2：`write(fd, "hello world", strlen("hello world"))`,直接写字符串，可不可以？
+ （4）为什么直接写字符串可以？
  
  ```c
  char buf[] = "hello world";
  write(fd, buf, 11));
  ```

  像这种情况，字符串直接缓存在了应用空间buf中，buf代表的数组第一个元素h所在空间的地址。直接写字符串常量时，字符串常量被保存（缓存）在了常量区，编译器在翻译如下这句话时
  
  ```c
  write(fd, "hello world", strlen("hello world"))
  ```
  
  会直接将"hello world"翻译为，"hello world"所存放空间的起始地址（也就是h所在字节的地址），换句话说，直接使用使用字符串常量时，字符串常量代表的其实是一个起始地址。
  
  `strlen("hello world")` 时，其实就是把起始地址传给了strlen函数。

  有关这个内容，实际上是c语言的基本知识，这里只是简单的介绍下，不清楚的同学，说明你的C语言还不过关，你需要认真的把c语言学好。

+ （5）思考3：`write(fd, "hello world"+1, 10)`，写入文件的又是什么样的内容。

### 3.2 `read`

#### 3.2.1 函数原型	

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);
```

+ （1）功能：从fd指向的文件中，将数据读到应用缓存buf中

+ （2）参数
  + 1）fd：指向打开的文件
  + 2）buf：读取到数据后，用于存放数据的应用缓存的起始地址
  + 3）count：缓存大小（字节数）

+ （3）返回值
  + 1）成功：返回读取到的字符的个数
  + 2）失败：返回-1，并自动将错误号设置给errno。

数据中转的过程：  

```txt
应用缓存（buf）<————open打开文件时开辟的内核缓存<——————驱动程序的缓存<——————块设备上的文件
```

#### 3.2.2 使用read		

+ （1）

  ```c
  #include <stdio.h>
  #include <string.h>
  #include <sys/types.h>
  #include <sys/stat.h>
  #include <fcntl.h>
  #include <unistd.h>
  #include <errno.h>

  #include <stdlib.h>

  #define print_error(str) \
  do{\
      fprintf(stderr, "File %s, Line %d, Function %s error\n",__FILE__, __LINE__, __func__);\
      perror(str);\
      exit(-1);\
  }while(0);

  int main(void)
  {
      int fd = 0;
      fd = open("file.txt", O_RDWR | O_CREAT, 0664);
      if(fd < 0){
          print_error("open fail");
      }else{
          printf("open successfully! fd = %d\n", fd);
      }

      // 字符数组一定要初始化，否则当不懂下标为0的位置开始存时，其他位置是空，到时候打印不出来buf_r字符串
      char buf_r[30]={0}; 
      // 成功读取了多少个字符
      int ret = read(fd, buf_r, sizeof(buf_r)); // 把buf_r读满

      // int ret = read(fd, buf_r, 5); // 读取少于实际的字符

      int ret = read(fd, buf_r, 5); // 读入的时候从第3个位置开始存，但是buf_r必须初始化，要不print不出来

      if(ret < 0){
          print_error("read error");
      }else{
          printf("read successgully! ret = %d\n", ret);
          printf("result : %s\n", buf_r);
      }

      close(fd);
      return 0;
  }
  ```
+ （2）思考
    
    ```c
    char buf[30];
    read(fd, buf+3, 11);
    ```

    会是什么样的效果？  
    猜测的话，数据从`buf[3]`开始存放，一直放到`buf[14]`

#### 3.2.3 API调用时的正规写法  

当函数调用失败后，为了能够快速准确的排查错误，原则上来说，应该对所有的函数都进行错误处理（错误打印）。

不过后续为了讲课的方便，除非非常必要，否则在我们写的测试代码中，就不进行错误检测了。

### 3.3 `0/1/2`这三个文件描述符

#### （1）程序开始运行时，有三个文件被自动打开了，打开时分别使用了这三个文件描述符 

```shell
root@6fb4b72f0c7c:/usr/include# ls /dev/std* -l
lrwxrwxrwx 1 root root 15 Mar 25 03:46 /dev/stderr -> /proc/self/fd/2
lrwxrwxrwx 1 root root 15 Mar 25 03:46 /dev/stdin -> /proc/self/fd/0
lrwxrwxrwx 1 root root 15 Mar 25 03:46 /dev/stdout -> /proc/self/fd/1
```

#### （2）依次打开的三个文件分别是`/dev/stdin`、`/dev/stdout`、`/dev/stderr`

+ 1）`/dev/stdin`：标准输入文件   

  ```c
  #include <stdio.h>
  #include <unistd.h>
  #include <errno.h>

  int main(void)
  {
      char buf[30] = {0};
      int ret = read(0, (void *)buf, sizeof(buf)); // 第一个参数fd = 0，表示从键盘读
      if(ret < 0){
          perror("read fail");
          return -1;
      }
      printf("buf = %s\n", buf);
      return 0;
  }
  ```
  
  + （a）程序开始运行时，默认会调用open("/dev/stdin", O_RDONLY)将其打开，返回的文件描述符是0
  + （b）使用0这个文件描述符，可以从键盘输入的数据  
        简单理解就是，/dev/stdin这个文件代表了键盘
  + （c）思考：read(0, buf, sizeof(buf))实现的是什么功能  
    实现的是，从键盘读取数据到到缓存buf中，数据中转的过程是：

    ```txt
    read应用缓存buf <—————— open /dev/stdin时开辟的内核缓存 <——————键盘驱动程序的缓存 <——————键盘
    ```

    疑问：能够像读普通文件一样读键盘吗？  
    答：在Linux下，应用程序通过OS API操作底层硬件时，都是以文件形式来操作的，不管是读键盘，还是向显示器输出文字显示，都是以文件形式来读写的，在Linux下有句很经典的话，叫做“在Linux下一切皆文件”，说的就是这么个意思。

  + （d）思考：为什么在我们的程序中，默认就能使用scanf函数从键盘输入数据  
    我们默认就打开了代表了键盘/dev/stdin，打开后0指向这个打开的文件  
    scanf下层调用的就是read，read自动使用0来读数据，自然就可以从键盘读到数据。

    ```txt
    scanf("%s", sbuf) C库函数
        |
        |
    read(0, rbuf, ***)
    ```
    我们从键盘读取数据时，可以直接调用read这个系统函数来实现，也可以调用scanf C库函数来实现，只不过在一般情况下，我们在实际编写应用程序时，调用的更多的还是scanf 这个c库函数，原因如下：
    + 调用库函数，可以让程序能够更好的兼容不同OS，能够在不同的OS运行
    + scanf在read基础上，加入更加人性化的、更加便捷的功能，比如格式化转换
    + 直接使用read的缺点
      首先我们必须清楚，所有从键盘输入的都是字符，从键盘输入100，其实输入的是三个字符'1'、'0'、'0'，因此使用read函数从键盘读数据时，读到的永远都是字符。但是如果你实际想得到的是整形数的100，你就必须自己将'1'、'0'、'0'转为整形的100，
      每次使用read获取一个整形数时，你都要自己转换，如果你要输入的是浮点数的话，转起来更麻烦。

      思考：字符串形式的浮点数怎么转为真正的浮点数？
      c语言面试题中就会经常考这么一道题，“请将字符串形式的"123.45"转为真正的浮点数”，请大家自行实现。
    + scanf的优点
      scanf可以解决read的缺点，虽然scanf调用read时，从键盘读到的全部都是字符，但是你只要给scanf指定%d、%f等格式，scanf会自动的讲read读到的字符串形式的数据，转为整形或者浮点型数据

      ```txt
      scanf("%d", &a);
            |
            |
      read(0, buf, ...);
      ```

      如果直接调用read读取，然后自己再来转换，相当于自己再实现scanf函数的基本功能。

      疑问：有了scanf函数后，read系统函数是不是没有调用的意义了?
      当然不是的，在有些时候，特别是讲到后面驱动时，有些情况还就只能使用read，不能使用scanf，学到后面就知道了。

  + （e）思考：close(0)后，scanf还能工作吗？为什么不能工作？

      ```txt
      scanf("%s", sbuf) C库函数
            |
            |
      read(0, rbuf, ***)
      ```



+ 2）/dev/stdout：标准输出文件
  + （a）程序开始运行时，默认open("/dev/stdout", O_WRONLY)将其打开，返回的文件描述符是1	  
      为什么返回的是1，先打开的是/dev/stdin，把最小的0用了，剩下最小没用的是1，因此返回的肯定是1  
      
  + （b）通过1这个文件描述符，可以将数据写（打印）到屏幕上显示  
      简单理解就是，`/dev/stdout这个文件代表了显示器`  
      
  + （c）思考：`write(1, buf, strlen(buf))`实现的是什么功能  
      将buf中的数据写到屏幕上显示，数据中转的过程是：

      ```
      write应用缓存buf ——————> open /dev/stdout时开辟的内核缓存 ——————> 显示器驱动程序的缓存 ——————> 喜爱能使其
      ```

  + （d）思考：为什么在我们的程序中，默认就能使用printf打印数据到显示器
      因为程序开始运行时，就默认打开了代表了显示器/dev/stdout文件，然后1指向这个打开的文件  
      printf下层调用的是write函数，write会使用1来写数据，既然1所指文件代表的是显示器，自然就可以将数据写到显示器了  

      ```txt
      printf("*****")
        |
        |
      write(1, buf, count);
      ```



  + （e）思考：如何使用write函数，将整数65输出到屏幕显示？

      + 直接输出行不行
      
        ```c
        int a = 65；
        write(1, &a, sizeof(a));
        ```

        为什么输出结果是A？  
        人只看得懂字符，所以所有输出到屏幕显示的，都必须转成字符  
        所以我们输出时，输出的必须是文字编码，显示时会自动将文字编码翻译为字符图形  
        所以我们输出65时，65解读为A字符的ASCII编码，编码被翻译后的图形自然就是A  

      + 怎么才能打印出65  
          如果要输出65，就必须将整形65转为'6'和'5'，输出这两个字符才行  
          输出'6'、'5'时，其实输出的是'6'、'5'这两个字符的ASCII编码，然后会被自动的翻译为6和5这两个图形  
          总之将整形65，转为字符'6'、'5'输出即可  

      + 为什么printf会对wirte进行封装  
        + 库函数可以很好的兼容不同的OS  
        + 封装时，叠加了很多的功能，比如格式化转换   
          通过指定%d、%f等，自动将其换为对应的字符，然后write输出，完全不用自己来转换
          
          思考：printf使用%s、%c输出字符串和字符时，还用转吗？  
          其实不用转，因为要输出的本来就是字符，printf直接把字符给write就行了，当然也不是一点事情也不做，还是会做点小处理的。

   + （f）思考：close(1)，printf函数还能正常工作吗？  
      
      ```txt
      printf("*****")
            |
            |
      write(1, buf, count);
      ```

+ 3）/dev/stderr：标准出错输出文件
   + （a）默认open("/dev/stderr", O_WRONLY)将其打开，返回的文件描述符是2	 
   + （b）通过2这个文件描述符，可以将报错信息写（打印）到屏幕上显示  
   + （c）思考：write(2, buf, sizeof(buf))实现的是什么功能  
        将buf中的数据写道屏幕上，数据中转的过程是：
        
        ```shell
        write应用缓存buf ——————> open /dev/stderr时开辟的内核缓存 ——————> 显示器驱动程序的缓存 ——————> 显示器
        ```
        
   + （d）疑问：1和2啥区别？

        + 使用这两个文件描述符，都能够把文字信息打印到屏幕。
          如果仅仅是站在打印显示的角度，其实用哪一个文件描述符都能办到。

        + 1、2：有各自的用途

          - 1：专用于打印普通信息

          - 2：专门用于打印各种报错信息

          - 使用write输出时
            + 如果你输出的是普通信息，就使用1输出。
            + 如果你输出的是报错信息，就使用2输出


        + printf和perror调用write时，使用的是什么描述符

          - printf  
            printf用于输出普通信息，向下调用write时，使用的是1  
            前面已经验证过，close(1)后，printf无法正常输出  

          - perror  
              perror专门用于输出报错信息的，因为输出的是报错信息，因此write使用的是2  
              
            验证：将2关闭，perror就会无法正常输出  

            后面讲到标准io时，还会讲到标准输入、标准输出、标准出错输出，到时候会介绍标准输出、标准出错输出的区别  

+ 4）`STDIN_FILENO`、`STDOUT_FILENO`、`STDERR_FILENO`
    系统为了方便使用0/1/2，`系统`分别对应的给了三个宏

    + `0`：`STDIN_FILENO`  
    + `1`：`STDOUT_FILENO` 
    + `2`：`STDERR_FILENO`  

    可以使用三个宏，来代替使用0/1/2  

    `这三个宏定义在了open或者read、write函数所需要的头文件中，只要你包含了open或者read、write的头文件，这三个宏就能被正常使用`
