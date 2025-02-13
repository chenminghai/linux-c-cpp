# 8. 制作、使用静态库和动态库
## 8.1 制作和使用的方法
方法有两种，
+ 第一种：是通过IDE图形界面来实现
+ 第二种：在命令行通过命令来实现

不管哪一种方式都是一样的，都是使用“编译器集合”所提供的命令（程序）来操作的。

只不过在命令行方式下，我们是直接使用这些命令来操作，而IDE方式则会提供图形界面给我们操作，至于IDE背后是如何调用命令去实现的，我们无需关心。不过IDE图形界面方式显然会比命令行方式更加的人性化  

我们会以Windows和Linux这两个平台来对比介绍库的制作和使用，在这两个平台下，我们都可以使用“命令行方式”和“IDE方式”来实现。

如果在这两个平台下，命令行方式和IDE方式都介绍的话，会把搞得太过麻烦，所以我们是这么安排：
+ Liunx平台：我们通过命令行方式来实现，Linux下的IDE方式省略
+ Windows平台：我们通过IDE（Codeblocks）方式来实现，windows下的命令行方式省略
  
库的制作和使用原理都是类似的，但是由于平台（OS）、编译器、语言的不同，所以在库的制作和使用细节上会
有所差异，很多同学觉得“库的制作和使用”有点混乱，就是因为这样的原因导致的。

我们目前制作c库时，所用的编译器为gcc，OS平台为linux和windows，至于与其它编译器、其它平台相关的c库的制作，请自行解决。

## 8.2 通过Linux命令行来制作c库

### 8.2.1 制作、使用静态库

#### （1）编写需要制作为静态库的代码

代码实现add/sub/mul/div/power等算数运算函数，然后做成静态库，在我们的程序中链接了这个静态库之后，我们就可以调用这些函数来进行算数运算了  
		
代码文件：  
+ `caculate.h`：头文件，存放库函数的声明  
+ `add_sub.c`：加法、减法  
+ `mul_div.c`：乘法、除法、平方  

##### 1）caculate.h

```c
#ifndef H_CACULATE_H
#define H_CACULATE_H

extern double add(double arg1, double arg2);
extern double sub(double arg1, double arg2);
extern double mul(double arg1, double arg2);
extern double div(double arg1, double arg2);
extern double power(double arg);

#endif
```

在程序中使用这些函数时，我们需要对这些函数进行声明，为了方便我们进行声明的操作，我们需要把这些函数声明放到.h中，到时候只要包含这个.h即可。  

我们的例子比较简单，目的只是为了向大家演示如何制作和使用静态库，实际上在.h不仅仅只会放函数声明，还需要各种宏定义、结构体类型定义、inline函数等等。  

库函数的.h与我们自己的.h没有任何区别，只不过.h所对应的.c被做成了静态库而已。  

##### 2）add_sub.c

```c
#include "caculate.h"

double add(double arg1, double arg2)
{
	double sum = 0;

	sum = arg1 + arg2;

	return sum;
}

double sub(double arg1, double arg2)
{
	double sum = 0;

	sum = arg1 - arg2;

	return sum;
}
```

##### 3）mul_div.c

```c
#include "caculate.h"

double power(double arg)
{
	double sum = 0;

	sum = mul(arg, arg);

	return sum;
}

double mul(double arg1, double arg2)
{
	double sum = 0;

	sum = arg1 * arg2;

	return sum;
}

double div(double arg1, double arg2)
{
	double sum = 0;

	sum = arg1 / arg2;

	return sum;
}
```

注意：库里面是不能包含main函数的，main在我们自己的程序中。
							
#### （2）将源码制作为静态库文件            

##### 1）得到.o文件 

```c
gcc -c add_sub.c -o add_sub.o 
gcc -c mul_div.c -o mul_div.o 
```
##### 2）将.o文件打包为静态库文件

```shell
ar  -crv  ./libcaculate.a  add_sub.o  mul_div.o
```

+ （a）`ar`：打包命令  
+ （b）`-crv`：显示打包过程  
+ （c）`./libcaculate.a`：静态库的路径名  
     + `./`：静态库的存放路径，你可以指定为任何路径，目前为了演示的便利，我们就放在./下  
     + `libcaculate.a`：静态库的完整名字，前缀lib和后缀.a为固定格式，中间的caculate才是真正的静态库的名字。制作.a静态库时前缀lib为固定格式  
+ （d）`add_sub.o`  `mul_div.o`  
	要打包为静态库文件的.o（原料）  

如果制作成功，使用ls查看时，你会发现在当前目录下就有一个libcaculate.a文件。file命令查看时，会告诉你静态库文件其实是一个归档文件。  

