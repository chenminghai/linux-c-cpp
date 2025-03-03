# 12. getcwd、chdir、mkdir、rmdir

## 12.1 getcwd

### 12.1.1 函数原型

```c
#include <unistd.h>
char *getcwd(char *buf, size_t size);
```

这是一个库函数，**pwd命令就是调用这个函数实现的**

+ （1）功能：获取进程的当前工作目录			
   什么是进程的工作路径呢？后面再说

+ （2）参数
  + 1）buf：存放获取到的当前路径的缓存
  + 2）size：缓存的大小

+ （3）返回值
  + 成功返回缓存buf的地址
  + 失败返回空，errno被设置

### 12.1.2 代码演示

```c
char buf[40] = {};
getcwd(buf, sizeof(buf));
printf("current path is %s\n", buf);
```

结果为

```shell
current path is /workspace/chapter03
```

pwd这个命令获取的是，当前终端这个进程的工作路径。

我自己的进程调用`getcwd`函数，获取的是我自己进程的当前工作路径，默认是你运行这个程序时所在的路径。演示：

当前工作路径有什么用？讲到后面你就知道了。

## 12.2 chdir

### 12.2.1 函数原型

```c
#include <unistd.h>
int chdir(const char *path);
```

+ （1）功能：切换进程当前工作目录到path
+ （2）返回值
  + 成功返回0
  + 失败返回-1, errno被设置。

### 12.2.2 代码演示

```c
char buf[40] = {};
chdir("/usr/include");
getcwd(buf, sizeof(buf));
printf("current path is %s\n", buf);
```

结果为

```shell
current path is /usr/include
```

我自己的进程调用chdir函数时，切换的是我自己进程的工作路径，切换后，你调用getcwd获取当前路径后，你会发现当前路径变成了切换后的路径。

cd命令也是调用chdir实现的，使用cd这个命令时，cd会调用getcwd函数来切换当前终端这个进程的工作路径。

## 12.3 mkdir函数

> mkdir命令调用的就是这个函数。

### 12.3.1 函数原型

```c
#include <sys/stat.h>
#include <sys/types.h>

int mkdir(const char *pathname, mode_t mode);
```

+ （1）功能：创建新目录。
  + 1）pathname：需创建目录的路径名
  + 2）mode：指定目录的原始权限，一般给0775
    给目录指定原始权限时，一定要有x权限，否者无法进入这个目录，有关这个问题，我们在前面的课程中已经讲得非常清楚了。

+ （2）返回值：调用成功返回0，失败返回-1，errno被设置

### 12.3.2 测试用例：

## 12.4 rmdir函数

rmdir和rm命令删除目录时，调用的都是rmdir这个函数。

rmdir命令：只能删除空目录，rmdir命令用的很少
rm：不管目录空不空，都能删除，rm用的最多

```shell
     rm
     |
   remove
   |    |
   |    |
   /    \
  /      \
unlink   rmdir
```

思考：使用图形化界面删除、创建、移动文件等操作，也是调用我们张讲的这些函数来实现的吗?  
答：是的，只不过提供给你操作的图形化的界面，最终调用的还是这些函数来实现的  

### 12.4.1 函数原型

```c
#include <unistd.h>
int rmdir(const char *pathname);
```

+ （1）函数功能：删除路径名为pathname的这个目录  
  不管是我们讲那个函数，在指定路径名时，可以是相对路径，也可以是绝对路径
  删除时，Linux系统会调用相关函数，将目录硬链数全部减位0，然后目录就被删除了

+ （2）函数返回值  
  + 调用成功返回0
  + 失败返回-1, errno被设置

### 12.4.2测试用例：

+ 如果目录不为空，必须递归调用rmdir函数，实现递归删除。

+ 什么是递归删除？  
  当目录不为空时，先调用chdir函数进入目中，然后调用rmdir、unlink把里面的内容删除完毕后，再回到上一级，将空目录删除  
  为了方便操作，可以调用remove函数来间接调用unlink和rmdir函数  
  如果目录很深，需要重复相同的过程。图：  
  我这里就不写递归删除非空目录的代码了，请大家自己下去后，根据我的提示来实现  
  rm命令删除非空目录时，最后还是调用rmdir函数来递归删除实现的  

+ 怎么理解递归这个词?
  起点和终点在一个位置。
