# vi编辑器

## 1 vi/vim编辑器介绍

vi/vim是visual interface的简称，是Linux中最经典的文本编辑器。

同图形化界面中的文本编辑器一样，vi是**命令行下**对文本文件进行编辑的绝佳选择。

vim是vi的加强版本，兼容vi的所有指令，不仅能编辑文本，而且还具有shell程序编辑的功能，可以不同 颜色的字体来辨别语法的正确性，极大方便了程序的设计和编辑性。

## 2 vi/vim编辑器的三种工作模式

<img src="https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303251611507.png" alt="image-20230325161143458" style="zoom: 45%;" />

- 命令模式（Command mode）

  命令模式下所敲的按键，编辑器都理解为命令，以命令驱动执行不同的功能。此模式下，不能自已进行文本编辑。

- 输入模式（Insert mode）

​		也就是所谓的编辑模式、插入模式。此模式下，可以对文件内容进程自由编辑。

- 底线命令模式（Last line mode）

​		以`:`开始，通常用于文件的保存，退出。

### 命令模式

如果需要通过vi/vim编辑器编辑文件，请通过如下命令：

`vi 文件路径`

`vim 文件路径`

vim兼容全部的vi功能，`后续全部使用`vim命令。

- 如果文件路径表示的文件不存在，那么此命令会用于编辑新文件。
- 如果文件路径表示的文件存在，那么此命令用于编辑已有文件。

<img src="https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303251851660.png" alt="image-20230325185126575" style="zoom: 67%;" />

**命令模式下常见快捷键**

|   模式   |      命令       |                描述                 |
| :------: | :-------------: | :---------------------------------: |
| 命令模式 |       `i`       |    在当前光标位置进入`输入模式`     |
| 命令模式 |       `a`       |  在当前光标位置之后进入`输入模式`   |
| 命令模式 |       `I`       |   在当前行的开头，进入`输入模式`    |
| 命令模式 |       `A`       |   在当前行的结尾，进入`输入模式`    |
| 命令模式 |       `o`       |   在当前光标下一行进入`输入模式`    |
| 命令模式 |       `O`       |   在当前光标上一行进入`输入模式`    |
| 输入模式 |      `esc`      | 任何情况下输入`esc`都能回到命令模式 |
| 命令模式 |  键盘`上`、键盘`k` |            向上移动光标             |
| 命令模式 |  键盘`下`、键盘`j`  |            向下移动光标             |
| 命令模式 |  键盘`下`、键盘`j`  |            向下移动光标             |
| 命令模式 |  键盘`左`、键盘`h`  |            向左移动光标             |
| 命令模式 |  键盘`右`、键盘`l`  |            向后移动光标             |
| 命令模式 |  `0`  |          移动光标到当前行的开头          |
| 命令模式 |  `$`  |            移动光标到当前行的结尾            |
| 命令模式 |  `pageup(PgUp)`  |      向上翻页           |
| 命令模式 |  `pangdown(PgDn)` |            向下移动光标             |
| 命令模式 |  `/`  |            进入搜索模式             |
| 命令模式 |  `n`  |            向下继续搜索            |
| 命令模式 |  `N`  |            向上继续搜索            |
| 命令模式 |  `dd`  |           删除光标所在行的内容             |
| 命令模式 |  `ndd`  |        n是数字，表示删除当前光标向下n行   |
| 命令模式 |  `yy`  |           复制当前行             |
| 命令模式 |  `nyy`  |         n是数字，复制当前行和下面的n行    |
| 命令模式 |  `p`  |           粘贴复制的内容             |
| 命令模式 |  `u`  |           撤销修改            |
| 命令模式 |  `ctrl+r`  |            反向撤销修改             |
| 命令模式 |  `gg`  |           跳到首行           |
| 命令模式 |  `G`  |          	跳到行尾          |
| 命令模式 |  `dG`  |           从当前行开始，向下全部删除          |
| 命令模式 |  `dgg`  |          从当前行开始，向上全部删除           |
| 命令模式 |  `d$`  |           从当前光标开始，删除到本行的结尾           |
| 命令模式 |  `d0`  |           从当前光标开始，删除到本行的开头           |

### 底线命令模式

在命令模式内，输入`：`，即可进入底线命令模式，支持如下命令：

**底线命令模式常见快捷键**

|     模式     |     命令     |     描述     |
| :----------: | :----------: | :----------: |
| 底线命令模式 |    `:wq`     |  保存并退出  |
| 底线命令模式 |     `:q`     |    仅退出    |
| 底线命令模式 |    `:q!`     |   强制退出   |
| 底线命令模式 |     `:w`     |    仅保存    |
| 底线命令模式 |  `:set nu`   |   显示行号   |
| 底线命令模式 | `:set paste` | 设置粘贴模式 |