再说说ar命令：  
ar命令其实“归档命令”，专门用来实现对文件进行归档，所以说制作静态库文件其实就是制作一个归档文件。  
归档文件和压缩文件很相似，只不过归档文件只打包不压缩，但是压缩文件不仅会打包，而且还会进行压缩。  

静态库文件放在了ubuntu桌面下的static_lib目录下。  
				
#### （3）使用静态库文件

##### 1）写一个调用库函数的main.c（放在ubuntu的桌面上） 		
```c
#include <stdio.h>		
#include "./static_lib/caculate.h"  //包含静态库的头文件

int main(void)
{
    double a = 10.5;
    double b = 20.6;
    double ret = 0;

    ret = add(a, b); //加
    printf("add: ret = %f\n", ret);

    ret = sub(a, b); //减
    printf("sub: ret = %f\n", ret);

    ret = mul(a, b); //乘
    printf("mul: ret = %f\n", ret);
    ret = div(a, b);
    printf("div: ret = %f\n", ret);

    ret = power(a);  //除
    printf("power: ret = %f\n", ret);

    return 0;
}
```

包含静态库的头文件时，如果caculate.h不在当前路径时怎么办，    
+ 可以在""中指定.h所在的路径  
+ gcc时通过-I选项，将.h所在路径加入"头文件系统路径"  

有关“头文件”的相关内容，我们在第2章有非常详细的介绍，不清楚的同学，请看这一章。  

##### 2）编译main.c并链接静态库

