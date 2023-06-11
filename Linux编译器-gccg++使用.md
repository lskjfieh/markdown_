## Linux编译器-gcc/g++使用

在ANSI C的任何一种实现中，存在两个不同的环境。

第一种是**翻译环境**，**在这个环境中源代码被转换为可执行的机器指令**。第二种是**执行环境**，它用于**实际执行代码**。

### 1 翻译环境

> 组成一个程序的每个源文件通过编译过程分别转换成目标代码（object code）。
>
> 每个目标文件由链接器（linker）捆绑在一起，形成一个单一而完整的可执行程序。
>
> 链接器同时也会引入标准C函数库中任何被该程序所用到的函数，而且它可以搜索程序员个人的程序库，将其需要的函数也链接到程序中。

#### 1.1 编译与链接

**在Linux中,我们可以使用gcc来看到编译期间的每一步。**

命令格式：`gcc [选项] 要编译的文件 [选项] [目标文件]`

> `test.c`
>
> ![image-20230415174158300](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304151741331.png)
>
> 1. **预处理（进行宏替换）**
>
>    - 预处理功能主要包括宏定义，文件包含，条件编译，去注释等。
>    - 预处理指令是以#号开头的代码行。
>    - 实例：`gcc -E hello.c -o hello.i`
>    - 选项"-E"，该选项的作用是让gcc在预处理结束后停止编译过程。
>    - 选项"-o"是指目标文件，"i"文件为已经过预处理的C原始程序。
>
>    ![image-20230415200319715](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152003769.png)
>
> 2. **编译（生成汇编）**[包括：语法分析、词法分析、语义分析、符号汇总]
>
>    - 在这个阶段中，gcc首先要检查代码的规范性、是否有语法错误等，以确定代码的实际要做的工作，在检查无误后，gcc把代码翻译成汇编语言。
>    - 用户可以使用"-S"选项来进行查看，该选项只进行编译而不进行汇编，生成汇编代码。
>    - 实例：`gcc -S hello.i -o hello.s`
>
>    ![image-20230415200631474](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152006517.png)
>
> 3. **汇编（生成机器可识别代码）**[形成符号表 汇编指令->二进制指令->`*.o`]
>
>    - 汇编阶段是把编译阶段生成的".s"文件转成目标文件
>    - 读者在此可使用选项"-c"就可看到汇编代码已转化为".o"的二进制目标代码了
>    - 实例：`gcc -c hello.s -o hello.o`
>
>    ![image-20230415200832185](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152008232.png)
>
> 4. **连接（生成可执行文件或库文件）**[合并段表、符号表的合并和符号表的重定位]
>
>    - 在成功编译之后，就进入了链接阶段。
>
>    - 需要链接来将我们自己的代码中的函数调用，外部数据和库关联起来。
>
>      （语言也是有库的，对于c语言，一套头文件+一套库文件[libc.a，libc.so]）
>
>    - 实例：`gcc hello.o -o hello`
>
>    ![image-20230415210022477](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152100515.png)

**隔离编译，一起链接。**

`ldd`命令，查看可执行程序依赖第三方库的命令

![image-20230415210546247](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152105287.png)

**在Linux中，库分为两种**：

> - 静态库：`.a`。值编译链接时，把库文件的代码全部加入到可执行文件中，因此生成的文件比较大，但在运行时也就不再需要库文件了。其后缀名一般为`.a`。
> - 动态库：`.so`。动态库与之相反，在编译链接时并没有把库文件的代码加入到可执行文件中，而是在程序执行时由运行时链接文件加载库，这样可以节省系统的开销。动态库一般后缀名为`.so`，如前面所述的`libc.so.6`就是动态库。gcc在编译时默认使用动态库。完成了链接之后，gcc就可以生成可执行文件。

因此，程序链接也分为两种：**静态链接**和**动态链接**。

> - 静态链接：库中有关的代码拷贝进自己的可执行程序中叫做静态链接。不再需要使用任何第三方库。
>   ![image-20230415213610731](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152136769.png)
>
> 
>
> **gcc默认生成的二进制程序，是动态链接的，这点可以通过`file`命令验证。（网吧模式）**
>   ![image-20230415214029642](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152140674.png)
>
> 而安装静态库后，可通过命令生成采用静态链接方式的可执行程序。
> ![image-20230415214731570](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304152147598.png)

**两种链接的优缺点**：
> 因为采用静态链接方式是将所需库拷贝到自己的可执行程序中，所以生成的可执行程序会比动态链接方式生成的大许多。而相比动态链接方式，静态链接方式生成的可执行程序可移植性更高。所以方式的选择根据不同的需求偏向选择。

#### 1.2 gcc选项

