# Linux项目自动化构建工具 - make/Makefile

## 前言

一个工程中的有许多源文件，其按类型、功能、模块分别放在若干个目录中。而是否会写makefile，从侧面说明了一个人是否具备完成大型工程的能力。

makefile定义了**一系列的规则**来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作。

makefile带来的好处就是**”自动化编译“**。

一旦写好，只需要一个make命令，整个工程完全自动编译，极大地提高了软件开发的效率。

<font color="red">make是一个**命令工具**，是一个解释makefile中指令的命令工具。</font>一般来说，大多数的IDE都有这个命令，比如：Delphi的make，Visual C++的nmake，Linux下GNU的make。可见，makefile都成为了一种在工程方面的编译方法。

<font color="red">make是一条命令，makefile是一个文件，两个搭配使用，完成项目自动化构建。</font>

## makefile的编写

makefile内部维护两种关系，**依赖关系与依赖方法**。

通过编写**文件与文件之间的依赖关系**与**依赖方法**完成makefile。

- 依赖关系：用于让make知道任何目标的来源。例如`hello:hello.o`，`hello`依赖于`hello.o`
- 依赖方法：生成目标的方法。例如`gcc hello.* -option hello.*`

### 实例

> 下面通过一个例子来认识一下makefile：

```bash
[lxt@iZ0jlfdhubp6dl6xyp7yqwZ lesson7_makefile]$ touch mycode.c #在新目录下新建一个.c文件
#将新建.c文件名写入Makefile（此时会自动建立Makefile文件）
[lxt@iZ0jlfdhubp6dl6xyp7yqwZ lesson7_makefile]$ ls > Makefile
[lxt@iZ0jlfdhubp6dl6xyp7yqwZ lesson7_makefile]$ vim Makefile   #进入vim对Makefile进行编辑
[lxt@iZ0jlfdhubp6dl6xyp7yqwZ lesson7_makefile]$ vim mycode.c
```
> 1. **进入vim编辑：**
>    ![image-20230417193619632](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304171936747.png)
>    （也可以使用下面的方法进行编辑：
>    ![image-20230417195319586](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304171953627.png)
>
> 2. **退出后，使用`vim mycode.c`对`.c`进行编辑**
>     ![image-20230417210615394](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304172106429.png)
>
> 3. 开始构建编译
>
>   ![image-20230417211705368](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202304172117731.png)
>
> 

## 原理

**make是如何工作的？在默认的方式下（即只输入make命令），make的工作方式如下：**

>1. make会在当前目录下找名字叫"Makefile"或"makefile"的文件。
>2. 如果找到，它会找文件中的第一个目标文件(target)，在上面的例子中，他会找到"hello"这个文件，并把这个文件作为最终的目标文件。
>3. 如果hello文件不存在，或是hello所依赖的后面的hello.o文件的文件修改时间要比hello这个文件新(可以用`touch`测试)，那么，他就会执行后面所定义的命令来生成hello这个文件。
>4. 如果hello所依赖的hello.o文件不存在，那么make会在当前文件中找目标为hello.o文件的依赖性，如果找到则再根据那一个规则生成hello.o文件。(这有点像一个堆栈的过程)
>5. 当然，你的C文件和H文件是存在的啦，于是make会生成 hello.o文件，然后再用hello.o文件声明make的终极任务，也就是执行文件hello了。
>6. 这就是整个make的依赖性，make会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。
>7. 在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。
>8. make只管文件的依赖性，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。

## 项目清理

> **工程是需要被清理的**。
>
> 像clean这种，没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以显示要make执行。即命令——"make clean”，以此来清除所有的目标文件，以便重编译。
>
> 但是一般我们这种clean的目标文件，我们将它设置为伪目标，用`.PHONY`修饰，伪目标的特性是，总是被执行的。

```makefile
#将clean设置为伪目标
.PHONY:clean
clean:
	rm -f mycode
```