+ （a）如果不链接静态库会怎样
    ```shell
    /tmp/ccTCMbDN.o: In function `main':
    main.c:(.text+0x42): undefined reference to `add'
    main.c:(.text+0x82): undefined reference to `sub'
    main.c:(.text+0xc2): undefined reference to `mul'
    main.c:(.text+0x139): undefined reference to `power'
    collect2: error: ld returned 1 exit status
    ```

报一堆的链接错误，提示链接时找不到这些函数的定义，这种链接错误是我么经常会碰到的错误。   

当你在调用某个函数时如果把函数名写错了，同样也会提示你找不到函数的定义的错误，因为而错误的函数名肯定没有对应任何的函数定义，自然会报错。  

在我们这里并没有把名字写错，而是没有链接定义这些函数静态库，所以找不到函数的定义。  


+ （b）链接静态库	

    ```shell
    gcc main.c -o main -L./static_lib -lcaculate -I./static_lib
    ```

   + -I：指定caculat.h所在的路径，路径为./static_lib

   + -L：指定静态库所在的路径（路径为./static_lib），链接时会到该路径下去查寻找静态库。  
        `疑问` ：能不能省略静态库的路径？  
        答：可以  
            只要将所在路径加入了环境变量，链接时会指定到该路径下寻找静态库，此时我们不用自己指定静态库所在的路径。   
            当然还有一种办法，那就是你可以将“库文件”放到之前早就被加入了环境变量的路径，这样也是可以的。    
        `疑问`：如何将路径加入环境变量呢？  
        答：我们在《Linux系统编、程网络编程》课程的第4章——进程环境中有详细介绍，请大家看这部分课程内容。在课程中windows和Linux的环境变量我们都有介绍了。  

    + -l：注意小的是L，这个用于指定库的名字  
        指定库名字时需要将前缀lib和后缀.a省略。  
        -L指定所在路径，-l指定库名字，如此就会在指定的路径下面找到指定名字的静态库库文件，然后链接它。  
        其实有关-l，我们在第1章介绍gcc -v详细信息就提到过，有关这一点，请大家看第1章的内容。  
			
### 8.2.2 制作、使用动态库

#### （1）编写制作为动态库的例子代码
我们任然还是前面制作静态库的代码。  
			
#### （2）制作动态库文件	

##### 1）得到.o文件 

```shell
gcc -c -fPIC add_sub.c -o add_sub.o 
gcc -c -fPIC mul_div.c -o mul_div.o 
```

+ -fPIC：生成位置无关码
    + 位置有关码：
      代码的地址为绝对地址，代码必须被放在绝对地址所指定的内存位置，这个绝对地址在编译链接时由编译器指定，如果不将代码放到指定位置，将无法正确运行。  
      
      这就好比两个人接头，指定说在“张家巷128号”接头，这个就是位置相关的，要是不到“张家巷128号”这个绝对位置去的话，两个人肯定碰不上面的。  
      
      所以对于位置有关的代码来说，必须运行在“编译链接”时指定地址的内存位置，否则无法正常运行。  
      
      总之，位置有关的代码依赖于编译链接时指定的地址，必须把代码放到地址所指向的内存空间，才能运行。  

   +  位置无关码：
        代码与绝对地址没有关系，放到内存中任何地址的位置都可以正常运行，这就好比两个人接头时约定，双方以烟花为号，大家到放烟花的位置去集合，这就跟绝对地址无关，任何地方都可以碰头，就看烟花在哪里。  

        对于位置无关码来说，不管在代码在内存中什么位置，都可以正常运行，不依赖于编译链接时所指定的地址。  

        有关位置无关码的具体原理，请看后面uboot的课程，里面会详细介绍。  

        `动态库的代码为什么需要是位置无关的？`
          动态库被加载到内存什么位置是不确定的，可能会加载到任何位置，所以必须编译为位置无关码。  
          总之，位置无关码的意思就是，代码不会受到内存位置的影响。  

##### 2）将.o文件打包为动态库文件
制作动态库的命令不再是ar，而是gcc，而且动态库文件不再是普通的归档文件。

```shell
gcc -shared add_sub.o  mul_div.o  -o  ./libmycaculate.so
```

+ （a）`-shared`：制作动态库  
+ （b）`./libmycaculate.so`：动态库的路径名   
        为了和前面介绍的静态库的名字进行区别，我们这里把动态库的名字取名为mycaculate  
+ （c）`add_sub.o` 、`mul_div.o`  
        制作动态库的原料  
									
#### （3）使用动态库文件
使用动态库有两步：  

+ 链接动态库  

+ 加载动态库到内存中  
    链接静态库时，代码会直接被包含到程序中，但是链接动态库时，代码并不会被直接包含到程序中，只是留了一个“函数接口”，所以需要另外将动态库的代码加载到内存中，如果不加载到内存中，则无法调用动态库中的函数。  
    加载动态库的方式有两种，  
    +  第一种：使用“动态库加载器”来加载，这种是最常见的方式  
    +  第二种：在程序中调用“加载函数”来加载  
    因为这两种加载方式的不同，所以在程序中调用“动态库函数”的方式也会有所不同。  

##### 1）使用“动态库加载器”来加载动态库			
+ （a）编写调用动态库函数的程序main.c  
    使用“动态库加载器”加载时，调用动态库函数的方式与调用静态库函数是一样的。  

    ```c
    #include <stdio.h>
    #include "caculate.h"  //包含动态库态库的头文件

    int main(void)
    {
            double a = 10.5;
            double b = 20.6;
            double ret = 0;

            ret = add(a, b); //加
            printf("add: ret = %f\n", ret);

            ret = sub(a, b); //减
            printf("sub: ret = %f\n", ret);

            ret = mul(a, b); //乘
            printf("mul: ret = %f\n", ret);

            ret = div(a, b);  //除
            printf("div: ret = %f\n", ret);

            ret = power(a);  //求平方
            printf("power: ret = %f\n", ret);

            return 0;
    }
    ```

+ （b）链接动态库  
```shell
gcc main.c -L./dynamic_lib -lmycaculate -o main -I./dynamic_lib
```

`-L`：指定动态库所在的路径  
`-l`：指定动态库的名字，需要将lib和.so去掉    
通过以上两个选项，就能在指定路径下找到指定名字的动态库，然后链接这个动态库。  
`-I`：指定动态库头文件的路径  

我们运行程序时会提示说找不到动态库，我们通过ldd命令可以查看程序用到了那些动态库，ldd ./main查
看，也直接提示libmycaculate.so动态库无法被找到。
   ```shell
    linux-vdso.so.1 =>  (0x00007ffedb555000)
    libmycaculate.so => not found   //我们自己的动态库
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f43a89cd000)  //c标准库的动态库，printf等函数就是由它提供的
    /lib64/ld-linux-x86-64.so.2 (0x00007f43a8d97000)  //这个是动态库加载器
   ```

`无法找到的原因？`  
因为“动态库加载器”去加载动态库时，找不到动态库，所以没有能加载到内存中。  

`那么“动态库加载器”怎样才能找到动态库，并将动态库加载到内存中呢？`  
这个问题后面再解决。  


`提问：为什么要链接动态库？`  
    在程序main.c中我们明确的调用了add、sub等动态库函数，编译时必须对这些函数名进行符号解析，而符号解析所需要的信息被包含在了“.so动态库”中，所以编译时必须链接动态库，否则就会提示“函数没有定义的错误”，说白了就是符号解析失败。
   ```shell
    main.c:(.text+0x42): undefined reference to `add'
    main.c:(.text+0x82): undefined reference to `sub'
    main.c:(.text+0xc2): undefined reference to `mul'
    ....
   ```
   不过有意思的是，linux的下的“.so动态库”会直接包含符号解析所需的各种信息，但是windows下的.dll动态库则有所不同，这些符号解析的信息不在.dll中，而是在与.dll配套的“动态库引导文件”中，链接时只需要链接“动态库引导文件”即可进行符号解析。  

