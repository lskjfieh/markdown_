# System V方案 — 详述共享内存

SystemV标准的进程间通信方式，是前人在OS层面专门为进程通信设计的一个方案。由于需要给用户提供功能使用，而OS又不相信任何用户，所以此时采用系统调用。

因此，System V进程间通信，一定会存在专门用来通信的接口（system call）。

总的来说，System V是一个由人们定制的，在**同一个主机内的进程间通信方案**（System V方案）。

 我们知道，进程间通信的本质是，先让不同的进程看到同一份资源。为此，System V提供的主流方案有三个，分别为：

- 共享内存
- 消息队列（有点落伍）
- 信号量

 其中，共享内存和消息队列以**传送数据**为目的，信号量以实现进程间同步或互斥为母的。

## 共享内存

<font color="red">共享内存区是最快的IPC形式</font>。一旦这样的内存映射到共享它的进程的地址空间，这些进程间数据传递不再涉及到
内核，换句话说是**进程不再通过执行进入内核的系统调用来传递彼此的数据**。

可以通过接下来的部分来深入理解这句话。

### 共享内存的原理

1. 通过某种调用，在内存中创建一份内存空间。
2. 通过某种调用，让参与通信的多个进程“挂接”到这份新开辟的内存空间上。

![image-20230518210159716](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305182115853.png)

此时就让不同进程看到了同一份资源，这就叫做共享内存。

 OS内可能存在多份共享内存，因此需要对这些不同的共享内存进行管理。

为了保证两个或者多个进程，看到的是同一个共享内存，共享内存一定要有一定的标识唯一性的ID，方便让不同的进程能识别同一个共享内存资源！

这个“ID”应该在哪里呢？应该在描述共享内存的数据结构中。

### 共享内存数据结构

```c
struct shmid_ds {
struct ipc_perm shm_perm; /* operation perms */
int shm_segsz; /* size of segment (bytes) */
__kernel_time_t shm_atime; /* last attach time */
__kernel_time_t shm_dtime; /* last detach time */
__kernel_time_t shm_ctime; /* last change time */
__kernel_ipc_pid_t shm_cpid; /* pid of creator */
__kernel_ipc_pid_t shm_lpid; /* pid of last operator */
unsigned short shm_nattch; /* no. of current attaches */
unsigned short shm_unused; /* compatibility */
void *shm_unused2; /* ditto - used by DIPC */
void *shm_unused3; /* unused */
}
```

### 共享内存函数

1. `shmget`

   ```c
   功能：用来创建共享内存
   原型:
     int shmget(key_t key, size_t size, int shmflg);
   参数:
     key:这个共享内存段名字
     size:共享内存大小 (共享内存在内核中申请的基本单位是页，内存页(4KB)
     shmflg:由九个权限标志构成，它们的用法和创建文件时使用的mode模式标志是一样的
   返回值：成功返回一个非负整数，即该共享内存段的标识码；失败返回-1
   ```

![image-20230519130134374](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305191301559.png)

2. `shmat`函数

   ```c
   功能：将共享内存段连接到进程地址空间
   原型:
     void *shmat(int shmid, const void *shmaddr, int shmflg);
   参数:
     shmid: 共享内存标识
     shmaddr:指定连接的地址
     shmflg:它的两个可能取值是SHM_RND和SHM_RDONLY
   返回值：成功返回一个指针，指向共享内存第一个节；失败返回-1
   
   //说明
   shmaddr为NULL，核心自动选择一个地址
   shmaddr不为NULL且shmflg无SHM_RND标记，则以shmaddr为连接地址。
   shmaddr不为NULL且shmflg设置了SHM_RND标记，则连接的地址会自动向下调整为SHMLBA的整数倍。
   公式：shmaddr-(shmaddr % SHMLBA)
   shmflg=SHM_RDONLY，表示连接操作用来只读共享内存
   ```

3. `shmdt`函数

   ```c
   功能：将共享内存段与当前进程脱离
   原型:
     int shmdt(const void *shmaddr);
   参数:
     shmaddr: 由shmat所返回的指针
   返回值：成功返回0；失败返回-1
   注意：将共享内存段与当前进程脱离不等于删除共享内存段
   ```

4. `shmctl`函数

   ```c
   功能：用于控制共享内存
   原型:
     int shmctl(int shmid, int cmd, struct shmid_ds *buf);
   参数:
     shmid:由shmget返回的共享内存标识码
     cmd:将要采取的动作（有三个可取值）
     buf:指向一个保存着共享内存的模式状态和访问权限的数据结构
   返回值：成功返回0；失败返回-1
   ```

|    命令    |                             说明                             |
| :--------: | :----------------------------------------------------------: |
| `IPC_STAT` |       把shmid_ds结构中的数据设置为共享内存的当前关联值       |
| `IPC_SET`  | 在进程有足够权限的前提下，把共享内存的当前关联值设置为shmid_ds数据结构中给出的值 |
| `IPC_RMID` |                        删除共享内存段                        |

