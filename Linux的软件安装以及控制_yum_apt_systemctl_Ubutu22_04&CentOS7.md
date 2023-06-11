## Linux系统的应用商店

操作系统安装软件有许多种方式，一般分为：

- 下载安装包自行安装

	- 如win系统使用exe文件、msi文件等
	- 如mac系统使用dmg文件、pkg文件等
- 系统的应用商店内安装                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               
  - 如win系统有MicrosoftSoft
  - 如mac系统又AppStore

Linux系统同样支持这两种方法。

首先来看看，Linux命令行内的“应用商店”，yum命令安装软件。

## yum命令

**yum**：`RPM`**包软件管理器**，用于自动化安装配置Linux软件，并可以自动解决依赖问题。

命令：`yum [-y] [install | remove | search] 软件名称`

- 选项：-y，自动确认，无需手动确认安装或卸载过程
- install：安装
- remove：卸载
- search：搜索

<font color="red">yum命令需要root权限哦，可以su切换到root或使用sudo提权。</font>

<font color="red">yum命令需要联网。</font>

例：下载wget

<img src="https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303292108036.png" alt="image-20230329210801970" style="zoom: 50%;" />

## apt命令

前面的各类基础Linux命令，都是通用的。但是软件安装，CentOS系统和Ubuntu是使用不同的包管理器。

CentOS使用**yum管理器**，Ubuntu使用**apt管理器**(`.deb`包软件管理器)。



命令：`apt [-y] [install | remove | search]` 软件名称

用法和yum一致，同样需要<font color="red">root权限以及联网</font>。

- `apt install wget`，安装wget
- `apt remove wget`，移除wget
- `apt search wget`，搜索wget

![image-20230329222059388](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303292220826.png)

## systemctl命令 控制软件关闭

### systemctl

Linux系统很多软件（内置或第三方）均支持使用systemctl命令控制：启动、停止、开机自启。

能够被systemctl管理的软件，一般也称之为：服务。

命令：`systemctl start | stop | status | enable | disable 服务名`

- start 启动
- stop 关闭
- status 查看状态
- enable 开启开机自启
- disable 关闭开机自启

1. **系统内置的服务**比较多，比如：

- NetworkManager，主网络服务
- network，副网络服务
- firewalld，防火墙服务

![image-20230330125641838](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303301256030.png)

- sshd，ssh服务（shell远程登录Linux使用的就是这个服务）

![image-20230330125739354](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303301257415.png)



2. 除了内置的服务以外，部分**第三方软件**安装后也可以以`systemctl`进行控制

- `yum install -y ntp`，安装ntp软件。可以通过ntpd服务名，配合systemctl进行控制。
- `yum install -y httpd`，安装apache服务器软件。可以通过httpd服务名，配合systemctl进行控制。

安装完成后，只要该软件内置有去注册服务的功能，即可通过`systemctl`命令控制它的启动和关闭。

> 例：安装ntp软件 ，一个时间同步的软件。该软件安装完成后会自动注册为系统的服务，即可通过`systemctl`命令控制它的启动和关闭。
>
> - `systemctl status ntpd`，查看状态，下图中未启动。
>
> ![image-20230330131150906](C:\Users\lxt\AppData\Roaming\Typora\typora-user-images\image-20230330131150906.png)
>
> - `systemctl start ntpd`，启动并查看状态。
>
> ![image-20230330131554416](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303301315464.png)
>
> - `systemctl enable ntpd`，打开开机自启动。
>
> ![](https://raw.githubusercontent.com/lskjfieh/typora/main/img/202303301318909.png)
>
> - 安装httpd步骤相同。



<font color="red">第三方软件安装后，之所以能够被systemctl控制是因为安装后能够**自动集成**到systemctl中，部分软件安装后没有自动集成到systemctl中，可以**手动进行添加**。</font>