总结起来就是：
+ 在Linux下：直接通过链接“.so动态库”来进行符号解析  
+ 在windows下：通过链接“动态库引导文件”来进行符号解析，而不是直接链接“.dll动态库”。  
    有关“动态库引导文件”，后面还会讲到，这里我们先建立点基本的映像。  


`提问`：编译链接后，main.c中add、sub等会变成什么?  
    add、sub等为动态库函数的函数名，函数名就是函数指针，所以编译后会变成函数的第一条指令的地址。  
    如果函数定义就在自己的程序中，那么函数指针一定是绝对地址，因为自己程序中的函数代码会被加载内存中什么位置，在编译阶段就能确定，既然能够确定，那么编译时就直接将“函数指针”指定为绝对地址。  
    但是当调用的是动态库函数时，动态库中的函数定义并不在我们自己的程序中，而是在动态库中。  
    链接动态库时只会留下接口，不会将代码包含到自己的程序中，所以动态库函数的定义不在自己的程序中。  
    而且动态库被加载到内存中什么位置是不确定的，所以说动态库中每个函数在内存中的地址也是不确定的，那么编译时，应该将add、sub等函数名翻译为什么样的地址呢？  
    我们接下来解释一下这个问题。  
    不过我们的解释不是100%准确的，与实际情况有所差别，但是如果完全按照真实情况来讲的话，会很难理解，如果我们采用不太准确的方式来介绍，虽然有点不太准确，但是不会出大错，关键是好理解。  
    我们这么来理解，编译时add、sub等会被编译为相对地址，这个“相对地址”为函数在动态库中相对于“动态库起始位置”的偏移。  
    图：动态库函数相对地址   
    动态库在没有被加载到内存中之前，动态库在内存中位置是不确定，但是一旦加载到了内存中后，动态库在内存中的起始地址就是确定的，系统会记录下动态库在内存中的起始地址，当程序调用add、sub等动态库函数时，  
    “相对地址” + “动态库起始地址”  ————> add、sub等动态库函数在内存中的地址  
    通过这个地址即可跳转到动态库中add、sub函数处，开始执行这些函数的指令，执行完后再返回。  
    图：
    实际情况会比我们这里描述的更难理解，而且涉及到地址映射的问题，我们这里不需要关心到这个层次。  

+ （c）“动态库加载器”怎样才能找到动态库，并将动态库加载到内存中  
    +  动态库加载器
        我们在第1章分析gcc -v的详细过程时就讲过，gcc编译链接程序时，会给我们的程序指定“动态库加载器”，
        `/lib64/ld-linux-x86-64.so.2`   
        前面通过ldd也查看到了这个玩意。  
        动态库加载器是自动工作的，我们不需要关心如何去启动它。  

    +  “动态库加载器”如何才能找到动态库呢？
            “动态库加载器”是通过“动态库环境变量”来知晓，因为“动态库环境变量”中会包含各种动态库所在的路径，“动态库加载器”会自动到这些路径下去搜索。  
            比如C标准库的libc.so的路径就在“动态库环境变量”中，所以“动态库加载器”才会找到libc.so，否者printf、scanf这些函数就用不了，而且我们在第1章就介绍过，由于c标准库中的大部分函数会被频繁用到，所以gcc会默认自动链接libc.so。  
	    `/home/zxf/Desktop/dynamic_lib`  
            总之，我们只要将我们的动态库路径/home/zxf/Desktop/dynamic_lib加入到“动态库环境变量”即可。  
            此时动态库加载器必然就能搜索到我们的动态库，然后将其加载到内存中。  

        疑问：如何设置Linux的“动态库环境变量”呢？
            设置命令：  
            `export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/zxf/Desktop/dynamic_lib`  
            设置完后，我们的程序就能正常运行了。  
            `export`：修改Linux环境变量的命令  
            `LD_LIBRARY_PATH`：“动态库环境变量”的名字，变量的内容就是各个动态库所在的路径  
            `$LD_LIBRARY_PATH`：$表示取变量的内容，所以$LD_LIBRARY_PATH代表的是“动态库环境变量”的内容  
            `:/home/zxf/Desktop`：在原有内容基础上，再加一个新的动态库路径，:为不同路径之间的间隔,新加的路径就是我们的动态库所在的路径。  
            有关环境变量请看《linux系统编程、网络编程》的第4章 进程环境这一章。  
            `疑问`：gcc时不是已经通过-L./dynamic_lib指定了动态库的路径了吗？  
            答：这只是给“编译链接”用的，与动态库加载无关。  

     +  “动态库加载器”是如何加载动态库的  
        运行程序时，“动态库加载器”会首先检测程序中所用到的“动态库代码”，有没有被加载到内存中，  
        +   有：不用再加载了，因为动态库是共享的，内存中只需要一份即可   
        +   没有：到LD_LIBRARY_PATH指定的每一个路径下去寻找，找到“动态库”后就加载到内存中而且加载时，整个动态库代码会被全部加载到内存中。  