- `-E`  只激活预处理,这个不生成文件,你需要把它重定向到一个输出文件里面
- `-S`  编译到汇编语言不进行汇编和链接
- `-c`  编译到目标代码
- `-o` 文件输出到文件
- `-static` 此选项对生成的文件采用静态链接
- `-g` 生成调试信息。GNU调试器可利用该信息。
- `-shared` 此选项将尽量使用动态库，所以生成文件比较小，但是需要系统有动态库
- `-O0`
- `-O1`
- `-O2`
- `-O3` 编译器的优化选项的4个级别，-O0表示没有优化，-O1为缺省值，-O3优化级别最高
- `-w` 不生成任何警告信息。
- `-Wall` 生成所有警告信息。

> 编译c++时用g++，方法同理。

### 2 运行环境

程序执行过程：

1. 程序必须载入内存中。在有操作系统的环境中∶一般这个由操作系统完成。在独立的环境中，程序的载入必须由手工安排，也可能是通过可执行代码置入只读内存来完成。
2. 程序的执行便开始。接着便调用main函数。
3. 开始执行程序代码。这个时候程序将使用一个运行时堆栈(stack)，存储函数的局部变量和返回地址。程序同时也可以使用静态( static )内存，存储于静态内存中的变量在程序的整个执行过程一直保留他们的值。
4. 终止程序。正常终止main函数，也有可能是意外终止。

## Linux调试器 - gdb使用

如果一个程序是可以被调试的，该程序的二进制文件一定加入了一些debug信息。

程序的发布方式有两种，debug模式和release模式。Linux gcc/g++出来的二进制程序，默认是release模式，不可进行调试。要使用gdb调试，必须在源代码生成二进制程序的时候，加上`-g`选项。

因为加入了debug信息，所以对于同一程序，debug模式发布的体积会大于release模式。

### 使用

`gdb binfile`

退出：`ctrl+d`或`quit`

![image-20230416002951971](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304160029020.png)

调试命令：

> - `list/l行号`:显示binFile源代码，接着上次的位置往下列，每次列10行。
> - `list /l函数名`:列出某个函数的源代码。
> - `r`或`run`:运行程序。
> - `n`或`next`:单条执行。
> - `s`或`step`:进入函数调用
> - `break(b) 行号`:在某一行设置断点
> - `break 函数名`∶在某个函数开头设置断点
> - `info break` :查看断点信息。
> - `finish`:执行到当前函数返回，然后挺下来等待命令
> - `print(p)`:打印表达式的值，通过表达式可以修改变量的值或者调用函数
> - `p 变量`:打印变量值。
> - `set var`:修改变量的值
> - `continue`(或`c`):从当前位置开始连续而非单步执行程序
> - `run`(或`r`):从开始连续而非单步执行程序
> - `delete breakpoints`:删除所有断点
> - `delete breakpoints n`:删除序号为n的断点
> - `disable breakpoints`:禁用断点
> - `enable breakpoints`:启用断点
> - `info`(或`i`) `breakpoints`:参看当前设置了哪些断点
> - `display 变量名`:跟踪查看一个变量，每次停下来都显示它的值
> - `undisplay`:取消对先前设置的那些变量的跟踪
> - `until X行号`:跳至X行
> - `breaktrace`(或`bt`):查看各级函数调用及参数
> - `info (/i) locals`:查看当前栈帧局部变量的值
> - `quit`:退出gdb

**以下面程序作为示例进行调试**

> ![image-20230416004613799](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304160046854.png)
>
> - 进入调试
>
>   ![image-20230416004733371](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304160047404.png)
>
> - `list` (`l`)显示代码
>
>   ![ ](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161051922.png)
>
> - -` b 行号`根据行号打断点
>
>   ![image-20230416111618095](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161116138.png)
>
> - `info b`显示当前已打断点
>
>   ![image-20230416112215178](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161122216.png)
>
> - `r` 开始运行
>
>   ![image-20230416112323637](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161123674.png)
>
> - `s` 逐语句调试，进入该函数内部
>
>   ![image-20230416112701297](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161127331.png)
>
>   继续`s`
>
>   ![image-20230416112748636](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161127674.png)
>
> - `n` 逐过程
>
>   ![image-20230416113006130](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161130161.png)
>
> - `display` 具体看某个变量的值，加`&`查看地址
>
>   ![image-20230416113233817](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161132858.png)
>
> 
>
> - `finsh` （结束当前函数）**VS** `continue`（直接到达下一个断点）
>
> - `until 行号` 跳转到指定行
>
>   ![image-20230416113704008](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161137042.png)
>   
> - `quit` 退出调试
>
>   ![image-20230416113904671](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304161139708.png)