#### 实例


> 接下来通过实例代码了解一下这些函数或接口，先实现以下代码
>
> - 共享内存的创建与删除

`comm.h`

```c
#pragma once
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>

#define PATH_NAME "./"
#define PROJ_ID 0x6666
#define SIZE 4097
```
`server.c`

```c
#include "comm.h"
int main(){
  key_t key = ftok(PATH_NAME, PROJ_ID);
  if(key < 0){
    perror("ftok");
    return 1;
  }
  //创建共享内存
  int shmid = shmget(key, SIZE, IPC_CREAT|IPC_EXCL);//创建全新的shm，如果和系统已经存在ID冲突，我们出错返回
  if(shmid < 0){
    perror("shmget");
    return 2;
  }
  printf("key: %u, shmid: %d\n", key, shmid);
  sleep(10);
  shmctl(shmid, IPC_RMID, NULL); //属性设为NULL
  return 0;
}
```

`client.c`

```c
#include "comm.h"
int main(){
  key_t key = ftok(PATH_NAME, PROJ_ID);
  if(key < 0){
    perror("ftok");
    return 1;
  }
  printf("%u\n", key);
  return 0;
}
```

> - 如图，执行完创建共享内存的程序后，该进程结束。
>   ![image-20230519233954186](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305192339218.png)
>
> - 通过`ipcs`查看后，可以发现，进程结束，该进程创建的共享内存并未被释放,依旧存在。
>   ![image-20230519233540138](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305192335184.png)
>
> - 这是因为systemV的IPC资源，生命周期是随内核的！只能通过，程序员显示的释放（命令，system call等）或者OS重启释放。
>
>   - 如下使用`ipcrm`删除（面对用户层面），再次查看已经被释放：
>     ![image-20230519233802730](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305192338792.png)
>
>   - 直接在源代码加入删除语句（开发者层面），通过`shmctl`
>
>     ![image-20230520000008939](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305200001389.png)
>
> - 由上述示例也可以看出**`key`和`shmid`的区别**
>
>   - key：只是用来在系统层面进行标识唯一性的，不能用来管理shm
>   - shmid：是OS给用户返回的id，用来在用户层进行shm管理
>   
> - perm为权限，可以设置
>
>   - ![image-20230520002305908](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305200023948.png)

> - 通过共享内存实现控制打印

`comm.h`

```c
#pragma once
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>

#define PATH_NAME "./"
#define PROJ_ID 0x6666
#define SIZE 4097
```

`server.c`

```c
#include "comm.h"
int main(){
  key_t key = ftok(PATH_NAME, PROJ_ID);
  if(key < 0){
    perror("ftok");
    return 1;
  }
  //创建共享内存
  int shmid = shmget(key, SIZE, IPC_CREAT|IPC_EXCL|0666);//创建全新的shm，如果和系统已经存在ID冲突，我们出错返回
  if(shmid < 0){
    perror("shmget");
    return 2;
  }
  printf("key: %u, shmid: %d\n", key, shmid);
  //sleep(10);

  char *mem =(char*)shmat(shmid, NULL, 0); //因为shmat返回值为void*，所以此处要做强转
  printf("attaches shm success\n");
  //sleep(15);

  //此处即为后面要进行的通信逻辑
  while(1){
    sleep(2);
    //这里有没有调用类似pipe or fifo中的read这样的接口呢？
    //没有，所以共享内存一旦建立好并映射进自己进程的地址空间，该进程就可以直接看到该共享内存，
    //就如图malloc的空间一般，不需要任何系统调用接口！
    //也因此，共享内存是所有的进程间通信中速度最快的！
    printf("%s\n", mem); // server 任务共享内存里面放的是一个长字符串
  }
  
  shmdt(mem);
  printf("detaches shm success\n"); 
  //sleep(5);
  shmctl(shmid, IPC_RMID, NULL); //属性设为NULL
  printf("key: 0X%x, shmid: %d -> shm delete success\n", key, shmid);

  //sleep(10);
  return 0;
}
```

`client.c`

```c
#include "comm.h"
int main(){
  key_t key = ftok(PATH_NAME, PROJ_ID);
  if(key < 0){
    perror("ftok");
    return 1;
  }
  printf("%u\n", key);
  //client这里只需获取即可
  int shmid = shmget(key, SIZE, IPC_CREAT);
  if(shmid < 0){
    perror("shmget");
    return 1;
  }
  char *mem = (char*)shmat(shmid, NULL, 0);
  //sleep(5);
  printf("client process attaches success!\n");
  
  //这个地方就是我们要通信的区域
  char c = 'A';
  while(c <= 'Z'){
    mem[c-'A'] = c;
    c++;
    mem[c-'A'] = 0;
    sleep(2);
  }
  shmdt(mem);
  //sleep(5);
  printf("client process detaches success\n");
  return 0;
}
```