##### 2）在程序中调用“动态库加载函数”来加载
+ （a）Linux下的动态库加载函数
    这里特意强调了Linux下，意思就是Windows下的动态库加载函数与Linux是不一样的。  
     Linux下的动态库加载函数是c库函数，但是不是c标准库提供的，而是Linux这个平台的c库函数，也就是说在windows下不能使用Linux下的动态库加载函数。
    + dlopen：
        - 函数原型：
                ```c
                #include <dlfcn.h>  
                void *dlopen(const char *filename, int flags);  
                ```

        - 功能：打开动态库文件，将动态库文件中的代码加载到内存中。  

        - 参数：
            `filename`：动态库的路径名，比如./dynamic_lib/libmycaculate.so，此时库的名字一定要写全名，lib和.so不能省略。  
            flags：打开方式  
                flags的选项有好多，由于dlopen函数用的并不多，所以这里只介绍我们会用到的RTLD_NOW选项。  
                RTLD_NOW：简单理解就是，立即打开动态库文件并加载到内存中，当然这个理解并不准确，但是我们目前就先这么理解。如果以后大家真用到了这个函数时，在自己去深入了解这个函数。  

        - 返回值：
            成功：返回一个void *指针，后续利用这个指针就可以去调用“动态库函数”。  
            失败：返回NULL。  


    + dlclose：
            dlopen用于加载动态库，dlclose刚好相反，用于将动态库从内存中卸载掉。  
            `int dlclose(void *handle);`
            参数为dlopen的返回值。  
            返回值：成功返回0，失败返回非零值  
            不过当多个程序都在共享使用动态库时，只要还有一个程序还在调用动态库，dlcose时就不会立即卸载，只有当最后一个调用动态库的程序dlcolse时，才会真正的从内存中卸载掉动态库。  

    + dlsym
        - 函数原型     
            ```c
            #include <dlfcn.h>
            void *dlsym(void *handle, const char *symbol);
            ```

        - 功能  
                动态库被dlopen加载到了内存中后，每个库函数的内存地址就是确定的，此时只要得到了add、sub、mul等函数入口地址（函数指针：绝对地址），自然就能调用这些函数了。  
                dlsym的作用就是用来返回“每个动态库函数”在内存中函数指针，只不过返回的类型为void *，使用时需要强制转为对应的函数指针类型。  
                图：  

        - 参数  
            `handle`：dlopen返回的指针  
            `symbol`：库函数的名字，为一个字符串，dlsym会通过名字去查找动态库函数的函数指针。  


        - 返回值  
            成功：返回某动态库函数在内存中的函数指针  
            失败：返回NULL  


        - 使用例子
            ```c
             double (*funp)();
             funp=(double (*)())dlsym(handle,"sub"); 
            ```
            由于“动态库代码”中保留了符号名称（字符串），所以才能在动态库代码中通过函数名称找到该函数的函数指针。

+ （b）main.c
    ```c
    #include "caculate.h"  //包含静态库的头文件
    #include <stdio.h>
    #include <dlfcn.h>

    int main(void)
    {
            double a = 10.5;
            double b = 20.6;
            double ret = 0;

            double (*funp)() = NULL; //函数指针变量，用于存放dlsym返回的库函数的函数指针
            void *handle = NULL;  //存放dlopen所返回的指针

            //打开动态库文件，并将代码加载到内存中，为了让代码简洁一些，我们省略出错处理
            handle = dlopen("./dynamic_lib/libmycaculate.so", RTLD_NOW); 


            //返回add库函数在内存中的绝对地址，并强制转换为double (*funp)()
            funp=(double (*)())dlsym(handle,"add");  
            ret = funp(a, b);  //通过add的函数指针来调用动态库中add函数
            printf("add: ret = %f\n", ret);

            funp=(double (*)())dlsym(handle,"sub");  //同上
            ret = funp(a, b);  //同上
            printf("sub: ret = %f\n", ret);

            funp=(double (*)())dlsym(handle,"mul"); 
            ret = funp(a, b);
            printf("mul: ret = %f\n", ret);

            funp=(double (*)())dlsym(handle, "div");
            ret = funp(a, b);
            printf("div: ret = %f\n", ret);

            funp=(double (*)())dlsym(handle,"power");
            ret = funp(a);
            printf("power: ret = %f\n", ret);

            dlclose(handle); //从内存中卸载掉动态库

            return 0;
    }
    ```

