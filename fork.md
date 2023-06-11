## fork初识

创建子进程

![image-20230424145134666](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304241451303.png)

首先看下面三个例子：

- 使用`fork()`后，`printf`内内容被打印两遍。
  ![image-20230424153215147](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304241532180.png)
- `fork()`作为返回值，`if`和`else`内两条语句都被执行
  ![image-20230424153925389](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304241539431.png)

- 通过`getpid()`查看父子进程

  ![image-20230425170541888](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304251705933.png)

## 如何理解fork创建子进程？

目前创建进程的方式有，`./cmd` or `run command` or `fork`，在操作系统角度，上面的创建进程的方式，通常是没有差别的。

`fork`本质是创建进程，导致的结果是系统里多了一个进程。而进程又相当于与进程相关的内核数据结构，以及进程的代码和数据，也就是说系统里多了一份相关文件，多了一个描述新增进程的数据结构`task_struct`。

我们只是`fork`了。创建了子进程，但是子进程对应的代码和数据呢？

子进程默认情况下，会“继承”父进程的代码和数据。内核数据结构`task_struct`也会以父进程为模板，初始化子进程的`task_struct`。

fork之后，子进程和父进程的代码是共享的。

运行时代码是不可以被修改的，父子代码只有一份。

而对于数据，默认情况下，也是共享的。不过需要考虑修改的情况。进程之间是独立的，为了维护进程的独立性，通过“写实拷贝”，来完成数据的独立性。

## fork的返回值

`fork`的返回值为`pid_t`。相当于int

## fork的意义

我们创建的子进程，就是为了和父进程干一样的事情吗？这样一般是无意义的。

一般是要让子进程和父进程做不一样的事情。通过fork的返回值来完成：

- 失败：<0

- 成功
  - 给父进程返回子进程的pid
  - 给子进程返回0

> 通过下面这个例子来体会一下。

```c
#include <iostream>
#include <unistd.h>
int main(){

  pid_t id = fork(); //int

  std::cout << "hello proc: " << getpid() << "  hello parent:" << getppid() << "  ret: "<< id << std:: endl;
  
  sleep(1);
  return 0;
}
```

> 运行可发现，父进程返回值为子进程的`pid`5402，子进程返回值为0
>
> ![image-20230425191120473](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304251911511.png)
>
> 
>
> 1. 为何一个`fork()`函数有两种返回值？
>
>    如果一个函数已经开始执行return了，函数的核心功能已经执行完了。`fork()`是创建子进程，当它准备`return`的时候，创建子进程的逻辑已经完成，此时子进程已经有了，代码是共享的，此时父进程、子进程都要执行后续语句，此时父子进程都return。
>
>    返回值是数据，return的时候，会写入吗？会，此时发生了写实拷贝。
>
> 2. 如何理解两个返回值的设置？
>
>    子进程可以有多个，而父进程只有一个，因此子进程可以直接找到父进程，父进程需要通过返回的`pid`控制子进程。

## fork实现多进程

> `fork`之后通常要用`if`进行分流。

```c
#include <iostream>
#include <unistd.h>
int main(){

  pid_t id = fork(); //int

  if(id == 0){
    //child
    while(true){
      std::cout << "I am child, pid: " << getpid() << ", ppid: " << getppid() << std::endl;
      sleep(1);
    }
  }
  else if(id > 0){
    //parent
    while(true){
      std::cout << "I am parent, pid: " << getpid() << ", ppid: " << getppid() << std::endl;
      sleep(2);
    }
  }
  else{
    //TODU
  }
  //std::cout << "hello proc: " << getpid() << "  hello parent:" << getppid() << "  ret: "<< id << std:: endl;
  
  sleep(1);
  return 0;
}
```

> 可以看到当前有两个进程（父子进程）在运行。
> ![image-20230425203601391](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304252036442.png)

fork之后，父子谁先运行是不确定的，这由调度器决定。