![image-20230520160141138](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305201601184.png)

> - 运行效果：
>
>   ![image-20230520154629199](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305201546387.png)
>
> - ctrl+c终止进程后，再次重启会发现如下现象，这是因为共享内存已存在，而上一个进程并未进行到释放该共享内存的部分，此时可手动删除再重启即可
>
>   ![image-20230520154924839](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305201549893.png)
>
>   ![image-20230520155333170](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305201553219.png)

## 消息队列

### 消息队列数据结构

```c
struct msqid_ds {
    struct ipc_perm msg_perm;     /* Ownership and permissions */
    time_t          msg_stime;    /* Time of last msgsnd(2) */
    time_t          msg_rtime;    /* Time of last msgrcv(2) */
    time_t          msg_ctime;    /* Time of last change */
    unsigned long   __msg_cbytes; /* Current number of bytes in queue (nonstandard) */
    msgqnum_t       msg_qnum;     /* Current number of messages in queue */
    msglen_t        msg_qbytes;   /* Maximum number of bytes allowed in queue */
    pid_t           msg_lspid;    /* PID of last msgsnd(2) */
    pid_t           msg_lrpid;    /* PID of last msgrcv(2) */
};

//The ipc_perm structure is  defined  as  follows  (the  highlighted  fields  are settable  using
IPC_SET):

struct ipc_perm {
    key_t          __key;       /* Key supplied to msgget(2) */
    uid_t          uid;         /* Effective UID of owner */
    gid_t          gid;         /* Effective GID of owner */
    uid_t          cuid;        /* Effective UID of creator */
    gid_t          cgid;        /* Effective GID of creator */
    unsigned short mode;        /* Permissions */
    unsigned short __seq;       /* Sequence number */
};

```

### 消息队列函数

#### 创建

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgget(key_t key, int msgflg);
```

#### 删除

```c
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```

> - 消息队列提供了一个从一个进程向另外一个进程发送一块数据的方法
> - 每个数据块都被认为是有一个类型，接收者进程接收的数据块可以有不同的类型值
> - 特性方面：IPC资源必须删除，否则不会自动清除，除非重启，所以system V IPC资源的生命周期随内核

## 信号量

管道，匿名or命名，共享内存，消息队列，都是以传输数据为目的的。

信号量不是以传输数据为目的的，它通过共享“资源”的方式，来达到多个进程的同步和互斥的目的。

信号量主要用于同步和互斥。

信号量的本质，是一个计数器，类似int count，用来衡量临界资源中资源数目的。信号量本身也是临界资源，而信号量内部的count--是原子性的。什么又是临界资源呢？

### 临界资源

凡是被多个执行流同时能够访问的资源就是临界资源（比如多进程启动后，同时向显示器打印，此时显示器就是临界资源），进程间通信时，管道，共享内存，消息队列等，都是临界资源 。

因此，凡是要进程间通信，必定要引入被多个进程看到的资源（通信需要），同时，也造就了引入一个新的问题，临界资源的问题。

### 临界区

进程的代码有很多，其中，用来访问临界资源的代码，就叫做临界区。

### 原子性

没有中间态

### 信号量数据结构

```c
//The semid_ds data structure is defined in <sys/sem.h> as follows:

struct semid_ds {
    struct ipc_perm sem_perm;  /* Ownership and permissions */
    time_t          sem_otime; /* Last semop time */
    time_t          sem_ctime; /* Last change time */
    unsigned long   sem_nsems; /* No. of semaphores in set */
};

//The ipc_perm structure is  defined  as  follows  (the  highlighted  fields  are  settable  using
IPC_SET):

struct ipc_perm {
    key_t          __key; /* Key supplied to semget(2) */
    uid_t          uid;   /* Effective UID of owner */
    gid_t          gid;   /* Effective GID of owner */
    uid_t          cuid;  /* Effective UID of creator */
    gid_t          cgid;  /* Effective GID of creator */
    unsigned short mode;  /* Permissions */
    unsigned short __seq; /* Sequence number */
};

```

### 信号量函数

#### 创建

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semget(key_t key, int nsems, int semflg);
```

#### 删除

```c
int semctl(int semid, int semnum, int cmd, ...);
```

### 进程互斥

> - 由于各进程要求共享资源，而且有些资源需要互斥使用，因此各进程间竞争使用这些资源，进程的这种关系为进程的互斥 
> - 系统中某些资源一次只允许一个进程使用，称这样的资源为互斥资源。
> - 在进程中涉及到互斥资源的程序段叫临界区
> - 特性方面：IPC资源必须删除，否则不会自动清除，除非重启，所以system V IPC资源的生命周期随内核

## 总结

综上所述，我们可以发现，这三种通信方式的数据结构很相似。接口相似，并且数据结构的第一个数据类型是完全一样的（`struct ipc_perm`）！这是由于，在内核中，所有的ipc资源都是通过数组组织起来的。

![image-20230520165011448](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202305201650568.png)