+  （c）编译main.c  
    `gcc main.c -o main -ldl`  
    运行程序时，会调用dlopen函数会将libmycaculate.so动态库打开并加载到内存中，然后dlsym函数会返回每个动态库函数在内存中的函数指针，然后通过函数指针即可调用这些库函数。    
    +  -ldl是什么意思  
        在linux下，dlopen、dlsym、dlclose由相应的c库提供，只不过并不是c标准库。而且这个c库还是动态库，名字叫libdl.so，编译时我们需要通过-ldl来链接。  
        如果不链接libdl.so的话，就没办法对dlopen、dlsym等进行符号解析。  
        提供dlopen、dlsym、dlclose函数的c库是Linxu平台的c库，只在linux平台有效。  

        `疑问`：为什么没有通过-L指定动态库libdl.so的路径？  
        答：libdl.so动态库的路径加入了环境变量，可以自动找到。
            不过要注意的是，这个环境变量不是前面说的那个，与动态库加载有关的“动态库环境变量”，而是另外的环境变量，有关这个环境变量我们就不再介绍了，《Linux系统编程、网络编程》会讲。  
            `libm.so  libptread.so  -lmycaculate`  
        我们在使用数学库、线程库时，要求指定-lm和-lpthread，比如：  
        ```shell
        gcc **.c **.c -o a.out -lm   //libm.so
        gcc **.c **.c -o a.out -lpthread  //libpthread.so
        ```

    +  为什么没有链接libmycaculate.so
        因为程序中没有直接调用add、sub等动态库函数，"add"、"sub"等只是dlsym的参数而已，不涉及到对add、sub等函数名的符号解析，所以不需要链接libmycaculate.so。  
						
## 8.3 使用IDE方式来制作、使用静态库和动态库
我们以windows下的IDE来介绍IDE方式，这样不仅介绍了IDE方式，而且还介绍了windows平台的静态库和动态库。  
我们所用的IDE为Codeblocks，而且Codeblocks使用的编译器也是gcc    
当然，我们也可以在windows命令行使用命令来制作，不过与Linux命令行方式是相似的，所以我们这里不再介绍。  
而且IDE方式才是主流，命令行方式并不是主流。  
	
### 8.3.1 制作、使用静态库

#### （1）创建制作静态库的c工程
+ 1）打开Codeblocks  
+ 2）File——>New——>Project———>Static library——>Next——>输入工程文件名（比如caculate） 和 选择工程存放路径   
+ 3）新建三个文件，分别命名为add_sub.c、mul_div.c、caculate.h文件的内容不变，还是之前的那些内容  
+ 4）编译制做静态库
    编译静态库c工程，最后就得到了静态库libcaculate.a，它被放在了工程目录下的`bin/Debug/`或者`bin/Release/`下   
    一般来说在windows平台下，静态库的名字应该叫***.lib，但是前面也说过，静态库的名字与编译器也有关系，比如使用gcc编译器来制作静态库时，不管针对的是什么平台，名字都是.a结尾的，当然还有一个lib前缀   
+ 5）我们可以将静态库libcaculate.a和caculate.h集中放到某个目录下，以方便后续使用.比如在桌面创建一个static_lib目录，将libcaculate.a和caculate.h放在里面  
		
#### （2）使用静态库
+ 1）关闭制作静态库的工程，创建一个使用静态库的c工程  
+ 2）新建一个main.c，然后将之前调用静态库的main.c中的内容，复制到当前工程的main.c中  
    main.c:
    ```c
    #include "caculate.h"  //包含静态库的头文件
    #include <stdio.h>

    int main(void)
    {
            double a = 10.5;
            double b = 20.6;
            double ret = 0;

            ret = add(a, b); //加
            printf("add: ret = %f\n", ret);

            ret = sub(a, b); //减
            printf("sub: ret = %f\n", ret);

            ret = mul(a, b); //乘
            printf("mul: ret = %f\n", ret);

            ret = div(a, b);
            printf("div: ret = %f\n", ret);

            ret = power(a);  //除
            printf("power: ret = %f\n", ret);

            return 0;
    }
    ```

+ 3）设置静态库的链接
   + （a）指定caculate.h的路径  
            Settings——>Compiler——>Global compiler settings——>Serch directories——>add——>caculate.h所在路径  
            当然，我们也可以直接将caculate.h复制到main.c所在的目录，此时直接#include "caculate.h"即可  

   + （b）指定要链接的静态库  
            Settings——>Compiler——>Global compiler settings——>Linker settings——>add——>选择你要链接的静态库  

+ 4）编译链接、并运行程序  
        编辑链接时就会链接我们指定的静态库，然后程序即可正常运行  
				
### 8.3.2 通过IDE方式来制作、使用动态库

