# Linux小程序 — 进度条

回车和换行其实是两个概念。

回车是回到当前行的最开始。换行是新起一行，列不变。

---

## 行缓冲区

> 看下面这个例子。左右两边的区别是输出语句中是否添加`\n`。
> ![QQ图片20230419221432](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304192214775.png)
> 可以发现，两个代码中的`sleep(5);`语句都在`printf`后，不同的是，去掉`\n`之后，休眠五秒后`printf`才会输出，这是否意味着`sleep(5)`此时先于`printf`执行呢？
>
> 并非如此。那上面的结果又是为什么呢？
>
> 这是因为，`printf`已经执行，但是数据没有被立即刷新到显示器当中。
>
> 没有`\n`，字符串暂时被保存到**用户C语言级别的缓冲区**，而显示器设备的**刷新策略**就是行刷新，所以有`\n`时会直接显示。

**那如果既不想带`\n`，又想让数据刷新出来，应该怎么做呢？**

- 可以调用`fflush`接口。

> 用`man fflush`查看`fflush`接口用法。
>
> ![image-20230419202216651](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304192022712.png)

  

```c
#include<stdio.h>
#include<unistd.h>

int main(){      
	printf("hello everyday!");
	fflush(stdout);   //调用fflush
    sleep(5);
    return 0;                 
} 
```

![image-20230419221145729](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304192211804.png)

## 关于C的输入输出流

> 因为要进行数据读取或显示，C程序默认会打开三个输入输出流：
>
> - stdin（键盘）
> - stdout（显示器）
> - stderr（显示器）
>
> 类型都为`FILE *`。
>
> ![image-20230419203019534](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304192030579.png)



## 进度条小程序

### 倒计时示例

下面代码可实现10至1倒计时

![image-20230419223350154](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304192233190.png)

有上述引入，完成进度条小程序。

代码如下：

```C
#include<stdio.h>
#include<unistd.h>
#include<string.h>
int main(){
#define NUM 100
    char bar[NUM+1];
    memset(bar, '\0', sizeof(bar));
    int i = 0;
   //添加旋转光标，同一位置刷新不同字符，参考倒计时
   const char *lable="|/-\\";
   while(i <= 100){
     printf("[%-100s][%3d%%][%c]\r", bar, i, lable[i%4]);//左对齐并预留100个位置,加上百分数
     fflush(stdout);
     bar[i]='#';
     i++;
     usleep(50000);
	}
    printf("\n");
    return 0;
}  
```

运行结果：

![image-20230419233933297](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304192339337.png)