#### （1）创建制作动态库的c工程
+ 1）打开Codeblocks
+ 2）File——>New——>Project———>Dynamic Link library——>Next——>输入工程文件名比如mycaculate 和 选择工程存放路径
+ 3）新建三个文件，分别命名为add_sub.c、mul_div.c、caculate.h文件的内容不变。
+ 4）编译得到动态库
    编译后就得了动态库，也被放在了工程目录下的bin/Debug/或者bin/Release/下。
        dynamic.dll  
        libdynamic.a   

    在Linux下制作动态库时，所得到只有一个***.so文件，但是制作windows动态库时，所得到文件会有两个：  
    + `***.dll`：真正的动态库  
    + `***.a`：windows下动态库的“动态库引导文件”  

    这个两个文件合起来等价于***.so，***.a这个“动态库引导文件”也是一个静态库，只不过这个静态库里面放的只是动态库符号解析所用基本信息，而不是库函数。  
    + （a）链接Linux下的动态库时：链接的是***.so  
    + （b）链接windows下的动态库时：链接的是“动态库引导文件”，而不是.dll  
+ 5）将dynamic.dll、libdynamic.a和caculate.h集中放到某个目录下，以后续方便使用.比如在桌面创建一个dynamic_lib目录，然后放在里面  
							
#### （2）使用动态库
根据加载动态库方式的不同，分为两种情况：  

+  通过“动态库加载器”加载  
+  在程序中调用“动态库加载函数”来加载  

##### 1）方式1：通过“动态库加载器”加载  
+ （a）创建一个使用动态库的工程  
        File——>New——>Project———>Console application ——>...  

+ （b）将之前调用动态库的main.c中的内容，复制到新工程的main.c中
    main.c:
    ```c
    #include "caculate.h"  //包含静态库的头文件
    #include <stdio.h>

    int main(void)
    {
            double a = 10.5;
            double b = 20.6;
            double ret = 0;

            ret = add(a, b); //加
            printf("add: ret = %f\n", ret);

            ret = sub(a, b); //减
            printf("sub: ret = %f\n", ret);

            ret = mul(a, b); //乘
            printf("mul: ret = %f\n", ret);
            ret = div(a, b);
            printf("div: ret = %f\n", ret);

            ret = power(a);  //除
            printf("power: ret = %f\n", ret);

            return 0;
    }
    ```

+ （c）设置对动态库的链接
    +  指定caculate.h的路径，include时才能找到  
        Settings——>Compiler——>Global compiler settings——>Serch directories——>add——>caculate.h所在路径  
        同样的，我们也可以直接将caculate.h复制到工程目录下，include "caculate.h"时，直接在当前目录下就可以找到狗文件  
    +  指定要链接的“动态库引导文件”    
        Settings——>Compiler——>Global compiler settings——>Linker settings——>add——>“动态库引导文件”所在的路径  
    +  将`***.dll`复制到`***.exe`可执行程序所在目录  
            运行程序时，动态库加载器才能找到这个动态库，并加载到内存中  

+ （d）编译链接、并运行  
    编辑链接时，会链接“动态库引导文件”，对程序中的add、sub等进行符号解析. 要不然会提示找不到add、sub等函数的函数定义  
    编译之后就得到了***.exe可执行程序，运行程序时，“动态库加载器”会将***.exe所在目录下的 mycaculate.dll动态库加载到了内存中，当然，如果之前已经加载了就不再加载   
    `疑问`：为什么linux的动态库和windows动态库会存在区别，.so没有动态库引导文件，而.dll确有动态库引导文件？  
    `答`：这个区别是由linux和windows目标文件的格式不同导致的，Linux目标文件格式ELF格式，windows下的目标文件格式为PE格式，因为这两种格式的不同，而导致了动态库的区别。  

##### 2）方式2：在程序中调用“动态库加载函数”来加载
linux平台所提供动态库加载函数为dlopen、dlsym、dlclose，同样的，windows平台这边也有对应的“动态库加载函数”，分别是LoadLibrary、GetProcAddress、FreeLibrary  
    + `LoadLibrary`：类比于`dlopen`  
    + `GetProcAddress`：类比于`dlsym`  
    + `FreeLibrary`：类比于`dlclose`  
    使用这几个函数时需要包含windows.h，类比于Linux这边的dlfcn.h  
    LoadLibrary、GetProcAddress、FreeLibrary与dlopen、dlsym、dlclose还是有些不同之处：  
    +  dlopen、dlsym、dlclose  
        Linux平台的C库函数，由libdl.so动态库提供，需要通过-ldl链接对应的动态库。    
        这几个函数只在Linux平台有效。  
    · LoadLibrary、GetProcAddress、FreeLibrary  
        这几个函数是Windows的OS API，不需要指定什么`-l***`来链接。  
        那么这几个函数也只能在windows下使用  

+ （a）创建一个调用LoadLibrary等函数来加载动态库的工程  
    File——>New——>Project———>Console application ——>...  

+ （b）在main.c中编写使用LoadLibrary等来加载并使用动态库的代码  
    main.c：  
    ```c
    #include <windows.h>
    #include "caculate.h"
    #include <stdio.h>
    int main(void)
    {	
            double ret = 0;
            HINSTANCE handle;  //类比于Linux这边void *handle
            double (*funp)() = NULL;  //函数指针变量

            //类比于dlopen
            handle = LoadLibrary("C:\\Users\\Administrator\\Desktop\\dynamic_lib\\mycaculate.dll"); //windows编译器所用的动态库为.dll结尾

            funp = (double (*)())GetProcAddress(handle, "add");  //类比于dlsym
            ret = funp(2.3, 3.6);
            printf("%f\n", ret);		

            funp = (double (*)())GetProcAddress(handle, "sub");
            ret = funp(2.3, 3.6);
            printf("%f\n", ret);		

            funp = (double (*)())GetProcAddress(handle, "mul");
            ret = funp(2.3, 3.6);
            printf("%f\n", ret);	

            funp = (double (*)())GetProcAddress(handle, "div");
            ret = funp(2.3, 3.6);
            printf("%f\n", ret);		

            funp = (double (*)())GetProcAddress(handle, "power");
            ret = funp(2.3);
            printf("%f\n", ret);	


            FreeLibrary(handle); //类比于dlclose(handle)

            return 0;
    }
    ```

+ （c）指定caculate.h的路径,与以前是一样的  

+ （d）编译运行   

    程序运行起来后：  
    +  LoadLibrary先将指定路径下的动态库打开，并加载到内存中.当然，如果内存中已经有一份了，LoadLibrary则不会重复加载的  
    +  通过GetProcAddress函数，即可获得某个动态库函数在内存中的函数指针  
    +  得到动态库函数的函数指针后，就可以调用这个动态库函数了  
										
												
### 8.3.3 使用VC++(IDE)来制作windows平台的静态库和动态库

我们这里只是大概的介绍一下，具体的实现请大家查阅资料完成，不过目前大家没有必要去深究，以后碰到时在深究也不晚。  
VC++是微软推出的IDE，所使用的编译器自然为Windows编译器，开发的程序基本都是针对windows平台的。  
	
#### （1）制作静态库
使用VC++所制作的静态库为***.lib，显然windows编译器所制作的静态库的名字，尾缀为.lib。

#### （2）制作动态库

+ 1）真正动态库：`***.dll`  
+ 2）动态库引导文件：`***.lib`，不再是gcc下的.a  

虽然制作库的原理都差不多的，但是VC++与codeblocks毕竟是完全不同的两个IDE，而且各自使用的编译器也不相同，因此制作库的细节会有差异，那么像这些有差异东西，我们就不可能一一讲到，这些东西只能由大家在工作中碰到后自行解决，我的建议是碰到后在解决，没有碰到时就不要去关心了，毕竟我们的时间是很宝贵的，划清工作和学习的重点是很重要的  

不过有一点需要说明一下，Codeblocks制作的静态库为`***.a`，如果想要拿到vc++上使用的的话，需要用工具软件转为`***.lib`，同样的`***.a`动态库引导文件也需要被转为`***.lib`动态库引导文件  
					
### 8.3.4 函数手册
一个正规的库制作完毕后，为了方便编程者使用，我们还应该配上函数手册，使用者通过函数手册，就知道应该如何调用这些库函数  
		
### 8.4.5 库的头文件
对于很重要的库来说，OS或者编译器会自动帮我们提供库所需的.h，include时会自动找到这些.h文件    
对于一般的库（私人库、公司库）来说，OS和编译器是不可能帮我们提供这些头文件的，我们需要自己添加，至于如何添加，在前面的课程中已经介绍过了  

### 8.4.6 再说说静态库和动态库

#### （1）静态库
链接静态库后，程序中就会包含所需的代码，编译链接之后，静态库对于我的程序来说就没有意义了。
			
			
#### （2）动态库
链接动态库时只是留了个接口，程序运行时需要单独的加载动态库

+ 1）对于重要的动态库来说，比如C标准库的动态库，各个平台都会支持，在我们的程序包中不需要包含动态库  

+ 2）对于自己库、公司库来说，平台是不可能提供的，所以我们做好动态库后，需要把动态库和可执行程序  
    打包放到一起，将动态库和可执行程序一起发布，只有这样运行程序时才能找到并加载动态库  
		
